---
layout: post
title: "TickCount instead of millisecond in compact framework"
date: 2009-03-30 13:42
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [.Net Compact Framework, Windows Mobile]
---
<p style="font-family:calibri;font-size:11pt;margin:0;">After long absence I'm back now. Recently I wanted to develop an application for windows mobile with .Net Compact framework. In this application I need to measure elapsed time in millisecond resolution. First of all like everyone I tried to use Millisecond property of DateTime.Now static class. But it won't work at all in .Net Compact framework even in version 3.5!
  <p style="font-family:calibri;font-size:11pt;margin:0;">If you want to use a timer with millisecond resolution you have to use TickCount property of Environment class.
  <p style="font-family:calibri;font-size:11pt;margin:0;">
  <p style="font-family:calibri;font-size:11pt;margin:0;">
  <p style="font-family:&#039;color:green;font-size:10pt;margin:0;">//Will be zero.
  <p style="font-family:&#039;font-size:10pt;margin:0;">lblMillisecondVal.Text = <span style="color:#2b91af;">DateTime</span>.Now.Millisecond.ToString();
  <p style="font-family:&#039;color:green;font-size:10pt;margin:0;">//Total milliseconds elapsed since the system started.
  <p style="font-family:&#039;font-size:10pt;margin:0;">lblTickCountVal.Text = <span style="color:#2b91af;">Environment</span>.TickCount.ToString();
  <p style="font-family:&#039;font-size:10pt;margin:0;">
  <p style="font-family:calibri;font-size:11pt;margin:0;">[Here](https://gkasoq.bay.livefilestore.com/y1p-j36TiPUqAUB6w5kYE05nlr4sJQ6zrqwpr1ggB654bL0E7JHIehasbHjEaireeiTg9vR_nxCaAskngmyKX-o1Q/TickCountTest.zip?download&amp;psid=1) is an example of using TickCount Instead of Milliseconds

