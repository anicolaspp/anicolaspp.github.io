# |> Operator in Scala

In a code exchange earlier this week, a coworker saw this weird symbol in my code and rapidly he asked what it was. He seemed very surprised finding about the Pipe Forward Operator (|>)that __F#__ has.

My friend is learning __Scala__ and he wants to use the __|>__ on it, a construct __Scala__ lacks of. Let’s see how we can add this construct to every __Scala__ object.

## Functional Composition

Using the __|>__ makes very easy to compose functions. The code becomes simpler to write and understand by others.

Let’s see an example:

<script src="https://gist.github.com/anicolaspp/10f16118c18852e29fec.js"></script>

Can we make use of |> so this composition is simpler to write? Well, we could write something like this:


We just redefined s so instead of nested function calls, we apply a function to an object and then another and so on, linearly.

I don’t know if it actually makes sense to have something like in Scala, but we can implement it easily.

Pipe Operator in Scala

Let’s take a look at how F# defines |>.


We can mimic the same in Scala, let’s see how:


We have defined a class Pipe that receives a value a of type A so we can apply the function f to a when calling the method |>.

We also defined an object PipeOps to do an implicit conversion to Pipe of any object we want. Please, take a look at the article Implicit conversions in Scala for C# developers for more information about implicit.

Once this has been defined, we can use |> in any object. Let’s see it in practice.


In here, we transform 5 into a Pipe(5) and then we apply the passed function f: x => x + 1 to it. Scala implicit helps a lot in the process.

Map

Let’s see how we could define a map function.


map applies the function f to each item in items. The signature is a little disturbing since normally we would send f as the second parameter and items as the first one in order to take advantage of some of the Scala syntax sugars. However, there is nothing wrong with it, it is just another way to define map.

We can call map by doing:


Let’s see another example with various |> operators.


Note how we have defined square. It is being composed by filter and map using the |> operator. If we have to do this without using |>, the code should look like follows:


Again, this is not how we normally define these kind of functions in Scala, but it will be essentially the same.

Let’s compare our current definitions to the other way we would do it which is more natural to the Scala language.


Conclusions

We have seen how Scala composes functions and how functions are composed in F# by using |>. We also saw how we can implement the |> operator in Scala and comparing how we define these functions by using or not the Pipe Forward Operator. I am not sure if people are actually using |> since Scala has another constructors and code styling that enable another set of possibilities, yet, we still can use |> and its benefits.

Hope this piece is useful to you and if you enjoyed it, please, recommend it (green heart) so others can benefit from it.

Exported from Medium on May 1, 2016.

View the original
