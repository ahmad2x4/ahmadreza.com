---
layout: post
title: "Code contracts (Part 1): Introduction"
date: 2009-05-05 12:13
author: ahmadreza
comments: true
categories: [Blog]
tags: [Code Contracts]
---


![](https://ahmadrezaa.files.wordpress.com/2009/05/codecontracts_sm.png "Code Contracts Logo")
  <p style="font-family:calibri;font-size:11pt;margin:0;">During developing applications we have always use unwritten rule in programming or if it is written in design documents user should consider them in development. For example if developer wants to call a method which has an integer parameter and this parameter is used as an index of array he should be aware calling that method with negative value or greater than array lenght. Calling method with negative value causes runtime error.
  <p style="font-family:calibri;font-size:11pt;margin:0;">There are lots of contracts that a developer should consider during development. Recently I came across new concept (However it is in research phase) named <span style="font-weight:bold;">CodeContracts</span> which is really wonderful. It seems that this concept will be in VS 2010. CodeContracts will organized all written and unwritten rules &amp; contract in development. In order to clarify CodeContract usage imagine following code:
  

&#160;
  <div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:16431c63-05a2-4b5f-8502-6de23c7a2d5c" class="wlWriterEditableSmartContent">


[sourcecode language="csharp"]
static void Swap(string[] arr, int itemIndex1, int itemIndex2)
{
    string temp;
    temp = arr[itemIndex1];
    arr[itemIndex1] = arr[itemIndex2];
    arr[itemIndex2] = temp;
}
[/sourcecode]
</pre></div>


    
    What are possible errors when we want to call Swap method?
    



*   <span style="font-family:calibri;font-size:11pt;">None of itemIndex1 and itemIndex2 could be negative (both should be positive)</span> 
*   <span style="font-family:calibri;font-size:11pt;">None of itemIndex1 and itemIndex2 could be greater than arr.length</span> 
*   <span style="font-family:calibri;font-size:11pt;">Array arr should be not null</span> 


    
    Before code contracts developer should always consider possible situation and avoid breaking rule. But CodeContracts provides rich methods by which you can add your Code Contracts to your method and after compiling application Contracts checker will warn about breaking rules. Let see how it should be implemented with CodeContracts
    

<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:a3de732f-82e9-4746-8e5d-007b5636e4c8" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
static void Swap(string[] arr, int itemIndex1, int itemIndex2)
{
     Contract.Requires(arr != null);
     Contract.Requires(itemIndex1 &gt;= 0);
     Contract.Requires(itemIndex1 &lt; arr.Length);
     Contract.Requires(itemIndex2 &gt;= 0);
     Contract.Requires(itemIndex2 &lt; arr.Length);

      string temp;
     temp = arr[itemIndex1];
     arr[itemIndex1] = arr[itemIndex2];
     arr[itemIndex2] = temp; 
} [/sourcecode]

</div>

<p style="font-family:calibri;font-size:11pt;margin:0 0 0 .375in;">&#160;


<p style="font-family:calibri;font-size:11pt;margin:0 0 0 .375in;">I added five contracts to this method which mean this method needs not null array as first parameter, second and third parameter (itemIndex1, itemIndex2) should be greater or equal to zero and should be less than array length.


<p style="font-family:calibri;font-size:11pt;margin:0 0 0 .375in;">In the next post I will show how to implement this with CodeContracts


<p style="font-family:calibri;font-size:11pt;margin:0 0 0 .375in;">But now I want to introduce some resources:




*   <a href="https://research.microsoft.com/en-us/projects/contracts/"><span style="font-family:calibri;font-size:11pt;">Microsoft Research - Code Contracts home page</span></a> 
*   <a href="https://channel9.msdn.com/posts/Peli/Getting-started-with-Code-Contracts-in-Visual-Studio-2008/"><span style="font-family:calibri;font-size:11pt;">Tutorial Video in channel9</span></a> 
*   <a href="https://channel9.msdn.com/pdc2008/TL51/"><span style="font-family:calibri;font-size:11pt;">Contract Checking and Automated Test Generation with Pex</span></a> 



<a href="https://www.dotnetkicks.com/kick/?url=http%3a%2f%2fahmadreza.com%2fgf%2fblog%2fcode-contracts-part-1-introduction%2f">![kick it on DotNetKicks.com](https://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fahmadreza.com%2fgf%2fblog%2fcode-contracts-part-1-introduction%2f)</a>

