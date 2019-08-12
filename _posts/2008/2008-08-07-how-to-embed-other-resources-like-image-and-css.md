---
layout: post
title: "How to embed other resources like image and css"
date: 2008-08-07 03:44
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [.Net, VS 2005, Web Controls]
---
<p style="font-family:calibri;font-size:11pt;margin:0;">Embedding other resources like Cascade style sheets and images is possible as well as JavaScript file. I wrote &quot;[How to embed JavaScript file in an assembly](How-to-embed-JavaScript-file-in-an-assembly.aspx)&quot; in previous post. For embedding images you have to do same way with minor changes in accessing resource. You should add you file into your project and
  <p style="font-family:calibri;font-size:11pt;margin:0;">change the value of &quot;Build Action&quot; property to &quot;Embedded Resource&quot;.&#160; In Assembly info file you should add following line for images into AssemblyInfo.cs file:
  <p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
  <p style="font-family:calibri;font-size:11pt;margin:0;">[assembly: System.Web.UI.<span style="color:#2b91af;">WebResource</span>(<span style="color:#a31515;">&quot;EmbededJSControl.Logo.GIF&quot;</span>, <span style="color:#a31515;">&quot;img/gif&quot;</span>)]
  <p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
  <p style="font-family:calibri;font-size:11pt;margin:0;">And following line for css file:
  <p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
  <p style="font-family:calibri;font-size:11pt;margin:0;">[assembly: System.Web.UI.<span style="color:#2b91af;">WebResource</span>(<span style="color:#a31515;">&quot;EmbededJSControl.Stylesheet.css&quot;</span>, <span style="color:#a31515;">&quot;text/css&quot;</span>)]
  <p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
  <p style="font-family:calibri;font-size:11pt;margin:0;">To use image resource in your pages you should add following code. In case you use resource in same project you could use this.GetType() as first parameter otherwise you should create an instance of control from project in which you included resource and get type of that control.
  <p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
  <p style="font-family:&#039;font-size:10pt;margin:0;"><span style="color:#2b91af;">Image</span> img = <span style="color:blue;">new</span> <span style="color:#2b91af;">Image</span>();
  <p style="margin:0;"><span style="font-family:&#039;font-size:10pt;">img.ImageUrl = Page.ClientScript.GetWebResourceUrl(</span><span style="font-family:&#039;color:blue;font-size:10pt;">this</span><span style="font-family:&#039;font-size:10pt;">.GetType(), </span><span style="font-family:&#039;color:#a31515;font-size:10pt;">&quot;</span><span style="font-family:calibri;color:#a31515;font-size:11pt;">EmbededJSControl.Logo.GIF</span><span style="font-family:&#039;color:#a31515;font-size:10pt;">&quot;</span><span style="font-family:&#039;font-size:10pt;">);</span>
  <p style="margin:0;"><span style="font-family:&#039;font-size:10pt;"></span>
  <p style="font-family:calibri;font-size:11pt;margin:0;">For including stylesheet use following code:
  <p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
  <p style="font-family:&#039;font-size:10pt;margin:0;"><span style="color:blue;">string</span> cssLink = <span style="color:#a31515;">&quot;&quot;</span>;
  <p style="margin:0;"><span style="font-family:&#039;color:blue;font-size:10pt;">string</span><span style="font-family:&#039;font-size:10pt;"> location = Page.ClientScript.GetWebResourceUrl(</span><span style="font-family:&#039;color:blue;font-size:10pt;">this</span><span style="font-family:&#039;font-size:10pt;">.GetType(), </span><span style="font-family:&#039;color:#a31515;font-size:10pt;">&quot;</span><span style="font-family:calibri;color:#a31515;font-size:11pt;">EmbededJSControl.Stylesheet.css</span><span style="font-family:&#039;color:#a31515;font-size:10pt;">&quot;</span><span style="font-family:&#039;font-size:10pt;">);</span>
  <p style="font-family:&#039;font-size:10pt;margin:0;"><span style="color:#2b91af;">LiteralControl</span> css = <span style="color:blue;">new</span> <span style="color:#2b91af;">LiteralControl</span>(<span style="color:#2b91af;">String</span>.Format(cssLink, location));
  <p style="font-family:&#039;font-size:10pt;margin:0;">Page.Header.Controls.Add(css);
  <p style="font-family:&#039;font-size:10pt;margin:0;">&#160;
  <p style="font-family:&#039;font-size:10pt;margin:0;">Complete sample code is available [here](https://gkasoq.bay.livefilestore.com/y1peTj3VAMB0CgQFtkrQCdRlXxnjvPJ5t6Pzku8YfWyuUSFx9bGusFmlFhOIFBGBtVtjShgUOybQp1UYCkvcTEkjw/EmbededJSSource2.zip?download&amp;psid=1)
  

<a href="https://www.dotnetkicks.com/kick/?url=http%3a%2f%2fahmadreza.com%2fgf%2fblog%2fhow-to-embed-other-resources-like-image-and-css%2f">![kick it on DotNetKicks.com](https://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fahmadreza.com%2fgf%2fblog%2fhow-to-embed-other-resources-like-image-and-css%2f)</a>

