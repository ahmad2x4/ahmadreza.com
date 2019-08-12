---
layout: post
title: "How to sign the XBAP with your own certificate"
date: 2011-05-20 09:30
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [XBAP]
---
*I am writing a series of posts about WPF Browser Application, XBAP  and configuration tips. I'm going to host this application in IIS 5.1 and I developed them in .Net 3.5. The reason I have chosen IIS5.1 and .Net 3.5 is because of challenges I had in one of my recent projects. Configuring this type of projects is different in other versions of IIS and .Net frameworks and they are not is subject of this post series.*


1.  [How to create a simple Browser Enables WPF application](https://ahmadrezaa.wordpress.com/2011/05/12/how-to-create-a-simple-browser-enabled-wpf-application/)
2.  [How to host a windows form application inside XBAP](https://ahmadreza.com/2011/05/17/how-to-host-a-windows-form-application-inside-xbap/)
3.  How to sign the XBAP with your own certificate
<div>**Note: Making a browser enabled application as full trust according to this method is not completely secured. This can be used for testing purposes in testing environments. Please select proper certificates and known trusted root certification authorities.**</div>
Before starting I think its better to have same understanding of file extension that we are going to talk about.

**.cer file: **Apublic key which is given by Certificate Authority

**.pvk file: **This file is your private key and should keep it confidential

**.pfx file: **This is a Personal Information Exchange file and again you should keep it confidential because it contains

We have created a simple WPF Browser application and a simple windows application which is hosted inside the XBAP application. When we created WPF application Visual Studio automatically create a .pfx (Which is used for signing ClickOnce manifest).

To create your own certificate you need to follow these steps:

**Step 1:** Creating your key pairs (Public and Private)

open Visual Studio Command Prompt (2010) and then goto your application path and type following command

[sourcecode language="Text"]

makecert -n &quot;CN=Your Company Name&quot; -r -sv Key.pvk Key.cer

[/sourcecode]

A password dialog box will be displayed and you set your own password. This command creates two files one private key and one certificate.

**Step2:** Then you need to create PFX file which is used for signing ClickOnce manifest and contains both private and public key.

[sourcecode language="Text"]

pvk2pfx.exe -pvk Key.pvk -spc Key.cer -pfx KeyPFX.pfx -po [password]

[/sourcecode]

Put your own password as [password] and this command will create a PFX file

**Step 3:** Back to the solution explorer delete "SimpleBrowserApplication_TemporaryKey.pfx" and goto Application property page and select signing tab. Click on "Select from file" and select the PFX file you have just created.

<a href="https://gkasoq.bay.livefilestore.com/y1pFsKJJV5SbKmeWkxSC7iJ-g2laeGSY_VkVN5ZGWCPeku6dJKdnN4sDdWnqkM3zLpSX-39bZVQE0MS7sb7ZGyYetm_jBRPfPoO/01SignTheClickOnce.png?psid=1">![](https://gkasoq.bay.livefilestore.com/y1pFsKJJV5SbKmeWkxSC7iJ-g2laeGSY_VkVN5ZGWCPeku6dJKdnN4sDdWnqkM3zLpSX-39bZVQE0MS7sb7ZGyYetm_jBRPfPoO/01SignTheClickOnce.png?psid=1)</a>

**Step 4: **Just like [before](https://ahmadreza.com/2011/05/12/how-to-create-a-simple-browser-enabled-wpf-application/)  publish it to you server.

**Step 5: **Give certificate to the client and register the certificate on the client machine. To do this double-clicking on .cer file. You will see following window. Click on Install Certificate button.

![](https://public.bay.livefilestore.com/y1pZFqlYjWUc6WMxKhEZnvPIObkf7syrmkj_3GrXsmwEKe44a3ON90H9L8IQpQ612HM2gk0BTrlQKBnZCZSwytECg/02InstallCertificate.png?psid=1)

Follow installation wizard and click Next on the first window.

![](https://public.bay.livefilestore.com/y1plD0sqjQ4lZqiA_weDaMseqiroigdUv6i-Dj8N7ClONOPN6NPPOHYEODk7yMJyVCI79cK5R4bmnArBfQ55dK0Hw/03CertificateImportWizard.png?psid=1)

In this window select "Place all certificates in the following store"  and then select "Browse..." button.

*![](https://public.bay.livefilestore.com/y1plD0sqjQ4lZqx_94u5tjllRK8ghYlHog30qKU-ajAV28ST8V0gFVMzLqnF7eVbInn0RtvugNkoBiu24Q7lrTsCw/04SelectCertificateStore.png?psid=1)*

**In this window select "Trusted Publishers" and then click Ok. Select "Next" previous windows and then select finish.

**Step 6: **Redo the step 5 but this time select "Trusted Root Certification Authorities" as the certificate store.

![](https://public.bay.livefilestore.com/y1puBVImTKJcP2bDTJu6iTMgASmHIEyxsqH703mZ9lyE2r91rlcRCYrr6khI_3V8KuctJ-wljsSVNBELW4DhxwROA/05TrustedRoot.png?psid=1)

Now you have enabled your client to accept this XBAP application as full-trust application.
