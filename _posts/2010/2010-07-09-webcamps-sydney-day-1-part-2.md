---
layout: post
title: "WEBCAMPS Sydney Day 1 / Part 2"
date: 2010-07-09 05:59
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [Australia, Data Dynamics, jQuery, MVC, Sydney, WEBCAMPS]
---
<p class="MsoNormal">As first day continued Scott and James tried to demonstrate Microsoft ASP.net 4.0 and MVC 2. Same question about ASP.Net was asked and was answered. Fellows was worried about replacement of ASP.Net with MVC 2 and as it was answered and like before Scott and James defined that ASP.Net is not going to be replaced by MVC. It is just a new approach of implementing web applications and depending on website structure and content websites could either be implemented by ASP.Net 4.0 or MVC 2.
  <p class="MsoNormal">Couple of new project templates has been added to Visual Studio 2010 and in web. One of them is “ASP.NET Web Application”. When a new ASP.NET Web Application is created visual studio automatically creates folder and files which are needed for a simple application. This type of project unlike
  

previous web application is not an empty project and contains necessary files and folders to create a simpl
  

e application.
  <p style="text-align:center;" class="MsoNormal">![](https://gkasoq.bay.livefilestore.com/y1pLnVOG0XwybOiItGKO0xDFYXBdbJJMW4wIfltcM7jkOkr7r-lTYbSi0JH2xkh_xNTUat4zWPIg487cLqJTGVhZ3uwXaMObIgx/image1.jpg?psid=1)
  <p style="text-align:center;" class="MsoNormal" align="center"><span></span>
  <p class="MsoNormal">This template includes Microsoft ASP.NET Membership Provider and if SQL Server Express is installed on developer machine this type of project is automatically configured for ASP.NET Membership provider which connects to SQL Express. This type of Web application is similar to ASP.Net application because the theme and style looks like default theme of ASP.Net MVC Application. Good news is Micros
  

oft has added jQuery to ASP.Net web application and ASP.Net MVC 2 Web application templates.
  <p style="text-align:center;" class="MsoNormal">![](https://public.bay.livefilestore.com/y1p0CyxJbDlGQX1TGsh9xYGAX5LM2BabKVTdC05NH5Xp8NvE50SN9b1BlluEvfUTBRPVzfNTVlwh9B9jC9kmR83oA/image2.jpg?psid=1)
  <p style="text-align:center;" class="MsoNormal" align="center"><span></span>
  <p class="MsoNormal">I think one of best features of MVC is its testability and on top of that you can change test framework of testing if you want. I’ve find some good articles and posts about adding customized test framework for testing MVC 2 projects. [How to: Add a Custom MVC Test Framework in Visual Studio](https://msdn.microsoft.com/en-us/library/dd381614.aspx) and a post about [NUnit Template for ASP.NET MVC 2](https://www.dalsoft.co.uk/blog/index.php/2009/11/17/nunit-templates-for-asp-net-mvc-2-0-preview-2/) .
  <p style="text-align:center;" class="MsoNormal">![](https://public.bay.livefilestore.com/y1p0CyxJbDlGQXtnu2vYWW5fll5DdKfq78g262MRz5l3vWRhKV4dJ0xXMvi1-S5gzEDBpwwgMbacdMtab3GYGn8SQ/image3.png?psid=1)
  <p style="text-align:center;" class="MsoNormal" align="center"><span></span>
  <p class="MsoNormal">Presenters created a new MVC2 Web application and for beginning they created a project without database just to keep it simple. View, Mode was discussed in this session and they talked about partial views and templates. I wanted to describe creating templates in MVC 2 but I found a good sample in Microsoft that helps to understand how to create templates in MVC. Following is link to [EditorFor](https://msdn.microsoft.com/en-us/library/ee407414.aspx?appId=Dev10IDEF1&amp;l=EN-US&amp;k=k(%22SYSTEM.WEB.MVC.HTML.EDITOREXTENSIONS.EDITORFOR%60%602%22)&amp;rd=true) extension method. On the bottom of page you can find a link to Template Helper Sample.
  <p class="MsoNormal">For describing model part of MVC a virtual project was defined named **Plan My Night**. It was a simple application to plan night. They described model and controller by creating a simple database and entity model to implement **Plan My Night**. During implementation o f **Plan My Night** they described partial views editor templates. For editor templates there is a good sample to create an editor template for datetime inputs called [MVC2 Editor Template with date and time](https://geekswithblogs.net/michelotti/archive/2010/02/05/mvc-2-editor-template-with-datetime.aspx).
  <p class="MsoNormal">They combined sample with newly introduced feature called jQuery Templates Data Linking. You can find a complete sample in [ScottGu’s Blog](https://weblogs.asp.net/scottgu/archive/2010/05/07/jquery-templates-and-data-linking-and-microsoft-contributing-to-jquery.aspx). Another cool feature of visual studio which is really useful is Dynamic JavaScript Intellisense in VS 2010. Not only client items in win form can be identified by intellisense of VS 2010 but also items which are being created by developer during development can be identified in VS 2010 like following picture. 1000 items with TestDev prefix is added to window object. Exactly after while command you all TestDiv0 to TestDiv1000 is added to suggestion list.
  <p style="text-align:center;" class="MsoNormal">![](https://public.bay.livefilestore.com/y1ptBeALZuZu6CwPxpnLlyhupBowCMtPGt0zbF-B1dV_0y7H0g4YeYlZClbjDim0T5IVJjUBwLI3rBd7QwJr3yRLQ/image4.jpg?psid=1)
  <p style="text-align:center;" class="MsoNormal" align="center"><span></span>
  <p class="MsoNormal">During the presentation Scott wanted to use a trick to cause a crash for visual studio 2010 (It was on purpose). He changed “while” condition to an infinite while condition and he expected VS to crash when suggestion list was being opened, but visual studio is clever enough and calculates list until certain amount of millisecond and if it’s going to be time consuming visual studio just ignores rest of computing and jump over next statement. There is a good post in [ScottGu’s Blog](https://weblogs.asp.net/scottgu/archive/2009/10/22/vs-2010-code-intellisense-improvements-vs-2010-and-net-4-0-series.aspx) about JS Intellisense and its improvements.
  <p class="MsoNormal">Other feature which was discussed in webcamps was about Web.config transformation and packaging and deployment of web application SQL server. There is a good clip about Web.Config transformation in [channel9](https://channel9.msdn.com/shows/10-4/10-4-Episode-10-Making-Web-Deployment-Easier/) about Web Deployment and Web.Config.
  <p class="MsoNormal"><span>&#160;</span>Rest of presentation was about IIS extensions which are available on [https://www.iis.net/download](https://www.iis.net/download) and can be downloaded and installed by Web Platform Installer. Among those extensions [SEO Toolkit](https://www.iis.net/download/SEOToolkit) was fantastic which was shown by Scott.
  <p class="MsoNormal">At the end of presentation fellows were asked to pitch their ideas for next day. Following are some of projects and people voted for them
  

.
  <p class="MsoNormal">My Noisy Neighbour (I voted this project)
  <p class="MsoNormal">RaceDay Commander
  <p class="MsoNormal">MongoDB Management Studio
  <p class="MsoNormal">Due Date – lightweight task-tracking for uni students
  <p class="MsoNormal">Address Book Lookup
  <p class="MsoNormal">I voted for first project “My Noisy Neighbour” and in second day I had to join to a group of people to develop this web application based on what we’d learned in first day. In the next post I’m going to describe what was our project was and what had happened.

