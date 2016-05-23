layout	title
post
Spark Custom Streaming Sources


# TDD and Ninject Modules

Following our commitment to Test Driven Development (TDD), we should not write a line of code before writing a failing test for it, right? yet, many people write, in an unconscious way, a lot of code without doing any kind of testing around it.

## Ninject

Ninject is an Open Source Dependency Injector for .NET and you should take a look at it if you haven’t.

Ninject allows us to manage the dependencies between our classes relieving us from this painful process.

We can simply configure Ninject by creating a Kernel and binding the dependencies.

{% gist 88a6b7cc0180f42fb97c95838c9e4b98 %}
