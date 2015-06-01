---
layout: post
title: "How to create a simple Browser Enabled WPF application"
date: 2011-05-12 14:59
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [XBAP]
---
*I am writing a series of posts about WPF Browser Application, XBAP  and configuration tips. I'm going to host this application in IIS 5.1 and I developed them in .Net 3.5. The reason I have chosen IIS5.1 and .Net 3.5 is because of challenges I had in one of my recent projects. Configuring this type of projects is different in other versions of IIS and .Net frameworks and they are not is subject of this post series.*


1.  How to create a simple Browser Enables WPF application
2.  [How to host a windows form application inside XBAP](http://ahmadreza.com/2011/05/17/how-to-host-a-windows-form-application-inside-xbap/)
3.  [How to sign the XBAP with your own certificate](http://ahmadreza.com/2011/05/20/how-to-sign-the-xbap-with-your-own-certificate/)
Firstly run Visual Studio 2010 and select new project from file menu

![](http://gkasoq.bay.livefilestore.com/y1pTyvopu7LF2C48XH1rU6cCIppkTTSRI2zbiG4IymcFYNyDvDYOAuiDPGP_BbwZCwfe_Msl-NT15LmvikS4opJ6IjFau3VEDJa/01ProjectType.png?psid=1 "WPF Browser Application Template")

To create simple WPF browser application you need to select "WPF Browser Application" template from project templates. Once project template is created open Page1.xaml Xaml code and change Grid into following code

[sourcecode language="XML"]
    &lt;Grid&gt;
        &lt;Rectangle
            Fill=&quot;#33CC66&quot;
            Width=&quot;2in&quot;       Height=&quot;1in&quot;
            Canvas.Top=&quot;25&quot;          Canvas.Left=&quot;50&quot;
            StrokeThickness=&quot;6px&quot; Stroke=&quot;Orange&quot; /&gt;
    &lt;/Grid&gt;
[/sourcecode]

This Xaml code will create a simple rectangle with border and if you run this application, It will show following shape inside your browser.

![](http://gkasoq.bay.livefilestore.com/y1pno2mRmcxK60x2XCCUpEH8AdIRl57EHzXxvUEScNJyTbdTGqN0TRpA25pd4ooBhFy-cQ9A7mOlpMz081IkhZQlhg9rhsmFeO8/02BrowserRectangle.png?psid=1)

**Deploying a WPF application**

There are multiple ways to do that. Simply you can publish your application using visual studio. Right-click on project and select publish menu. Publish wizard is displayed. Follow the steps until the end of publish steps.

You must follow Microsoft instruction for [How to: Configure IIS 5.0 and IIS 6.0 to Deploy WPF Applications](http://msdn.microsoft.com/en-us/library/ms752346.aspx) to configure your server and client requires Internet Explorer plus .Net Framework to run this application.

Basically internet application which runs inside the browsers should have restricted access to critical resources. It means WPF browser application -By default- should respect to these restrictions so that client can make sure there is no harm to execute this application. Browser Enabled application by default is marked as partially trusted and ClickOnce security setting is set to Internet Zone so that your application will be running on the client browser without any problem.

![](http://gkasoq.bay.livefilestore.com/y1p2yMA3hrhMwt0iBWIUIIxE0kfNCGovL0YPJE27avS6fATUyXgPmATXdE8EH0G4w5m-wsW9E84rBfviY_Vt0AFxzHer8bMiQrn/03ProjectSecurity.png?psid=1)

As you see "Enable ClickOnce security settings" and "This is a partial trust application" are ticked by default and and ClickOnce manifests is signed by a temporary key. Which means on one your application will be restricted to security permission which is fully described in this [document](http://msdn.microsoft.com/en-us/library/aa970910.aspx). On the other hand your application will be executed on client browser without any other additional configuration.

![](http://gkasoq.bay.livefilestore.com/y1pKjv-M2LBhfxzwWwSd_L29SOoyeqW5BcSkFwkyOQmjzksBojy0rFiaCALaIDovh5S7ieC6j-sV7cqiPiULDVI3idjIL9e3EwZ/04InternetZone.png?psid=1)

Above picture shows that you can run This simple WPF Browser Enabled application in "Internet Zone".

The main reason of writing this post series for me is hosting windows form application inside XBAP application and running inside browser. In next post we will see how to host an existing windows form application within a WPF Browser application.

*Sample project is also available you can download it [here](http://gkasoq.bay.livefilestore.com/y1pMj5rWwxWH0xh978PfXKv51b9cKKKJ2zKKHaycMMZbHVm6kHqLQbIOej8nUumLrJYZlMQbeCSbDSxCQier38MWVQvihSHve5O/SimpleBrowserApplication.zip?download&amp;psid=1)*
