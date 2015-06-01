---
layout: post
title: "Value type field members are in heap"
date: 2014-01-31 00:15
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [.Net, C#, CLR, ValueType]
---
I think developers are a bit confused about the concept of ValueTypes and Reference Type. Often I hear that all primitive data types and value types are in stack. First of all it is thread’s stack and it is not a single stack for application. Secondly, when we say value types are in stack it doesn't include field members of the class. Only method parameters and local variable which are value type will be in the thread’s stat. Field members, even if they are value type, will be in heap. Consider following example:

[sourcecode language="csharp"]

using System;

namespace ClassLibrary
{
    public class Foo
    {
	public int number = 0;
        public int Bar(int x, int y)
        {
         	int sum = x + y + number
	 	return sum;
        }
    }
}
[/sourcecode]

In this example, only x, y, sum and return value are in thread's stack and number variable will be held in heap as part of object instance and is accessible using this keyword (which compiler let you omit that).
In other word when we say object instance is created in the heap, what does really stored there? Field members are object state that needs to be stored in object instance.
