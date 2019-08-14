---
layout: post
title: "How to mess with &quot;click test&quot; facebook application"
date: 2010-07-27 05:07
author: ahmadreza
comments: true
categories: [Blog]
tags: [Developer Tools, Facebook, Internet Explorer, JavaScript]
---
Couple of days ago I came across a fantastic challenging application in Facebook which is called “click test”. This application measures your click per second and you have to click as much as you can in 10 seconds. For sure in my 130 friend there are many who can click more than me, but I had to do my bests especially a friend of mine (Kambiz) who is not only good at typing and clicking but also good programming. What should I do! should I left the battle or keep challenging? Of course I have to stay in this battle. I tried to score a good click per second but honestly the best I did was about 90 clicks per second which is lower than lots of my friends.

What is “click test” application? This is a simple but

ton that you have to click on with a countdown timer which runs inside browser. Ok, the first thing pops in mind is writing a simple application to simulate clicking on this button. Ohhh, you have to find browser windows handle then use SendMessage with appropriate (wParam, lParam) parameters to simulate clicking on specific area of window. That sounds great. But the fact is I’m more JavaScript and HTML guy than *API* guy. So let Keep It Simple Stupid (KISS).

If you view source of “test” application page you’ll face huge amount of complex source code which is really complicated. At the beginning I wanted to find the function which is responsible for counting down t from 10 to zero and then increase it but the code is really messy and it takes time to find the function you want. Should I inspect all HTML and Jscript source code? The answer is No

Recently browsers are delivered with quite sophisticated feature for developers. For example Internet Explorer 8 has got famous feature which is called “Developer Tools” and you can activate it by pressing F12. Having pressed F12, nice window will be displayed to help you debug your HTML and JavaScript.

![](https://gkasoq.bay.livefilestore.com/y1pgb6p3gjZOvci4CQ4-CI8jaf-iguAubvX2uR7xqNdULrrHKx7My7QYsQJKS_Ey24_5EkYK3uhsZrr-BmF6jNaLllFNFjkYWiF/DeveloperTools.png?psid=1)

Ok to find start button you can press Ctrl+B or select Find -&gt; Select element by click.

![](https://public.bay.livefilestore.com/y1pvcpmP8PZM8fATmWYqu4qorK5fLe5RiOrgUvLf_p3mfjiNjptuccwSSDi6mJd9oy1GThRDnutlhRfp5d7V8eeag/TestPage.png?psid=1)

Then move your mouse to element that you want inside browse.

![](https://public.bay.livefilestore.com/y1p-DlrtiOiedIskHyea18dPshXMhqGwmLTSfEjIu5HOunS6JA3IPGMJAlLk20Uxq7hw3qEKTNPGGb0faOrP8LF2Q/DeveloperTools2.png?psid=1)

As soon as you move your mouse inside browse you’ll notice that browser mark current element by bordering it. So find start button and then click on it. Gotcha! Developer tools automatically scrolls to input tag related to selected element. You can also scroll horizontally to find “onmouseup” which is actually responsible for counting your clicks. Ok, double click inside onmouseup and then press Ctrl+A to select all body of onmouseup.

![](https://public.bay.livefilestore.com/y1pxbt7UDMLPbPjHtq80EfxPHVQ46mbNVwYx2kurFfhJRgXx00ZOXkGdLi7vTeCr5Vrdb2h9zcSRNQujGQOhHGHBA/DeveloperTools3.png?psid=1)

So I can copy this element into clipboard and then paste in notepad. You may also have somthing similar to following:
<div id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:53eb48f1-2c8a-4c29-b26a-e6be489cadbc" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;">[sourcecode language="javascript"]
fbjs_sandbox.instances.a130168443678744.bootstrap();
return fbjs_dom.eventHandler.call([fbjs_dom.get_instance(this,130168443678744),function(a130168443678744_event)
{a130168443678744_game.update($FBJS.ref(this).getForm())},130168443678744],new fbjs_event(event));
[/sourcecode]

</div>
Actually this is an event handler and this code runs every time you click on your mouse button. But it’s rather complicated. It calls a function at first and then calls another function. So If I want to change this code I have to find these functions and then change body of them.

But what if I don’t change any code and just make this event handler to run multiple times per click. That is sound good idea. So let copy and paste event handler for couple of times. But I should notice that second function call has “return” and I have to remove it for my new copies, and just leave it for last one.
<div id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:31c95ce5-dccf-446d-8f59-bcb17247d81c" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;">[sourcecode language="csharp"]
fbjs_sandbox.instances.a130168443678744.bootstrap();
fbjs_dom.eventHandler.call([fbjs_dom.get_instance(this,130168443678744),function(a130168443678744_event)
{a130168443678744_game.update($FBJS.ref(this).getForm())},130168443678744],new fbjs_event(event));
fbjs_sandbox.instances.a130168443678744.bootstrap();
fbjs_dom.eventHandler.call([fbjs_dom.get_instance(this,130168443678744),function(a130168443678744_event)
{a130168443678744_game.update($FBJS.ref(this).getForm())},130168443678744],new fbjs_event(event));
fbjs_sandbox.instances.a130168443678744.bootstrap();
return fbjs_dom.eventHandler.call([fbjs_dom.get_instance(this,130168443678744),
function(a130168443678744_event)
{a130168443678744_game.update($FBJS.ref(this).getForm())},130168443678744],new fbjs_event(event));
[/sourcecode]

</div>
As you see I copied all code for three times and left return just for last call. It means my clicks will be tripled and I can register 150 clicks just for 50. It’s good. So let’s copy this code back to onmouseup element it is ready for testing.

Enjoy it!
