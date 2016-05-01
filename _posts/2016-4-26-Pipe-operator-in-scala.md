# |> Operator in Scala

In a code exchange earlier this week, a coworker saw this weird symbol in my code and rapidly he asked what it was. He seemed very surprised finding about the Pipe Forward Operator (|>)that __F#__ has.

My friend is learning __Scala__ and he wants to use the __|>__ on it, a construct __Scala__ lacks of. Let’s see how we can add this construct to every __Scala__ object.

## Functional Composition

Using the __|>__ makes very easy to compose functions. The code becomes simpler to write and understand by others.

Let’s see an example:

<script src="https://gist.github.com/anicolaspp/10f16118c18852e29fec.js"></script>

Can we make use of __|>__ so this composition is simpler to write? Well, we could write something like this:

<script src="https://gist.github.com/anicolaspp/661c423dd70eea6915ec.js"></script>

We just redefined s so instead of nested function calls, we apply a function to an object and then another and so on, linearly.

I don’t know if it actually makes sense to have something like in __Scala__, but we can implement it easily.

## Pipe Operator in Scala

Let’s take a look at how __F#__ defines __|>__.

<script src="https://gist.github.com/anicolaspp/b51f7986571bad061a1d.js"></script>

We can mimic the same in __Scala__, let’s see how:

<script src="https://gist.github.com/anicolaspp/774851ad942ad54541ec.js"></script>

We have defined a class __Pipe__ that receives a value __a__ of type __A__ so we can apply the function __f__ to a when calling the method __|>__.

We also defined an object __PipeOps__ to do an implicit conversion to __Pipe__ of any object we want. Please, take a look at the article [Implicit conversions in Scala for C# developers](https://medium.com/@anicolaspp/implicit-conversions-in-scala-for-c-developers-92ea6c7902fa#.vnw84hxr9) for more information about _implicit_.

Once this has been defined, we can use __|>__ in any object. Let’s see it in practice.

<script src="https://gist.github.com/anicolaspp/184ccb4b48cb814f4959.js"></script>

In here, we transform 5 into a __Pipe(5)__ and then we apply the passed function __f: x => x + 1__ to it. __Scala__ implicit helps a lot in the process.

## Map

Let’s see how we could define a map function.

<script src="https://gist.github.com/anicolaspp/98e5c954b80102a0f981.js"></script>

__map__ applies the function __f__ to each item in __items__. The signature is a little disturbing since normally we would send __f__ as the second parameter and items as the first one in order to take advantage of some of the __Scala__ syntax sugars. However, there is nothing wrong with it, it is just another way to define __map__.

We can call map by doing:

<script src="https://gist.github.com/anicolaspp/b4e8a94b0cddaa7d1931.js"></script>

Let’s see another example with various __|>__ operators.

<script src="https://gist.github.com/anicolaspp/0234c7ddfada5449f341.js"></script>

Note how we have defined __square__. It is being composed by __filter__ and __map__ using the __|>__ operator. If we have to do this without using __|>__, the code should look like follows:

<script src="https://gist.github.com/anicolaspp/e9421dbe243db4636536.js"></script>

Again, this is not how we normally define these kind of functions in __Scala__, but it will be essentially the same.

Let’s compare our current definitions to the other way we would do it which is more natural to the __Scala__ language.

<script src="https://gist.github.com/anicolaspp/c2ed858794971e9697e6.js"></script>

## Conclusions

We have seen how __Scala__ composes functions and how functions are composed in __F#__ by using __|>__. We also saw how we can implement the __|>__ operator in __Scala__ and comparing how we define these functions by using or not the Pipe Forward Operator. I am not sure if people are actually using __|>__ since __Scala__ has another constructors and code styling that enable another set of possibilities, yet, we still can use __|>__ and its benefits.
