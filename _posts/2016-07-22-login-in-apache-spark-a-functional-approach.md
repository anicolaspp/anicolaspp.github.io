---
layout: post
title: How to log in Apache Spark, a functional approach
---

# How to log in Apache Spark, a functional approach

Logging in Apache Spark comes very easy since Spark offers access to a logobject out of the box. Only some configuration setups need to be done. In a previous post we have looked at how to do this while showing some problems that may arise. However, the solution presented might cause some problems at the moment we want to collect the logs since they are distributed across the entire cluster. Even if we utilize Yarn log aggregation capabilities, there will be some contentions that might affect performance or even worse, in some cases we could end with log interleaves corrupting the nature of logs itself, they time ordered properties they should present.
In order to solve these problems, a different approach needs to be taken, a functional one.
The Monad Writer
I do not intend to go over the details about monads or in this particular case, the Monad Writer, if you are interested in learning more, take a look at this link (functor, applicative, and monad) which is very informative about this topic.
Just to put things in context, let’s say that the monad writer (writer) is a container that holds the current value of a computation in addition to history (log) of the value (set of transformation on the value).
Because the writer monadic properties, it allows us to do functional transformations and we will soon see how everything sticks together.

## A Simplistic Log

The following code demonstrates a simplistic log.

```
object app {
  def main(args: Array[String]) {
    val log = LogManager.getRootLogger
    log.setLevel(Level.WARN)

    val conf = new SparkConf().setAppName("demo-app")
    val sc = new SparkContext(conf)

    log.warn("Hello demo")

    val data = sc.parallelize(1 to 100000)

    log.warn("I am done")
  }
}
```

The only thing to note is that logging is actually happening on the Spark driver, so we don’t have synchronization or contention problems. Everything starts to get complicated once we start distributing our computations.
The following code won’t work (read previous post to know why)

```
val log = LogManager.getRootLogger
val data = sc.parallelize(1 to 100000)

data.map { value => 
    log.info(value)
    value.toString
}
```

A solution to this was also presented in the previous post, but it requires extra work to manage the logs.
Once we start logging on each node of the cluster, we need to go to each node, and collect each log file in order to make sense of whatever is in the logs. Hopefully, you are using some kind of tool to help you with this task, such as Splunk, Datalog, etc… yet you still need to know a lot of stuffs to get those logs into your system.

## Our Data Set

Our data set is a collection of the class Person that is going to be transformed while keeping an unified log of the operations on our data set.
Let’s suppose we want our data set to get loaded, then filter each people whose age is less than 20 years, and finally extract its name. It is a very silly example, but it will demonstrate how the logs are produced. You could replace these computations, but the ideas of building an unified log will remain.
Getting the Writer

We are going to use Typelevel / Cats library to import the monad writer, to do this we add the following line to our build.sbt file.
libraryDependencies += "org.typelevel" %% "cats" % "0.6.1"

## Playing with our data

Now, let’s define the transformations we are going to use.
First, let’s load the data.

```
def loadPeopleFrom(path: String)(implicit sc: SparkContext) = 
  s"loading people from $path" ~> sc.textFile(path)
                                    .map(x => User(x.split(",")(0), x.split(",")(1).toInt))
```

In here the ~> operation is defined via implicit conversions as follows.

```
implicit class toW(s: String) {
  def ~>[A](rdd: RDD[A]): Writer[List[String], RDD[A]] = Writer(s :: Nil, rdd)
}
```

If you look closely, our loading operation is not returning an RDD, in fact, it returns the monad writer that keeps track of the logs.

Let’s define the filter that we want to apply over the collection of users.

```
def filter(rdd: RDD[User])(f: User => Boolean) = "filtering users" ~> rdd.filter(f)
```

Again, we are applying the same function (~>) to keep track of this transformation.

Lastly, we define the mapping which follows the same pattern we just saw.

```
def mapUsers(rdd: RDD[User])(prefix: String): Writer[List[String], RDD[String]] = 
  "mapping users" ~> rdd.map(p => prefix + p.name)
```

## Putting it together

So far we have only defined our transformations, but we need to stick them together. Scala for is a very convenient way to work with monadic structures. Let’s see how.

```
val result = for {
  person          <- loadPeopleFrom("~/users_dataset/")(sc)
  filtered        <- filter(person)(_.age < 20)
  namesWithPrefix <- mapUsers(filtered)("hello")
} yield namesWithPrefix

val (log, rdd) = result.run
```

Please note that result is of type: Writer[List[String], RDD[String]].

Calling result.run will give us the log: List[String] and the final computation expressed by rdd: RDD[String].
At this point we could use Spark logger to write down the log generated by the chain of transformations. Note that this operation will be executed on Spark master which implies that one log file will be created with all the log information. Also, we are removing potential contention problems during the log writes. In addition, we are not locking the log file, which avoid performance issues by creating and writing to the file in a serial way.

## Conclusions

We have improved how we log on Apache Spark by using the Monad Writer. This functional approach allows us to distribute the creation of logs along with our computations, something Spark knows well how to do. However, instead of writing the logs on each worker node, we are collecting them back to the master to write them down. This mechanism has certain advantages over our previous implementation. We now control exactly how and when our logs are going to be written down, we boost performance by removing IO operations on the worker nodes, we also removed synchronization issues by writing the logs in a serial way, and we avoid the hazard of fishing logs across our entire cluster.
