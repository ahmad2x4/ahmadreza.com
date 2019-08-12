---
layout: post
title: "Quick Note: Memory Leak Or Missing Configuration "
date: 2011-11-14 17:24
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [SQL Server]
---
Imagine you have multiple instances of SQL server on your server and one of them is using almost all of available memory and second one is facing memory shortage. What are possible Cause for this symptom. Does one of application have memory leak?

The answer is **NO**, When you install multiple instances of SQL server on a single server you have to consider memory allocation for each instances because windows does not balance memory across applications with the memory notification.

The first instance with a work load will  used huge portion of memory (Especially when you have actual data - not testing - on that instance).

Three approaches are available for [Server Memory Option](https://msdn.microsoft.com/en-us/library/ms178067.aspx) documented in the section "Sunning Multiple Instances of SQL server" and if you have selected third one which is "Do nothing", you might have same problem.



> *   *Do nothing (not recommended). The first instances presented with a workload will tend to allocate all of memory. Idle instances or instances started later may end up running with only a minimal amount of memory available. SQL Server makes no attempt to balance memory usage across instances. All instances will, however, respond to Windows Memory Notification signals to adjust the size of their buffer pools. As of Windows Server 2003 SP1, Windows does not balance memory across applications with the Memory Notification API. It merely provides global feedback as to the availability of memory on the system.*



