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

 <div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:0540c9dc-bb32-44f9-9c92-dd21c7955bb6" class="wlWriterEditableSmartContent">


[sourcecode language="csharp"]
this.Top = Screen.PrimaryScreen.WorkingArea.Height - this.Height;
this.Left = Screen.PrimaryScreen.WorkingArea.Width - this.Width; [/sourcecode]
</pre></div>

    
    #### WPF application:
    
    
<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:03ee6d9c-208a-4f7e-b2a3-3f1113406375" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
this.Top = System.Windows.SystemParameters.WorkArea.Height - this.Height;
this.Left = System.Windows.SystemParameters.WorkArea.Width - this.Width; [/sourcecode]

</div>


<span style="font-weight:bold;">P.S. 2009/11/30: </span>A friend of mine just asked "Is it possible to get all screen size (in multiple screen) in WPF without refering to System.Windows.Forms". As far as I see, It seems it is not possible to get other screens than current workarea unless you Use System.Windows.Forms.

