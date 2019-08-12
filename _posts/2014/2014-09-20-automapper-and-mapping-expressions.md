---
layout: post
title: "AutoMapper and mapping Expressions"
date: 2014-09-20 10:19
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [.Net, AutoMapper, C#, MongoDB]
---
AutoMapper is a great library helps developer to almost get rid of left hand right hand assignment between different types. In real world C# .net applications there are lots of scenarios developer wants to map different types of objects to each other. Normally it happens in system boundaries, like between UI and services or between ViewModel and Model. Without AutoMapper you have to do it for all properties every time you want to map to objects. This task is cumbersome, hard to test and bug prone. Automapper with does this for you with minimum cost. It has its own conventiones to facilitate type mapping. On top of built in defaults you as developer can teach Automapper new tricks.

You can find lots of blogs and articles of how to use AutoMapper. In fact [this](https://github.com/AutoMapper/AutoMapper/wiki/Getting-started), [this](https://cpratt.co/using-automapper-getting-started/) and [this](https://lostechies.com/jimmybogard/2009/01/23/automapper-the-object-object-mapper/) can help you.

Following is also an example of rather complex mapping between Source and Destination types.

{% gist ahmad2x4/acb5669edd84beb8aab2 %}

In this example I’ve used xUnit to test and AutoFixture to generate random data. Then in constructor of test class I have created mapping by using static method Mapper.CreateMap. this creates mapping at application domain context and you just need to set this up once and will be available across your application. After that in our test we created a source type and then mapped that to destination type finally checked the destination properties.

This post is not about normal scenario of using Automapper. When you are working with database you’d normally use repositories (arguably you might not need repository). in simple case of repository you would have following code

{% gist ahmad2x4/45ba0cf235a5e2a2ef1b %}

This is a simple interface for repository. Implementation of this doesn't even need mapping (you might do the mapping in other layers, though). However if we want to separate domain model from persistence model we could introduce another interface like following:

{% gist ahmad2x4/45ba0cf235a5e2a2ef1b %}

In this interface we have to generic types TI and TO. TI is our domain model and TO is our persistence model. For example we pass Source object to Add method but repository implementation would save that in Destination model (See bellow example as Add implementation)

{% gist ahmad2x4/cddb3921e5b672ff7f2a %}

This is possible if we map Source object to Destination object. Automapper does that for us.

Now, the real issue is when we want to implement Find method which accepts Expression&lt;Func&lt;Source, bool. Mapping an expression of func of type source to expression of func of type destination is more difficult as [Jimmy Bogard](https://twitter.com/jbogard) main author of Automapper stated in this [mailing list](https://groups.google.com/forum/#!topic/automapper-users/oYxpR_f3Hls). Anyway, with a quick search I’ve found this question where somebody asked in [stackoverflow](https://stackoverflow.com/questions/7424501/automapper-for-funcs-between-selector-types) about AutoMapper for Func’s between selector types. Person how answered this question updated the answer later with answer to Expression mapping with Automapper. The first class which does this magic is an implementation of ExpressionVisitor. This class can traverese an expression tree and replace a single parameter with an arbitrary expression

{% gist ahmad2x4/bce451d879bd731b2de5 %}

The second piece that comes into play is an extension method for Expression&lt;Func&lt;Z,Y&gt;&gt; called “Compose”  and it returns a new Expression&lt;Func&lt;X, Y&gt;&gt;.

{% gist ahmad2x4/8f586c3e08a799ef792a %}

With this extension method and ExpressionVisitor Class I can complete the repository class for. This repository class is implementing SQL Serve implementation. Later on we will implement another repository class for MongoDB as well.

{% gist ahmad2x4/63b6c37597c105bfccbe %}

This is the Entity Framework implementation of our repository. Interesting point is “Find” Method which is taking advantage of Mapper.Engine.CreateMapExpression and Compose extension method. CreateMapExpression in AutoMapper creates an expression for mapping between Source to Destination. In the next line we use the mapper expression and pass this to Compose extension method to build a new expression. And finally we call Where method on Users DBSet and it returns IQueriable of type Destination. I have written a Unit Test to test this.

{% gist ahmad2x4/ae925ff32eeec9685cf1 %}

This test successfully passes. As you see I used Source type to add records and query the records. The repository maps the add part to destination and also map predicates expression.

When I debug the code I can see entity framework successfully constructed the query. This is using Destination type.

{% gist ahmad2x4/74257a9267cd9466f9b4 %}

Well this is really good, although there is a down side to this approach. This is good and pretty mych abstracted the Destination class and I just work with Source type. However, if I want to join results of two different IQueriables I have to mention the property name of the Model.

Anyway it worked reasonably good with SQL server and EntityFramework. Let see if it works with MongoDB.
In order to connect to MongoDB I use mongo csharp driver and on top of that I’m using [MongoRepository](https://mongorepository.codeplex.com). Now let see if the same scenario works for Repository Implemented for MongoDb.

In following code I have implemented the IRepository for MongoDB.

{% gist ahmad2x4/3d4269858753f6e01428 %}

The target is to test Find(Expression&lt;Func&lt;Source, bool&gt;&gt; predicate) and see if it works, however, I have added another Find with Find(Expression&lt;Func&lt;Destination, bool&gt;&gt; predicate) signature and I will describe why I added that method.

Now the unit test to test this implementation.

{% gist ahmad2x4/ec769fe5d372116ad0c2 %}

This test fails!. This complains about full name not having serialization information


Additional information: Unable to determine the serialization information for the expression: <span class="kwrd">&lt;</span><span class="html">MemberInitExpression</span><span class="kwrd">&gt;</span>.FullName.

This approach doesn't work for MongoDB (probably Mongo CSharp driver could not handle that). I have written another method that accepts Expression&lt;Func&lt;Destination, bool&gt;&gt; predicate, If you uncomment this method call and comment the other in unit test it’ll work.

Having looked at transformed expression shows that this approach does not completely re-write the expression.

{% gist ahmad2x4/ed12e9e4538646cfaf5e %}

This is just wrapper around another lambda that does the mapping. Interestingly EntityFramework handles that very well but MongoDB CSharp Driver not.

On option it to find a way to rewrite the whole expression based on the mapping configuration. But as Jimmy mentioned it could be quite hard to implement.

You can find the full implementation and tests [here](https://github.com/ahmad2x4/90d341b36f94325d596e)
