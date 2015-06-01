---
layout: post
title: "Value type property and C# compilation error"
date: 2009-11-28 21:48
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [C#]
---


Two days ago a question was asked in [StackOverflow.com](http://stackoverflow.com/questions/1802203/setting-padding-why-it-says-padding-all-is-not-variable/1802300#1802300) and I found this question so good to prepare a post. I want to explain this question:
 

[Petr](http://stackoverflow.com/users/193605/petr) asked: *I do not understand why there is Control.padding.all which is int and according to hint there is set as well as get but I cannot set it (Control.Padding.All=5)? I would be grateful for explanation.*
 

For example when you want to set
 <div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:d11fb316-d936-4882-8797-89710b92356b" class="wlWriterEditableSmartContent">


[sourcecode language="csharp"]
textBox1.Padding.All = 5;[/sourcecode]
</pre></div>

    
    You will get this compiling error: Cannot modify the return value of 'System.Windows.Forms.TextBoxBase.Padding' because it is not a variable
    

    
    ![](http://public.bay.livefilestore.com/y1peB4CgbDckyrE2kbNHpWrhSulExj9LvU64qvmgG_dU4Gd4EmnUUOcDl4SnKuqJEylKOAIFOqvnAzL12OoQflk1g/ErrorValueProperty.jpg?psid=1)
    

    
    Following example is an implementation of this error:
    
<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:a874c062-57dc-40ae-ac39-72f773d18212" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
public class ARAControl 
{ 
    public ARAPadding Padding { get; set; } 
}
public struct ARAPadding 
{ 
    public int All { get; set; } 
}[/sourcecode]
</pre></div>

    
    If you use this you'll get mentioned error:
    
<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:f7048276-6b21-4770-8272-39ad55dd6f70" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
    ARAControl control = new ARAControl();
    control.Padding.All = 10;
[/sourcecode]

</div>


And why this compiling error happens. It hapens because structure is a value type. By setting this property you first call get Method. "Property Get" will return a copy of ARAPadding, so it is a value type and C# will detect out mistake and prevent compiling.

