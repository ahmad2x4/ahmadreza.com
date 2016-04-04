---
layout: post
title: "Unit testing ModelBinders in ASP.Net Web Api projects"
date: 2016-04-04 21:25
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [.Net, Web API, C#, Unit Test]
---

My Colleague Matt Davies has done a blog post a couple of years ago about [Unit testing MVC3/MVC4 Model Binders](http://mdavies.net/2013/06/07/unit-testing-mvc3mvc4-model-binders/). This is a nice post but as you know Web Api Side of ASP.Net is usually using a different set of object model therefore the setting up the `BindingContext` would be different than what we have in MVC. I also searched and did not find a good blog post about how to unit test ModelBinders in ASP.NEt Web Api project, so I decided to put together this post.

Let’s assume we have following implementation of model binder which bind a sort expression like `FirstName desc` or `LastName` to a simple `class` structure which has `FieldName` and `Ascending` as `boolean` to indicate sorting direction. 
Following is the code for `ModelBinder`:

{% gist ahmad2x4/325c0a5adc0230f28e46cd11995c8bb2 %}

We want to write a few unit test to make sure logic of this ModelBinder is working and in each different test case we want to provide different value and check the result. In order to be able to call `.BindModel` we need two parameters `HttpActionContext` and `ModelBindingContext`. `HttpActionContext` can be completely mocked. We are using [NSubstitute](http://nsubstitute.github.io/) as mocking framework. `ModelBindingContext` has a property called `ValueProvider` which obviously provide value of the model. So we create a `ModelBindingContext` and set the `ValueProvider` to a mocked version and keep the reference to be able to change value for each test. 

Following is the complete code for setting up the model binder and testing that.

{% gist ahmad2x4/0b927ae129050cb6f01573530676d763 %}

In each unit test we set up the `_valueProvider` mock to return expected sortExpression and then call the method and assert on expected values.

Simple!
