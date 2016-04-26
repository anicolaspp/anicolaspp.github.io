---
layout: post
title: Spark Custom Streaming Sources
---

Streaming data is a hot topic these days and Apache Spark has a very strong presence when we talk about this.

Spark Streaming brings to us the ability to stream from a variety of sources while at the same time we could use the same concise API for accessing to data streams or to query in SQL fashion, or even to create machine learning algorithms. This makes Spark a preferable framework for streaming (or any workflow) applications since we can use all flavors of the framework without limitation on any aspect of the IoT World.

The problem comes to how to integrate custom data sources into Spark so we could leverage its power without the necessity to change to more standard sources. It might seem logical to change, but in some cases it is just not possible or not convenience at a particular moment in time.

## Streaming Custom Receivers

Spark offers different extension points, as we could see when we extended the Data Source API [here](https://medium.com/@anicolaspp/extending-our-spark-sql-query-engine-5f4a088de986#.veec9onyo) so we could integrate our custom data store into Spark SQL.

Today, we are going to do the same, but this time, we are going to extend the streaming API so could stream from __anywhere__.

In order to implement our custom receiver we need to extend the Receiver[A] class. Note that is has type annotation, so we could enforce type safety on our DStream from the streaming client side point of view.

We are going to use this custom receiver to stream orders that one of our applications send over a socket.

The structure of the data traveling through the network look like this:

```
1 5
1 1 2
2 1 1
2 1 1
4 1 1
2 2
1 2 2
```
In order words, we first receive the order id and the total amount of the order and then we receive the line items of the order where the first value is the item id, the second the order it (match the order id value) and then the cost of the item. In the example, we have two orders. The first one with 4 items and the second one with only one item.

The idea is to hide all this from our Spark application so what it receives on the DStream in a complete order so we could define a stream like follows:

<script src="https://gist.github.com/anicolaspp/a6ccb1940ef3a01415e53a815a2593df.js"></script>

At the same time, we are also using the receiver to stream our custom streaming source. Even though that it sends the data over a socket it will be quite complicated to use the standard socket stream from Spark since we will have not control on how the data is coming in and we will have the problem of conforming orders on the app itself. This could be very complicated since once we are in the app space we are running in parallel and it is hard to sync all this incoming data. However, in the receiver space it is easy to create orders from the raw input text.

Let’s take a look how our initial implementation looks like.

<script src="https://gist.github.com/anicolaspp/f5cb0f73f77447fb11c81c8d3cb1d5b4.js"></script>

Our OrderReceiver extends Receiver[Order] which allows us to store an Order (type annotated) inside Spark. We also need to implement the onStart() and onStop() methods. Note that onStart() creates a thread so it is non-blocking which is very important for proper behavior.

Now, let’s take a look at the receive method, where the magic really happens.

<script src="https://gist.github.com/anicolaspp/e6bc69b4aa7bd8541d895f3fa60c8621.js"></script>

In here, we create a socket and point it to our source and then we just simply start reading from it until a stop command has been dispatched or our socket has no more data on it. Note that we are reading the same structure we have defined previously (how our data is being sent). Once we have completely reading an Order, we call store(…) so it gets saved into Spark.

There is nothing left to do here but to use our receiver in our application, which look like this:

<script src="https://gist.github.com/anicolaspp/c9ae8a17389d55cf4660f6626bcc41da.js"></script>

Note how we have created the stream using our custom OrderReceiver (the val stream has been annotated only for clarity but it is not required). From now on we use the stream (DString[Order]) as any other stream we have used in any other application.

<script src="https://gist.github.com/anicolaspp/76cf974b7f0889002544017c58ba3d26.js"></script>

## Closings
Spark Streaming comes very handy when processing sources that generate endless data. It allows us to use the same API we could use for Spark SQL, and other components in the system, but also it is flexible enough to be extended so we could bend it to our necessities.
