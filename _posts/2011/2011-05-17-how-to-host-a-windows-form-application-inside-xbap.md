---
layout: post
title: "How to host a windows form application inside XBAP"
date: 2011-05-17 09:29
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [XBAP]
---
*I am writing a series of posts about WPF Browser Application, XBAP  and configuration tips. I'm going to host this application in IIS 5.1 and I developed them in .Net 3.5. The reason I have chosen IIS5.1 and .Net 3.5 is because of challenges I had in one of my recent projects. Configuring this type of projects is different in other versions of IIS and .Net frameworks and they are not is subject of this post series.*


1.  [How to create a simple Browser Enables WPF application](http://ahmadrezaa.wordpress.com/2011/05/12/how-to-create-a-simple-browser-enabled-wpf-application/)
2.  How to host a windows form application inside XBAP
3.  [How to sign the XBAP with your own certificate](http://ahmadreza.com/2011/05/20/how-to-sign-the-xbap-with-your-own-certificate/)
<div>Imagine we have an existing windows form application or you are developing a windows form application. The question is how you can make your windows application browser enabled. The answer is Browser Enabled WPF application. But usually when we create a WPF application we have to use WPF elements inside Xaml. "WindowsFormsIntegration" will help you to host your windows form application inside your WPF.</div>
<div>For rest of this post I consider you already have previous [post](http://ahmadrezaa.wordpress.com/2011/05/12/how-to-create-a-simple-browser-enabled-wpf-application/) sample, because I'm going to update the same project. Lets open the SimpleBrowserApplication and add a reference to "WindowsFormsIntegration". As the second step I want to add new project to our solution of type "Windows Form Application" and I call it "WinFormSample".</div>
<div>**Note: **Make sure when you are creating windows form application you selected .Net 3.5 because we are going to reference this project inside WPF project and they should be compatible.</div>
<div>![](http://gkasoq.bay.livefilestore.com/y1pTR0DEBuQvABWLQzd6YhAQZuWSbC9O6dOBDZc0k0ikuOoYd9DcKUc8nMOo1HM4y7M1aeLGHg3mlzj61BTkYuIIVH73rTHoSwf/01WinProject.png?psid=1)</div>
<div>That is how our solution looks like after adding windows form application and couple of simple controls on it. Next step we have to reference WinFormSample in SimpleBrowser application. Now our WPF application has got two more references which are "WindowsFormsIntegrations" and "WinFormSample". Now we have to change Page1.xaml and put the following StackPanel instead of previous &lt;Grid&gt;</div>
[sourcecode language="xml"]
&lt;StackPanel x:Name=&quot;stackPanel&quot;&gt;
&lt;/StackPanel&gt;
[/sourcecode]

Actually this is a place holder for WindowsFormsHost that we are going to place in main form. For next step we need add System.Windows.Froms reference to WBP project. Then  we have to change page1.xaml code as follow.

[sourcecode language="csharp"]
public partial class Page1 : Page
{
	private readonly Form1 mainForm = new Form1();
	WindowsFormsHost windowsFormsHost;

	public Page1()
	{
		InitializeComponent();
		AddWindowsForm();
	}
	private void AddWindowsForm()
	{
		windowsFormsHost = new WindowsFormsHost();

		stackPanel.Children.Add(windowsFormsHost);

		// If you don't write this line you'll get &quot;The child control cannot be a top-level form&quot; exception
		mainForm.TopLevel = false;
		windowsFormsHost.Child = mainForm;
	}

}
[/sourcecode]

We created a WindowsFormHost and added this control into stackPanel Child list and set the child property of windowsFormsMost to mainForm which is already instantiated of Form1.

One of important thing is setting mainForm.TopLevel to false. Because if you don't do that you will get an exception and if you dive into innerexeptions you will find out that main reason is System.ArgumentException: The child control cannot be a top-level form.

If you run this application you'll see following browser window which hosts Form1.

![](http://gkasoq.bay.livefilestore.com/y1pghRlG0xxzMlDb6UuxaxWxJIBslm4JuJx94sfUGjwQ_B65isf3BvSQXnDmkUv1LeLqc3Y35rgRKRKmh0zuuGUzynNDfxp1AXD/02WinFormInBrowser.png?psid=1)

The point is when you run this application from visual studio it runs in My Computer Zone so there is no problem for security. According Microsoft document "[WPF Partial Trust Security](http://msdn.microsoft.com/en-us/library/aa970910.aspx)" section "Partial Trust Programming" when you run WPF application which requires full trust and current zone is "My Computer" behavior is "Automatic full trust" and for getting full trust no action is required.

But if you publish this project and try to browse this application you will get Trust Not Granted error. Because application will request for full trust and it fails with "Trust Not Granted". In order to get full trust is signing XBAP with certificate.

![](http://gkasoq.bay.livefilestore.com/y1pnin42vXvjTHQJ4c0_gDomTTngNGWiMBNIholYltYHRJ6rLN5jJBlD1jzXGP8Ynlg30JtoWPAqtUiz9XwM_HAwpMK5rVy7wYh/03TrustNotGranted.png?psid=1)

In the next post we will see how to sign XBAP with your own certificate and make it work.

The source code of this application is also available you can download it [here](http://gkasoq.bay.livefilestore.com/y1pxaDZrXteX6D_gi2J4LOV9id05FJDax0nnSWPCp0-QXe2hd0glJoGIlxVkTt-viR292pddb7rs3MomJimgtH_MpmTMc3XuDS2/SimpleBrowserApplication.zip?download&amp;psid=1)
