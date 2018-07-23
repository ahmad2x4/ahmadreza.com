---
layout: post
title: "HOW TO propagate WCF Impersonation to COM objects"
date: 2010-09-16 17:19
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [COM, Impersonation, WCF]
---
I was working on a project to write a wrapper on a COM component in WCF. The COM object needs impersonation in some levels to provide certain functionalities to impersonated user. Basically impersonation in .Net application doesn't propagate COM calls. I’ve seen following sentence in[ How To: Use Impersonation and Delegation in ASP.NET 2.0](http://msdn.microsoft.com/en-us/library/ff647404.aspx)


> *The impersonation token does not propagate across threads if you use COM interop with components that have incompatible threading models, or if you use unmanaged techniques to create new threads*


Actually I have a COM component and I want to use this component in my WCF Services. One of classes in COM object has got a property which returns current user. I wrote a console application to test impersonation and see if impersonation does propagate to COM object or not. I wrote a class for impersonation and  it has a method called “ImpersonateUser” that impersonates to passed username and password.  Following code returns different username for .net and COM object. It means it doesn’t propagate to COM object.

``` CSharp 

static void Main(string[] args)
{
	DoWork();
	Console.ReadLine();
}

private static void DoWork()
{
	ImpersonateClass impersonate = new ImpersonateClass();
	using (impersonate.ImpersonateUser("AnotherUser",
					"Domain",
					"passw0rd"))
	{
		COMTESTLib.TestClass obj = new COMTESTLib.TestClass();
		Console.WriteLine(string.Format("COM:{0} .Net:{1}", obj.CurrentUserName,
			System.Security.Principal.WindowsIdentity.GetCurrent().Name));
	}
}
//Returns:COM:DOMAIN\CurrentUser .Net:DOMAIN\AnotherUser&lt;/pre&gt;

```


But I used CoInitializeSecurity to initialize security with impersonation and cloaking. CoInitializeSecurity should be called before any marshalling so I called CoInitializeSecurity as first function call. 

``` CSharp 

[DllImport("ole32.dll")]
public static extern int CoInitializeSecurity(IntPtr pVoid, int
cAuthSvc, IntPtr asAuthSvc, IntPtr pReserved1, RpcAuthnLevel level,
RpcImpLevel impers, IntPtr pAuthList, int dwCapabilities, IntPtr
pReserved3);

static void Main(string[] args)
{
	int result = CoInitializeSecurity(IntPtr.Zero, -1,
	IntPtr.Zero, IntPtr.Zero,
	RpcAuthnLevel.Connect, RpcImpLevel.Impersonate,
	IntPtr.Zero, Convert.ToInt32(EoAuthnCap.DynamicCloaking), IntPtr.Zero);

	DoWork();
	Console.ReadLine();
}

private static void DoWork()
{
	ImpersonateClass impersonate = new ImpersonateClass();
	using (impersonate.ImpersonateUser("AnotherUser",
					"domain",
					"passw0rd"))
	{
		COMTESTLib.TestClass obj = new COMTESTLib.TestClass();
		Console.WriteLine(string.Format("COM:{0} .Net:{1}", obj.CurrentUserName,
			System.Security.Principal.WindowsIdentity.GetCurrent().Name));
	}
}
//Returns:COM:DOMAIN\AnotherUser .Net:DOMAIN\AnotherUser&lt;/pre&gt;

```

Somebody suggested using built-in impersonation method for WCF so I changed my service to following code by setting [OperationBehavior(Impersonation = ImpersonationOption.Required)] and some changes in Web.Congif to make it work. 

``` CSharp 
[OperationBehavior(Impersonation = ImpersonationOption.Required)]
public string GetUserNames()
{
	return DoWork();
}

private string DoWork()
{
	string result;

	COMTESTLib.TestClass obj = new COMTESTLib.TestClass();
	result = string.Format("COM:{0} .Net:{1}", obj.CurrentUserName,
		System.Security.Principal.WindowsIdentity.GetCurrent().Name);

	return result;
}

```
I passed user credentials from client for impersonation.

``` CSharp 
static void Main(string[] args)
{
	ServiceReference.ServiceImpersonateClient client = new ServiceReference.ServiceImpersonateClient();
	client.ClientCredentials.Windows.AllowedImpersonationLevel = System.Security.Principal.TokenImpersonationLevel.Delegation;
	client.ClientCredentials.Windows.ClientCredential.Domain = "Domain";
	client.ClientCredentials.Windows.ClientCredential.UserName = "AnotherUser";
	client.ClientCredentials.Windows.ClientCredential.Password = "Passw0rd";

	string str = client.GetUserNames();
	Console.WriteLine(str);
	Console.ReadLine();
	client.Close();
}
// Returns COM:DOMAIN\CurrentUser .Net:DOMAIN\AnotherUser

```

But it returns different usernames which means impersonation of WCF does not propagate to COM object.

Ok I posted this issue in [WCF Forum](http://social.msdn.microsoft.com/Forums/en-US/wcf/thread/ec442221-f750-4253-ac12-b41ab6e4ba1b/#c7b7fa20-b9b7-4576-9ca5-c2c21c6cf909) and I’ve got a simple response but it gave me a good clue. Allen Chen (The moderator) suggested me to use self-hosting for WCF instead of IIS hosting.

Self-hosted WCF was great idea!  Actually it works on self-hosted. But my WCF service should be unattended so that I created a managed service for WCF hosting. To make it work I’ve created three projects. One WCF service library which is used in managed service application. And the third one is a console application for testing. I’ve called CoInitializeSecurity on constructor of my service class as following


``` CSharp 

private ServiceHost serviceHost;
 public WCFServiceHost()
 {
  int result = CoInitializeSecurity(IntPtr.Zero, -1,
  IntPtr.Zero, IntPtr.Zero,
  RpcAuthnLevel.Connect, RpcImpLevel.Impersonate,
  IntPtr.Zero, Convert.ToInt32(EoAuthnCap.DynamicCloaking), IntPtr.Zero);
  //EventLog.WriteEntry(result.ToString());
  InitializeComponent();
 }

 protected override void OnStart(string[] args)
 {
  if (serviceHost != null)
  {
  serviceHost.Close();
  }

  // Create a ServiceHost for the CalculatorService type and
  // provide the base address.
  serviceHost = new ServiceHost(typeof(ServiceImpersonate));

  // Open the ServiceHostBase to create listeners and start
  // listening for messages.

  serviceHost.Open();

 }

 protected override void OnStop()
 {
  if (serviceHost != null)
  {
  serviceHost.Close();
  serviceHost = null;
  }
 }

```

]It doesn’t work with built in WCF impersonation. But I used my impersonation class inside the service class to impersonate to desired user and it works.

``` CSharp 
public string GetUserNames()
 {
  ImpersonateClass impersonate = new ImpersonateClass();

  using (impersonate.ImpersonateUser("AnotherUser",
         "Domain",
          "passw0rd"))
  {
   return DoWork();
  }
 }

 private string DoWork()
 {
  string result;

   COMTESTLib.TestClass obj = new COMTESTLib.TestClass();
   result = string.Format("COM:{0} .Net:{1}", obj.CurrentUserName,
       System.Security.Principal.WindowsIdentity.GetCurrent().Name);
  return result;
 }

```

In the console application I simply called my service like following code:

``` CSharp
ServiceReference.ServiceImpersonateClient client = new ServiceReference.ServiceImpersonateClient();
  string str = client.GetUserNames();
  Console.WriteLine(str);
  client.Close();
  Console.ReadLine();
// Returns COM:DOMAIN\ AnotherUser .Net:DOMAIN\AnotherUser

```

>*When you host WCF in IIS the process is spawned by IIS and before your code running code managed by IIS will be run in the process, which is almost out of your control. CoInitializeSecurity thus might be called before you calling it.*


