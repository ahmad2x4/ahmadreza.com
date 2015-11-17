---
layout: post
title: "Screen size in WPF applications"
date: 2009-11-20 10:31
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [WPF]
---


I was about to write a simple application deal with screen size in WPF. Previously in windows form applications we could get WorkingArea parameters from System.Windows.Forms.WorkingArea but in WPF application typically we don’t have this assembly (however this assembly could be added to project). In WPF applications screen size could be obtained from System.Windows.SystemParameters. Following code is an example of setting a form right-bottom aligned in screen.
 

#### Windows form application:


``` CSharp 
this.Top = Screen.PrimaryScreen.WorkingArea.Height - this.Height;
this.Left = Screen.PrimaryScreen.WorkingArea.Width - this.Width; [/sourcecode]
```
    
#### WPF application:
    
    
```Csharp 
this.Top = System.Windows.SystemParameters.WorkArea.Height - this.Height;
this.Left = System.Windows.SystemParameters.WorkArea.Width - this.Width; [/sourcecode]
```


P.S. 2009/11/30: 
A friend of mine just asked "Is it possible to get all screen size (in multiple screen) in WPF without refering to System.Windows.Forms". As far as I see, It seems it is not possible to get other screens than current workarea unless you Use System.Windows.Forms.

