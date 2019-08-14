---
layout: post
title: "Hosting WCF Service in IIS and Configuration may be needed"
date: 2009-05-11 15:36
author: ahmadreza
comments: true
categories: [Articles]
tags: [IIS, WCF]
---
<p style="font-family:calibri;font-size:11pt;margin:0;">This article is an attempt to depict steps may needed to configure IIS for hosting WCF application. An important thing that should be considered&#160; is that all user do not face following situations. First of all I try to create a simple WCF application and then I try to publish this simple WCF simple application.
  <p style="font-family:calibri;font-size:11pt;margin:0;">In order to create a WCF application we just need to know ABC of WCF application which are Address, Binding and Contract. (You can download code from [here](https://localhost/be/admin/Pages/resources/WCFConfigIIS/AccountSolution.zip))
  <p style="font-family:calibri;font-size:11pt;margin:0;">First of all I created a blank solution with Visual Studio 2008 by clicking <span style="font-weight:bold;">File &gt; New &gt; Project</span> and selecting <span style="font-weight:bold;">Other Project Types &gt; Visual Studio Solutions &gt; Blank Solution</span>
  <p style="font-family:calibri;font-size:11pt;margin:0;">I created solution named AccountSolution. I added new project named **Accounting **by right clicking on AccountingSolution and selecting <span style="font-weight:bold;">Add &gt; New project</span> menu. In the next step I added reference of System.ServiceModel to project references.
  <p style="font-family:calibri;font-size:11pt;margin:0;">I changed name of class1.cs to IAccount.cs and I added new class named AccountService.cs. Here is code for IAccount and AccountService:
  

IAccount.cs
  <div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:17ead555-b6c1-4857-b87d-e5fd46313bbc" class="wlWriterEditableSmartContent">


[sourcecode language="csharp"]
using System;
using System.Text;
using System.ServiceModel;
 
 
namespace Accounting
{
    [ServiceContract]
    public interface IAccount
    {
        [OperationContract]
        int GetBalance(int accountNumber);
    }
}[/sourcecode]
</pre></div>


    
    &#160;
    


    
    AccountService.cs
    

<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:bc963dd4-8aa3-48d1-b7e5-a6181298f06f" class="wlWriterEditableSmartContent"><pre>
[sourcecode language="csharp"]
using System;
using System.Text;
using System.ServiceModel;
 
namespace Accounting
{
    public class AccountService : IAccount
    {
        #region IAccount Members
 
        public int GetBalance(int accountNumber)
        {
            Random rnd = new Random(1000000);
            return Convert.ToInt32(rnd.Next());
        }
 
        #endregion
    }
}[/sourcecode]
</pre></div>

<p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
    

<p style="font-family:calibri;font-size:11pt;margin:0;">I added new web site named <span style="font-weight:bold;">WCFServiceAccount </span>and added new project reference of <span style="font-weight:bold;">Accounting. </span>Since I added project reference to Accounting project, cs under App_Data folder is no longer needed. So let delete two cs file under App_Data.
    

<p style="font-family:calibri;font-size:11pt;margin:0;">&#160;
    

<p style="font-family:calibri;font-size:11pt;margin:0;">![](https://gkasoq.bay.livefilestore.com/y1plDs_sJrdofjmxAJx5KLB1IRyo8OQaej-EbkO50xRPY-imFFs90p8YcQSZyn_32U0clWyp_dGQuQPHeD8T2wk4VK1hJPUhpQz/DeleteServiceFile.jpg?psid=1)
    

<p style="font-family:calibri;font-size:11pt;margin:0;">I changed Service.svc source to:
    

<p style="font-family:&#039;font-size:10pt;margin:0;"><span style="background:yellow;">&lt;%</span><span style="color:blue;">@</span> <span style="color:#a31515;">ServiceHost</span> <span style="color:red;">Language</span><span style="color:blue;">=&quot;C#&quot;</span> <span style="color:red;">Debug</span><span style="color:blue;">=&quot;true&quot;</span> <span style="color:red;">Service</span><span style="color:blue;">=&quot;Accounting.AccountService&quot;</span><span style="background:yellow;">%&gt;</span>
    

<p class="AraBlogHeader1">Setting Service and end points:
    

<p style="font-family:calibri;font-size:11pt;margin:0;">In order to setService and end points I used WCF Configuration option by right-clicking on web.config.
    

<p style="font-family:calibri;font-size:11pt;margin:0;" class="AraBlogTip">**Tip**: Sometimes &quot;Edit WCF Configuration&quot; does not appear in right-click menu. Once open &quot;WCF Service Configuration&quot; from tools menu and close it, this cause menu appear next time in right click menu.
    

<p style="font-family:calibri;font-size:11pt;margin:0;">I set first service name to <span style="font-weight:bold;">Accounting.AccountService </span>by browsing Accounting.dll in bin directory.
    

<p style="font-family:calibri;font-size:11pt;margin:0;">![](https://public.bay.livefilestore.com/y1pB_4Kia8qCvnVm84Z_mAPdTLsttzeYRSgavEDmipD1J2TQnRIww9aCgWUtkzJgmgjbpYQ4AJ-X6_t_fQTagoemg/WCFConf1.jpg?psid=1)
    

<p style="font-family:calibri;font-size:11pt;margin:0;">And I set first end point contract attribute to <span style="font-weight:bold;">Accounting.IAccount </span>and delete second end point.
    

<p style="font-family:calibri;font-size:11pt;margin:0;">![](https://public.bay.livefilestore.com/y1pNQJ7zjCYsuC64j1u4mzjceRtIOIUwlgh5uoo41IKuJ8l1aAoJRut7RFqtruUwN9nmq2lzezxzG81uNdDPTbGCA/WCFConf2.jpg?psid=1)
    

<p style="font-family:calibri;font-size:11pt;margin:0;">And at last save the WCF configuration and build solution. Before running web site let set WCFServiceAccount as startup project and set Service.svc as start page. Our web site is ready to be executed. After successful execution let host our website to local IIS. Simply website could be hosted by publishing web site to local IIS. If IIS is installed before visual studio 2008 you may not face with any following errors and simply you can jump next step.
    

<p style="font-family:calibri;font-size:11pt;margin:0;">I tried to publish WCFServiceAccount by right clicking on Web site project and selecting &quot;…&quot; then Local IIS but I've got following errors. (My windows was Vista Ultimate)
    

<p style="font-family:calibri;font-size:11pt;margin:0;">To Access local IIS Web sites, you must install the following IIS component.
    



*   <span style="font-family:calibri;font-size:11pt;">IIS 6 Metabase and IIS 6 Configuration Compatibility</span> 
*   <span style="font-family:calibri;font-size:11pt;">ASP.NET</span> 
*   <span style="font-family:calibri;font-size:11pt;">Windows Authentication</span> 


    
    ![](https://public.bay.livefilestore.com/y1pTZqdAli9pFNV3Ue4EJsjyy7yxI73MHpJ2qV7eQ4t1HetvropmejZs60FD3KRdvxQHYuUzqycVYL_ZiuLoGGF1g/LocalIIS.jpg?psid=1)
    

<p style="font-family:calibri;font-size:14pt;font-weight:bold;margin:0;" class="AraBlogHeader1">Configuring IIS to publish a web site (Specially WCF web site):
    

<p style="font-family:calibri;font-size:11pt;margin:0;">To rectify mentioned errors I've found a document in [Technet](https://technet.microsoft.com/en-us/library/bb397374.aspx) for installing IIS6 Metabase and IIS 6 Configuration
    

<p style="font-style:italic;font-family:calibri;font-size:11pt;margin:0;">To install the IIS 6.0 Management Compatibility Components by using the Windows Vista Control Panel
    



1.  <span style="font-style:italic;font-family:calibri;font-size:11pt;">Click </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">Start</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">, click </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">Control Panel</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">, click </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">Programs and Features</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">, and then click </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">Turn Windows features on or off</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">.</span> 
2.  <span style="font-style:italic;font-family:calibri;font-size:11pt;">Open </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">Internet Information Services</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">.</span> 
3.  <span style="font-style:italic;font-family:calibri;font-size:11pt;">Open </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">Web Management Tools</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">.</span> 
4.  <span style="font-style:italic;font-family:calibri;font-size:11pt;">Open </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">IIS 6.0 Management Compatibility</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">.</span> 
5.  <span style="font-style:italic;font-family:calibri;font-size:11pt;">Select the check boxes for </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">IIS 6 Metabase and IIS 6 configuration compatibility</span><span style="font-style:italic;font-family:calibri;font-size:11pt;"> and </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">IIS 6 Management Console</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">.</span> 
6.  <span style="font-style:italic;font-family:calibri;font-size:11pt;">Click </span><span style="font-style:italic;font-family:calibri;font-size:11pt;font-weight:bold;">OK</span><span style="font-style:italic;font-family:calibri;font-size:11pt;">.</span> 

<p style="font-family:tahoma;color:#666666;font-size:8pt;margin:0;">Pasted from &lt;[https://technet.microsoft.com/en-us/library/bb397374.aspx](https://technet.microsoft.com/en-us/library/bb397374.aspx)&gt;
    

<p style="font-family:calibri;font-size:11pt;margin:0;">Following image shows how to install &quot;IIS6 Metabase and IIS 6 Configuration &quot;. Surprisingly you can set other option such as Installing ASP.NET and Windows Authentication.
    

<p style="font-family:calibri;font-size:11pt;margin:0;">![](https://public.bay.livefilestore.com/y1p98ev7XdDN9VOCAtu6E64aR9cfG1_-lRJRZk1r9BzkreaCp8mrRiBVTmCrdjkV88tdxHH926kCoEiQjtb9yEIkw/WindowsFeatures2.jpg?psid=1)
    

<p style="font-family:calibri;font-size:11pt;margin:0;">Alternatively, For installing ASP.NET you can use visual studio command prompt and running aspnet_regiis with -i parameter:
    

<pre>aspnet_regiis -i</pre>


    
    &#160;
    

<pre class="AraBlogHeader1">Publishing WCFServiceAccount to local IIS:</pre>



1.  <span style="font-family:calibri;font-size:11pt;">Right click on WCFServiceAccount select publish from menu 
        
    </span>&#160;![](https://public.bay.livefilestore.com/y1psS3DvG852bzs4I8falh5ecsbtliNY19pE18ZF-zi6SMa9U3wOybrBRvrJ_QbUaTjWZs--NNgKBV-3f_CWAYTPw/PublishWebSite.gif?psid=1) 
2.  Select local IIS option 
3.  Select create new application icon 
4.  Type web application (e.g. WCFServiceAccount) 
      
    &#160;![](https://public.bay.livefilestore.com/y1p8G2-IOJpyazU2KdzmRewVDyxiiFsckgqy4jdyopSmKNsJq3f5uv67yZ1YuKnhkPSEh9EU7wc_iRy3VBExsMQjg/PublishWebSiteIIS.gif?psid=1) 
5.  Now open a browser to test your application &quot;https://localhost/WCFServiceAccount/Service.svc&quot; 
6.  Then select Open button 


    
    If you get following error:
    

<table style="width:80%;" border="1" cellspacing="0" cellpadding="0"><tbody>
    <tr>
      <td>
        
    
    # Server Error in '/WCFServiceAccount' Application.
    
    

        <hr size="1" />

        
    
    # 
    
    

        
    
    ## *Configuration Error*
    
    

        
    
    <span style="font-family:arial, helvetica, geneva, sunsans-regular, sans-serif;"><span style="font-family:arial, helvetica, geneva, sunsans-regular, sans-serif;">**Description: **An error occurred during the processing of a configuration file required to service this request. Please review the specific error details below and modify your configuration file appropriately.</span></span>**Parser Error Message: **Unrecognized configuration section system.serviceModel.
    

        
    
    **Source Error:**
    

        <table style="width:100%;" class=" TE__ShowTableBorders" border="0" bgcolor="#ffffcc"><tbody>
            <tr>
              <td>
                <pre>Line 101:
Line 102:
<span style="color:red;">Line 103: </span>Line 104:
Line 105:

              </td>
            </tr>
          </tbody></table>

        

**Source File: **C:\inetpub\wwwroot\WCFServiceAccount\web.config**&#160;&#160;&#160; Line: **103


        <hr size="1" />

        

<span style="font-family:arial, helvetica, geneva, sunsans-regular, sans-serif;">**Version Information:** Microsoft .NET Framework Version:2.0.50727.3053; ASP.NET Version:2.0.50727.3053 </span><span style="font-family:times new roman;">&#160;</span>

      </td>
    </tr>
  </tbody></table>



OR


<table style="width:80%;" border="1" cellspacing="0" cellpadding="0"><tbody>
    <tr>
      <td>
        <p style="font-family:calibri;font-size:20pt;font-weight:bold;margin:0;">Server Error in Application &quot;Default Web Site/WCFServiceAccount&quot;


        <p style="font-style:italic;font-family:calibri;font-size:18pt;font-weight:bold;margin:0;">HTTP Error 404.3 - Not Found


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Description:</span> The page you are requesting cannot be served because of the Multipurpose Internet Mail Extensions (MIME) map policy that is configured on the Web server. The page you requested has a file name extension that is not recognized, and is not allowed.


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Error Code:</span> 0x80070032


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Notification:</span> ExecuteRequestHandler


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Module:</span> StaticFileModule


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Requested URL:</span> [https://localhost:80/WCFServiceAccount/Service.svc](https://localhost/DerivativeCalculator/Service.svc)


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Physical Path:</span> E:\Projects\Learning\Blog\WCFServiceAcount\Service.svc


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Logon User:</span> Anonymous


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Logon Method:</span> Anonymous


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Handler:</span> StaticFile


        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">Most likely causes:</span>


        

*   <span style="font-family:arial;font-size:11pt;">It is possible that a handler mapping is missing. By default, the static file handler processes all content.</span> 
*   <span style="font-family:arial;font-size:11pt;">The feature you are trying to use may not be installed.</span> 
*   <span style="font-family:arial;font-size:11pt;">The appropriate MIME map is not enabled for the Web site or application. (Warning: Do not create a MIME map for content that users should not download, such as .ASPX pages or .config files.)</span> 

        <p style="font-family:arial;font-size:11pt;margin:0;"><span style="font-weight:bold;">What you can try:</span>


        

*   <span style="font-family:arial;font-size:11pt;">In system.webServer/handlers: </span>
*   <span style="font-family:arial;font-size:11pt;">Ensure that the expected handler for the current page is mapped.</span> 
*   <span style="font-family:arial;font-size:11pt;">Pay careful attention to preconditions (e.g. runtimeVersion, pipelineMode, bitness) and compare them to the settings for your application pool.</span> 
*   <span style="font-family:arial;font-size:11pt;">Pay careful attention to typographical errors in the expected handler line.</span> 
*   <span style="font-family:arial;font-size:11pt;">Please verify that the feature you are trying to use is installed.</span> 
*   <span style="font-family:arial;font-size:11pt;">Verify that the MIME map is enabled or add the MIME map for the Web site using the command-line tool appcmd.exe. </span>

                

    1.  <span style="font-family:arial;font-size:11pt;">Open a command prompt and change directory to %windir%\system32\inetsrv.</span> 
    2.  <span style="font-family:arial;font-size:11pt;">To set a MIME type, use the following syntax: appcmd set config /section:staticContent /+[fileExtension='string',mimeType='string']</span> 
    3.  <span style="font-family:arial;font-size:11pt;">The variable fileExtension string is the file name extension and the variable mimeType string is the file type description.</span> 
    4.  <span style="font-family:arial;font-size:11pt;">For example, to add a MIME map for a file which has the extension &quot;.xyz&quot;, type the following at the command prompt, and then press Enter:</span> 
    5.  <span style="font-family:arial;font-size:11pt;">appcmd set config /section:staticContent /+[fileExtension='.xyz',mimeType='text/plain'] 
                    
Warning: Ensure that this MIME mapping is needed for your Web server before adding it to the list. Configuration files such as .CONFIG or dynamic scripting pages such as .ASP or .ASPX, should not be downloaded directly and should always be processed through a handler. Other files such as database files or those used to store configuration, like .XML or .MDF, are sometimes used to store configuration information. Determine if clients can download these file types before enabling them. </span>
          

        

*   <span style="font-family:arial;font-size:11pt;">Create a tracing rule to track failed requests for this HTTP status code. For more information about creating a tracing rule for failed requests, click </span><a href="https://go.microsoft.com/fwlink/?LinkID=66439"><span style="font-family:arial;font-size:11pt;">here</span></a><span style="font-family:arial;font-size:11pt;">. </span>

        <p style="font-size:11pt;margin:0;"><a href="https://go.microsoft.com/fwlink/?LinkID=62293&amp;IIS70Error=404,3,0x80070032,6000"><span style="font-family:arial;font-weight:bold;">More Information...</span></a><span style="font-family:arial;">This error occurs when the file extension of the requested URL is for a MIME type that is not configured on the server. You can add a MIME type for the file extension for files that are not dynamic scripting pages, database, or configuration files. Process those file types using a handler. You should not allows direct downloads of dynamic scripting pages, database or configuration files. </span>


        <p style="font-size:11pt;margin:0;"><span style="font-family:calibri;">
              
</span><span style="font-family:arial;font-weight:bold;">Server Version Information:</span><span style="font-family:arial;"> Internet Information Services 7.0. </span>

      </td>
    </tr>
  </tbody></table>



You should register service model according to following steps:




1.  <p class="style2">Run a Visual studio Command prompt


      
2.  <p class="style2">Enter &quot;cd c:\windows\Microsoft.Net\Framework\v3.0\Windows Communication Foundation\&quot;<span class="style3"> And press enter</span>


      
3.  Run &quot;servicemodelreg -i&quot; 
