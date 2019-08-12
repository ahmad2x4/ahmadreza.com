---
layout: post
title: "Value type property and C# compilation error"
date: 2009-11-28 21:48
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [C#]
---


Two days ago a question was asked in [StackOverflow.com](https://stackoverflow.com/questions/1802203/setting-padding-why-it-says-padding-all-is-not-variable/1802300#1802300) and I found this question so good to prepare a post. I want to explain this question:
 

[Petr](https://stackoverflow.com/users/193605/petr) asked: *I do not understand why there is Control.padding.all which is int and according to hint there is set as well as get but I cannot set it (Control.Padding.All=5)? I would be grateful for explanation.*
 

For example when you want to set
 

``` Csharp 

textBox1.Padding.All = 5;

```

    
You will get this compiling error: Cannot modify the return value of 'System.Windows.Forms.TextBoxBase.Padding' because it is not a variable



![Error Value Property](/images/2009/ErrorValueProperty.jpg)



Following example is an implementation of this error:
    
``` CSharp
public class ARAControl 
{ 
    public ARAPadding Padding { get; set; } 
}
public struct ARAPadding 
{ 
    public int All { get; set; } 
}

```

If you use this you'll get mentioned error:

``` CSharp

ARAControl control = new ARAControl();
control.Padding.All = 10;

```

And why this compiling error happens. It hapens because structure is a value type. By setting this property you first call get Method. "Property Get" will return a copy of ARAPadding, so it is a value type and C# will detect out mistake and prevent compiling.