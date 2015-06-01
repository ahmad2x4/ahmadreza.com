---
layout: post
title: "How to change mouse cursor icon in windows mobile application"
date: 2009-04-06 12:28
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [.Net, Windows Mobile]
---
<p style="font-family:Calibri;font-size:11pt;margin:0;">Informing user about application state is one of important aspects in user experience. In windows applications mouse icon indicates one of great application states. When an application is busy mouse icon should be changed to wait cursor mode or hourglass icon. In windows application you can achieve this by changing Cursor property of any Control, Usually by changing "this.Cursor" which "this" is current form in application. See example:

<p style="font-family:Calibri;font-size:11pt;margin:0;">&nbsp;

<p style="font-family:'Courier New';font-size:10pt;margin:0;"><span style="color:blue;">this</span>.Cursor = <span style="color:#2b91af;">Cursors</span>.WaitCursor;

<p style="font-family:Calibri;font-size:11pt;margin:0;">&nbsp;

<p style="font-family:Calibri;font-size:11pt;margin:0;">Since control class in compact framework does not have Cursor property, not only developer could not change every control cursor icon but also he could not change form cursor property. So what should developer do in order to indicate application state?

<p style="font-family:Calibri;font-size:11pt;margin:0;">There is a static class in System.Windows.Forms named Cursor and it has a property named "Current" which developer should set it to an appropriate cursor based on application state.

<p style="font-family:Calibri;font-size:11pt;margin:0;">&nbsp;

<p style="font-family:Calibri;font-size:11pt;margin:0;">&nbsp;

<p style="font-family:'Courier New';font-size:10pt;margin:0;">System.Windows.Forms.<span style="color:#2b91af;">Cursor</span> = <span style="color:#2b91af;">Cursors</span>.WaitCursor;

<p style="font-style:italic;font-family:Calibri;font-size:11pt;margin:0;">Or

<p style="font-family:'Courier New';font-size:10pt;margin:0;">System.Windows.Forms.<span style="color:#2b91af;">Cursor</span> = <span style="color:#2b91af;">Cursors</span>.Default;

