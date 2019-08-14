---
layout: post
title: "Code contracts (Part 2): Creating my first project with code contracts"
date: 2009-05-22 04:35
author: ahmadreza
comments: true
categories: [Blog]
tags: [Contracts]
---
<p style="font-family:calibri;font-size:11pt;margin:0;">I downloaded code contracts from here. After installing code contract, I tried to create my first project with code contracts. I created new console project name <span style="font-weight:bold;">CodeContractTest . </span>I added reference to Microsoft.Contracts library which is appeared in .Net references after installation.
  <p style="font-family:calibri;font-size:11pt;margin:0;">I wrote following code this is clearly compiled without any warning.
  <div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:c219d176-c5cd-4990-9d79-8b2d87780ee3" class="wlWriterEditableSmartContent">


[sourcecode language="csharp"]
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Diagnostics.Contracts;
using System.Diagnostics;
 
namespace CodeContractTest
{
    class Program
    {
        static void Main(string[] args)
        {
            Swap(new string[] { &quot;Code&quot;, &quot;Contract&quot; }, 1, 4);
        }
        static void Swap(string[] arr, int item1, int item2)
        {
            string temp;
            temp = arr[item1];
            arr[item1] = arr[item2];
            arr[item2] = temp;
        }
 
    }
 
}[/sourcecode]
</pre></div>

<p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
    

<p style="font-family:calibri;font-size:11pt;margin:0;">However, after running this application we will encounter with <span style="font-weight:bold;">IndexOutOfRangeException</span>. This is occurred because illegal parameter in Swap method call.
    

<p style="font-family:calibri;font-size:11pt;margin:0;">Let take a look at new feature added to Visual Studio 2008. I open properties window of project. New tab is added named &quot;Code Contracts&quot;. I checked &quot;Perform Static Contract Checking&quot; and &quot;Implicit Array Bound Obligations&quot;. If I rebuild project new warnings and message will be appeared in Error window.
    


    
    ![](https://gkasoq.bay.livefilestore.com/y1pwEGUN0tRu6dxokdijnyldZr4WfWrx-nQi7pHU2mrIHyhTcbGo10ji8ayYdHYz-xNHOz73AGZuV8rXZPV428u5n_NWXjoc6Ht/SettingCodeContractOptions.png?psid=1)
    

<p style="font-family:calibri;font-size:11pt;margin:0;">Four warnings are about Array access and four suggestion message plus one summary message.
    


    
    ![](https://public.bay.livefilestore.com/y1pBs0IHM3468q69kDP_2J5RP29NNxZFXqQUBAVguOaynepL0KF5Jm2EKF2tqa8iZTOHcK9n5UKSWcFRux2i12O6w/ErrorAndWarnings.png?psid=1)
    

<p style="font-family:calibri;font-size:11pt;margin:0;">In order to improve code and get rid of warnings I should add preconditions in swap method. I added four line of precondition which assure item1 , item2 is in array boundary. I used Contract class and Requires static method which is used for preconditions it has two overloads first overload has a bool parameter. We can use this method to explicitly define the conditions of input parameters.
    

<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:4f0857b6-0863-4d23-a131-94e122485f0c" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
static void Swap(string[] arr, int item1, int item2)
{
    Contract.Requires(item1 &lt; arr.Length);
    Contract.Requires(item1 &gt;= 0);
    Contract.Requires(item2 &lt; arr.Length);
    Contract.Requires(item2 &gt;= 0);
    string temp;
    temp = arr[item1];
    arr[item1] = arr[item2];
    arr[item2] = temp;
}[/sourcecode]
</pre></div>


    
    After rebuilding project new warning is appeared. It shows that one of contract precondition rules is broken.
    


    
    ![](https://public.bay.livefilestore.com/y1poY9KdXrsIFuNilWTT3b8M9YpWysCGDo9xJh4uHlucvRPmvRt5GHbMZ0Tz68HCH1uszbFJtOq5_crrPxI6533qw/WarningAfterContracts.png?psid=1)
    


    
    If I change the swap with proper input parameters after rebuilding project warning messages will be disappeared.
    

<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:eb843702-43fe-4505-8a4a-d7b97e71c77e" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
Swap(new string[] { &quot;Code&quot;, &quot;Contract&quot; }, 0, 1);[/sourcecode]
</pre></div>


    
    If I check &quot;Implicit Non-Null Obligation&quot; on code contracts tab of property window building application will be face with new warning &quot;contracts: Possible use of a null array 'arr'&quot;
    


    
    ![](https://public.bay.livefilestore.com/y1pw_b-tHzJKUiOjB0_Zqb3W59YOBjZGLmL9DSCXsYF-oEccgyBGQfrMVN2_nf_1tYszJM-WW6l_M0yfsr5x1-w5A/ImplicitNon-Null.png?psid=1)
    


    
    Swap method should be called with non-null array parameter. This is another contract that should be added to preconditions. So I changed Swap method as follow.
    

<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:0ebfbb9d-824f-450c-8455-ad7b75d74512" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
static void Swap(string[] arr, int item1, int item2)
 {
     Contract.Requires(arr != null);
     Contract.Requires(item1 &lt; arr.Length);
     Contract.Requires(item1 &gt;= 0);
     Contract.Requires(item2 &lt; arr.Length);
     Contract.Requires(item2 &gt;= 0);
     string temp;
     temp = arr[item1];
     arr[item1] = arr[item2];
     arr[item2] = temp;
 }[/sourcecode]

</div>

<p style="font-family:calibri;font-size:11pt;margin:0;">In this case developer of Swap method define contract for input parameters and there is no need to swap caller know all swap code in order to prevent bad parameter calling.


<p style="font-family:calibri;font-size:11pt;margin:0;">You can download code from [here](https://gkasoq.bay.livefilestore.com/y1pw_b-tHzJKUgqKrlw9pLOxLozFqtBFlOmu3ovl6vcd255a9LLqPkMMBo6Zxe22f9NKvrEMAh0VZxgAKcFsGDsYA/CodeContractTest.zip?download&amp;psid=1)




<a href="https://www.dotnetkicks.com/kick/?url=http%3a%2f%2fahmadreza.com%2fgf%2fblog%2fcode-contracts-part-2-creating-my-first-project-with-code-contracts%2f">![kick it on DotNetKicks.com](https://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fahmadreza.com%2fgf%2fblog%2fcode-contracts-part-2-creating-my-first-project-with-code-contracts%2f)</a>

