---
layout: post
title: Spark Streaming and Twitter Sentiment Analysis
---

This post is the result of an effort to show my coworkers the infinite number of possibilities that Apache Spark along with other ***IoT*** technologies have to offer.

I wanted to show them how to do some simple, yet very interesting analytics that will help them to solve real problems by analyzing specific areas of a social network. Using a subset of the endless twitter ***stream*** looked as the perfect choice since it has everything we need, an endless and continuous datasource (***streams***) ready to be explored. 

## Our Goal

The point of this demostration is to help my coworker to get the insights he needs while showing the power of the Apache Spark by using its streaming capabilities and consice API. 

## Spark Streaming, Minimized 

Spark Streaming is very well explained [here](http://spark.apache.org/docs/latest/streaming-programming-guide.html) so we are going to skip some of the details about the Streaming API and go directly to use it (still we are going to explain the steps to get what we need).

### Setting Up Our App

Let's see how to prepare our app before doing anything else. 

*A very standard initial configuration:*

```scala 
val config = new SparkConf().setAppName("twitter-stream-sentiment")
val sc = new SparkContext(config)
sc.setLogLevel("WARN")
 
val ssc = new StreamingContext(sc, Seconds(5))
 
System.setProperty("twitter4j.oauth.consumerKey", "consumerKey")
System.setProperty("twitter4j.oauth.consumerSecret", "consumerSecret")
System.setProperty("twitter4j.oauth.accessToken", accessToken)
System.setProperty("twitter4j.oauth.accessTokenSecret", "accessTokenSecret")
 
val stream = TwitterUtils.createStream(ssc, None)
```
In here, we have created the Spark Context `sc`, set the log level to `WARN` to eliminate the log noise Spark creates. We also create a Streaming Context `ssc` using `sc`. Then we need to set up our Twitter credentials (before doing this we need to follow [these steps](http://iag.me/socialmedia/how-to-create-a-twitter-app-in-8-easy-steps/)) that we can get from the Twitter website. *Now the real fun starts*.

### What is trading right now

Is it easy to know what is trending on Twitter right. It is just matter of counting the appearances of each tag. Let's see how Spark allows us to do this operation.

```scala
val tags = stream.flatMap { status =>
	status.getHashtagEntities.map(_.getText)
}

tags
	.countByValue()
	.foreachRDD { rdd =>
		val now = org.joda.time.DateTime.now()
		
        rdd
          .sortBy(_._2)
          .map(x => (x, now))
          .saveAsTextFile(s"~/twitter/$now")
      }
```

In here, we got the tags from the tweets, count how many times appears and sort them. After that, we just save the result in order to point Splunk to it. We could build some dashboards using this information so we can track the most tranding hashtags. 


### Analyzing Tweets 

Now, we want to add functionality to get an overral opinion of what people think about a set of topics. For the sake of this example, let's say that we want to know is the ***sentiment*** about **BigData** and **Food**, two very unrelated topics. 

There are several APIs for analyzing sentiments from tweets, but we are going to use an interesting library from **The Stanford Natural Language Processing Group** in order extract the corresponding ***sentiment*** of each twitt.

In our `build.sbt` file we need to add the corresponding dependencies. 

```scala
libraryDependencies += "edu.stanford.nlp" % "stanford-corenlp" % "3.5.1"
libraryDependencies += "edu.stanford.nlp" % "stanford-corenlp" % "3.5.1" classifier "models"
```

Now, we need to select only the twitts we really care about by filtering the ***stream*** so we get only twitts with certain *hashtag (#)*. This filtering is quite easy thanks to unified Spark API. 

Let's see how. 

```scala
 val tweets = stream.filter {t =>
      val tags = t.getText.split(" ").filter(_.startsWith("#")).map(_.toLowerCase)
      
      tags.contains("#bigdata") && tags.contains("#food")
    }
```
Here, we get all the tags in the twitt and check that at least one of them is subways, in order words, the twitt has been tagged with ***#bigdata*** and ***#food***.

Now we have our tweets, we want to extract the correspoding sentiment from them which will give us an overrall view about what people are saying about Subway in social media (Twitter). Let's define a function that extract the sentiment from the twitt content so we could plug it in in our pipeline. 

```scala
def detectSentiment(message: String): SENTIMENT_TYPE
```

We are going to use this function assuming it does what it should and we will put its implementation at the end. In order to give an idea of how it works, let see some tests around it. 

```scala
it("should detect not understoood sentiment") {

      detectSentiment("") should equal (NOT_UNDERSTOOD)

    }

    it("should detect a negative sentiment") {

      detectSentiment("I am feeling very sad and frustrated.") should equal (NEGATIVE)

    }

    it("should detect a neutral sentiment") {

      detectSentiment("I'm watching a movie") should equal (NEUTRAL)

    }

    it("should detect a positive sentiment") {

      detectSentiment("It was a nice experience.") should equal (POSITIVE)

    }

    it("should detect a very positive sentiment") {

      detectSentiment("It was a very nice experience.") should equal (VERY_POSITIVE)

    }
```

These tests should be enought to show it `detectSentiment` works.  

At this point, we are ready to transform our twitts by doing the extraction of the ***sentiment***.

```scala
val data = tweets.map { status =>
	val sentiment = SentimentAnalysisUtils.detectSentiment(status.getText)
	val tags = status.getHashtagEntities.map(_.getText.toLowerCase)

	(status.getText, sentiment.toString, tags)
}
```

`data` represents a `DStream` of tweets we want, the associated sentiment and the hashtags in the tweet (here we should find the tags we use to filter). 

### SQL Interoperability

Now, we want to cross reference the sentiment data with another dataset we have externally and that we can query using SQL. For my friend makes a lot of sense to be able to ***join*** the twitter stream with this other dataset. 

Let's take a look at how we could achieve this. 

```scala
val sqlContext = new SQLContext(sc)

import sqlContext.implicits._

data.foreachRDD { rdd =>

	rdd.toDF().registerTempTable("sentiments")
}
```

We just transformed or stream into a different representation that is backed by other Spark concepts (Resilian, distributed, very fast) so he can use his beloved SQL. The table *sentiment* will be expose as any other table on his system that he could query using SQL. 

```scala
sqlContext.sql("select * from sentiments").show()
```

We easily could increase the size of this stream by increasing the batch time, of by doing windowed operations on the stream which increases the flexibility of our application. Windowed operation are important if we need to look back on the stream. This kind of operations are not trivial and require certain amount of work to accomplish. However, in Spark we could just do:

```scala
tags
	.window(Minutes(1))
	. (...)
```
