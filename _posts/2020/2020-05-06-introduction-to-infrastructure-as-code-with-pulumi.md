---
layout: post
title: "Introduction to Infrastructure as Code with pulumi"
date: 2020-05-06
author: ahmadreza
comments: true
categories: [Blog]
tags: [IaC, Pulumi, AWS, S3, DevOps]
---

This blog post is part one of two blog posts about Infrastructure as Code with Pulumi:

1. Introduction to Infrastructure as Code with pulumi
2. Creating a secure deployment of AWS RDS and Elastic Beanstalk with Pulumi

## Introduction 

This post covers the following topics:

- Introduction to Infrastructure as Code
- Using Pulumi to automate the deployment
- Hello world of Infrastructure as Code (IaC) with Pulumi

## Introduction to Infrastructure as Code
IAC or Infrastructure As Code is getting a lot of attention recently amongst DevOps teams. 

Definition of Infrastructure as Code according to [Wikipedia](https://en.wikipedia.org/wiki/Infrastructure_as_code):

>_Infrastructure as code (IaC) is the process of managing and provisioning computer data centres through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools. The IT infrastructure managed by this comprises both physical types of equipment such as bare-metal servers as well as virtual machines and associated configuration resources. The definitions may be in a version control system. It can use either scripts or declarative definitions, rather than manual processes, but the term is more often used to promote declarative approaches._

### Benefits of Infrastructure as Code
- Stored in repository along with your application
- Traceable, changes can be tracked in the same manner as code via commit history and logs
- Empowers DevOps teams
- In-sync with your application


### Different types of IaC
#### Declarative:
In declarative IaC techniques, you define *what* you want as infrastructure, you declare end-result state in a DSL (Domain Specific Language) or even in a popular programming language like Javascript. In other words, you declare the desired state of your environment and the framework/tool you are using creates the environment to fulfil the desired state.

#### Imperative:
With the imperative IaC approach, you define *how* you want to create your infrastructure. You breakdown the end-result into instructions using scripting languages or similar approaches which then create your infrastructure.

#### Intelligent:
The intelligent approach determines the desired state of the system and determines what needs to change before applying changes.
>_Environment aware desired state is the next generation of IaC._



## Pulumi
According to Pulumi web site:

>_Pulumi is a modern infrastructure as code platform. It includes a CLI, runtime, libraries, and a hosted service that, working together, deliver a robust way of provisioning, updating, and managing cloud infrastructure. Instead of YAML or a domain-specific language (DSL), Pulumi leverages existing, familiar programming languages, including TypeScript, JavaScript, Python, Go, and .NET, and their native tools, libraries, and package managers._

In terms of categories of infrastructure as code, Pulumi falls into *Declarative* category. You can declare the state of your cloud stack in your preferred language, Pulumi analyzes the current state of the environment, determines differences and prepare a set of instructions which transform the current state of your environment to the desired state. 

Pulumi currently supports four runtime and multiple languages on those runtimes. For more information, please see [Languages](https://www.pulumi.com/docs/intro/languages/)

- **Node.js** - JavaScript, TypeScript, or any other Node.js compatible language
- **Python** - Python 3.6 or greater
- **.NET Core** - C#, F#, and Visual Basic on .NET Core 3.1 or greater
- **Go** - statically compiled Go binaries

### Key features and differences of Pulumi:

- Since Pulimi is using an existing language, it can use techniques in that language or ecosystem of the language for abstraction, code reuse, and defining modules.
- No need to learn a new custom language defined by an IaC framework. For example, if you want to use [Terraform](https://www.terraform.io/), you need to learn new domain-specific-language (DSL) defined by HashiCorp.
- Pulumi figures out dependencies and manages concurrency. For example, when the ID of a resource is used in another resource, Pulumi automatically waits for the creation of the first resource to acquire values required by the other resource and then attempts to create the other resource.
- Most cloud providers have their definition of IaC. This can be in the form of JSON or YAML files which can often result in big bloated files that are hard to read and difficult to reason about your code. However, Pulumi's code is in your preferred language, which is much more familiar and therefore readable than the native option.

Pulumi is actually a command-line tool (CLI) and some libraries in different languages supported by Pulumi. Pulumi also has a paid subscription-based platform which provides additional services. For example, features like continuous integration and continuous deployment, CrossGuard Policy as code, Security and additional support.

However, you can still use CLI tool which is free  and successfully set up your infrastructure as code using command line tools. You just need to install [Pulumi CLI tool](https://www.pulumi.com/docs/get-started/install/) on your development environment and CI/CD agents. 


### Hello world of IaC with Pulumi
I want to go through setting up Pulumi and creating your first stack by what I call "Hello world of IaC with Pulumi". 
"Hello world" of IaC is going to be something as simple as creating blob storage in your preferred cloud provider platform with your preferred runtime and language. In this post, I am going to create my stack (Blob storage) on AWS and also use Node.js runtime with typescript.

#### Setup Pulumi

If you are using MACOS, you can use Homebrew to install Pulumi. To install Pulumi on other platforms, you can check [Install Pulumi](https://www.pulumi.com/docs/get-started/aws/install-pulumi/) 

```bash
$ brew install pulumi
```

#### Login to Pulumi service:
To manage your stack, you need to initialise your Pulumi CLI that sets up storage service which is used to manage your stack's state. Pulumi uses this Service/Storage to keep your current state and history of changes to your stack. So each successful update to your stack by Pulumi also updates this storage to keep track of your stack's changes.
There are many options you can use to initialise the Pulumi stack.

- Local storage
- Pulumi SaaS (subscription-based, Software as a Service)
- Cloud blob storage (AWS s3, Azure Blob Storage)

In this example, we use AWS S3 bucket as state storage for Pulumi. To login to the storage, you need to [install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) [generate an Access Key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) and [configure your AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-quick-configuration) to use that Access Key to communicate with AWS account 

Using a terminal, create the pulumi state bucket 


``` bash
$ aws s3api create-bucket --bucket pulumi-infra-bucket --region ap-southeast-2 --create-bucket-configuration LocationConstraint=ap-southeast-2
```

```
$ pulumi login s3://pulumi-infra-bucket
```

Make sure Node.js is installed on your system

``` bash
$ mkdir hello-world-pulumi && cd hello-world-pulumi
$ pulumi new aws-typescript
```


``` 
project name: (hello-world-pulumi)
project description: (A minimal AWS TypeScript Pulumi program)
Created project 'hello-world-pulumi'

stack name: (dev)
Enter your passphrase to protect config/secrets:
Re-enter your passphrase to confirm:
Created stack 'dev'

Enter your passphrase to unlock config/secrets
    (set PULUMI_CONFIG_PASSPHRASE to remember):
aws:region: The AWS region to deploy into: (us-east-1) ap-southeast-2
Saved config

Installing dependencies...
.
.
.
```

This step could take some time as Pulumi is installing all npm packages and plugins.

In the meantime, we can have a look at what pulumi has created as source code. The folder structure looks like the following image:

![Folder and file structure](images/Pulumi-file-structure.png)

Inside this folder, there is `index.ts` file which is the primary source code for Pulumi. There is also `package.json` which is npm package file. There are two YAML file called `Pulumi.yaml` and `Pulumi.dev.yaml` the first file is the general configuration file for Pulumi. Configurations like `name`, `runtime` and `description`. The other YAML file is per environment. Depending on the number of environments you have you'll have multiple `Pulumi.*.yaml` files. Currently we only have one environment in the project which is dev therefore we have `Pulumi.dev.yaml`. These YAML files are defining the configuration per environment. 

```  yaml
encryptionsalt: <ENCRYPTIONSALT>
config:
  aws:region: ap-southeast-2
```

In this example, it has `aws:region` defined for the dev environment as `ap-southeast-2`. 

These are the main files for the quick hello world example out-of-the-box. But let's have a look in the main file `index.ts` and see what this sample is building.


``` typescript
import * as aws from "@pulumi/aws";

// Create an AWS resource (S3 Bucket)
const bucket = new aws.s3.Bucket("my-bucket-123");

// Export the name of the bucket
export const bucketName = bucket.id;

```

There are only three lines of code here. The first line is importing pulumi/aws package and alias as `aws`. The next line is using this API to create a new S3 bucket called `my-bucket-123`. The last line exports the name of the bucket. 



Now the initial setup of this sample is done, and we know what it is going to build, let's run this in our connected AWS account and see the result. To run this, you can use the CLI tool and run the following command.

``` bash
$ pulumi up
```

Next, Pulumi asks you for the passphrase. Enter the same passphrase you have chosen during the initial setup. Then it analyses the current state of the environment (which in this case is not created yet) and then reports back all the resources missing. It is essential to know that Pulumi does not create the resource straight away; it asks for confirmation if you want to create those resources. This allows you to inspect the details of commands/resource it should create.

```
Resources:
    + 2 to create

Do you want to perform this update?  [Use arrows to move, enter to select, type to filter]
  yes
> no
  details
```

If you choose *yes* then Pulumi runs commands and create the resources.

```

Do you want to perform this update? yes
Updating (dev):
     Type                 Name                    Status      
 +   pulumi:pulumi:Stack  hello-world-pulumi-dev  created     
 +   └─ aws:s3:Bucket     my-bucket-123           created     
 
Outputs:
    bucketName: "my-bucket-123-0fe575c"

Resources:
    + 2 created

Duration: 10s
```

Having executed command successfully, Pulumi produces some output. In this case it returns `my-bucket-123-0fe575c`. If you notice there is an extra 7 character added to the original name that we gave to the bucket. The reason is that Pulumi can create multiple sets of stacks on the same account/region. If it does not add the postfix to the name, it would be impossible to create the two resources with the same name. 

Alright, so that was quite a simple example with Pulumi. This example is what I call the equivalent hello world in Pulumi. In the next blog post we are going to develop more real-world scenario. We are going to create a secure deployment of AWS RDS and Elastic Beanstalk with pulumi.