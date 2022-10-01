---
layout: post
published: true
title: Deploy static site with Serverless Framework and GitHub Actions
author: Ahmadreza
comments: true
categories: [Blog]
tags: [serverless,github,github-actions,aws]
---

I have started working on a simple static website which I need to be able to continuesly deploy to AWS where it is hosted. This website will evolve into a progressive mobile application later on. I have also been tinkering with serverless building blocks on AWS as well as Serverless Framework. I decided to write this blog post about deplying a static web site with Serverless Framework and GitHub Actions.



## What is Serverless Framework?

[Serverless Framework](https://www.serverless.com/framework) helps developers to develop and deploy serverless application easily by defining the application as functions and events. It lets developer define the infrastructure as code and deploy the infrastructure with a single command. It has an extention model which let developers create plugins and extend functionality of Serverless Framework. Serverless Framework abstract away complexity of defining infrastrucutre using cloud providers specific templates, instead it offers a simple yaml definition format to define and connect your serverless components.

Serverless Framework and cli is free, however serverless has a subscription model which has different tiers, ranging from free to enterprise. Extra features like monitoring tools and support is in paid subscription. 

## What is GitHub Actions?

Github action is an automation tool in github which helps you automate workflows. You can compile, build, test and deploy your application using automation tools in GitHub Actions. You can compare it with the traditional continous integration and continious deployment tools (CI/CD) but in my opinion is more than that. GitHub Actions is flexible and there are many community actions you can use as well as github official [official actions](https://github.com/actions).

Actions are triggered by GitHub platform events directly in a repository and run on-demand workflows either on Linux, Windows or macOS virtual machines or inside a container in response. 

### GitHub Actions pricing

GitHub Actions are for free on public repositories. For private repositories you can use up to 2000 minutes per month.  For more information visit [github documentation about billing](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions) on github website.

## Overview and Approach

I use severless framework to deploy a simple html page to AWS S3. Initially deploying the html page to AWS s3 using command line and then define a workflow in GitHub Actions. Commits to `main` branch will trigger this action and GitHub Actions deploy the static site to AWS. 

I will create a sample step by step that you can follow along and re-create the same sample in your local maching and github account. 

![diagram](/img/serverless-github-actions.png)

Disclaimer: There are many other ways of deploying a static website perhaps potentially even easier that this approach. There are two main reasons I am using these tools for this task. First, the app I am working on will later have more components and moving parts which this set up help me to have more flexible deployment pipeline. Second, I want to demo capabilities of both Serverless Framework and GitHub Actions and potentially this post could be a starting point for bigger serverless tasks if someone come across this blog post.


## Requirements 

- Github account 
- AWS account if you want to deploy the statis site to AWS
- Serverless Framework. Follow [this instructions](https://www.serverless.com/framework/docs/getting-started) to install Serverless Framework on your local machine


## Github repository

Login with your github account to github and create [new](https://github.com/new) repository. 
I choose `static-site-serverless` as the name of this repository and add some description. 

![new repository](/img/github-static-site-serverless.png)



## Initialize Serverless Framework

After cloning your empty repository run the `serverless` command. (See requirements if you have not installed the serverless freamework).


``` bash
$ serverless
```

As I mentioned earlier Serverle


This command line will ask you questions to create a serverless project from template. 1) What do you want to make? answer: AWS - Node.Js - started. 2) What do you want to call this project? `static-site-serverless`. 3) Do you want to login/register to Serverless Dashboard? No. 4) Do you want to deploy your project? No

This command creates a simple serverless project which has a hello world lambda function. We need to synchronize a simple static site into an s3 bucket. So lets build a simple static site which could be a html file. So create a `src` folder and then create an `index.html` file in it.

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Hello World serverless</title>
</head>
<body>
    Hello world! serverless with GitHub Actions! 
</body>
</html>
```

As I mentioned earlier, Serverless framework has a plugin echosystem, and we can install them using `npm`. For the purpose of this sample we need a couple of packages like `serverless-s3-sync` and `serverless-s3-remover`. Use the following npm command to install and save these packages in `package.json`. 

``` bash
npm install --save-dev serverless-s3-sync serverless-s3-remover 
```

The main file which defines our serverless structure is `serverless.yml` file. This file is located in the root of the folder we have created. We need to define our serverless components in here. To find out more about this you can have a look at [Serverless Framework documentation](https://www.serverless.com/framework/docs/getting-started). 

We don't need the lambda function created by the template. Let's delete `handler.js` and modify the yaml file to following. 


``` yaml
service: serverless-github

frameworkVersion: '2'

provider:
  name: aws
  region: ap-southeast-2  
  stage: dev
  runtime: nodejs12.x

custom:
  siteName: ${opt:stage, 'dev'}.samples.ahmadreza.com
  s3Sync:
    - bucketName: ${self:custom.siteName}
      localDir: src 
      acl: public-read
      defaultContentType: text/html
      params: 
        - index.html:
            CacheControl: 'no-cache'

resources:
  Resources:
    StaticSite:
      Type: AWS::S3::Bucket
      Properties:
        AccessControl: PublicRead
        BucketName: ${self:custom.siteName}
        WebsiteConfiguration:
          IndexDocument: index.html
          ErrorDocument: error.html

plugins:
  - serverless-s3-sync
```

The main structure had section for defining `provider`, some `custom` configuration, `resources` and `plugins`. Our provider is `AWS` and we defined the region to be `ap-southeast-2` which is Asia Pacific (Sydney). The runtime is for lambda functions which you can ignore it now for this example. 

We need to use a plugin called `serverless-s3-sync` which syncs the local folder to the s3 and we need one AWS resource which is `s3` bucket we lest call this resource `StaticSite`. The properties for is S3 bucket defineing accss controll which in this case is `PublicRead` and also website configuration the default document is `index.html` and error document is `error.html`. The bucket name comes from a variable `self:custom.siteName`. Serverless has a [variable syntax](https://www.serverless.com/framework/docs/providers/aws/guide/variables/) which lets us to define varilables and use it in other sections or even other files. In the `custom` section, I defined the variables `siteName` which is concatination of the stage name with default value of `dev` and `.samples.ahmadreza.com`. For example if the stage name is prod then the variable value will be `prod.samples.ahmadreza.com`. In the `custom` section we also set plugins properties. We are using s3sync plugnin, therefore, we need to set properties like `bucketName` which is set to the same varialbe `siteName` and local directory which is `src`. By setting these properties the plugin will sync the local folder to the S3 bucket.


### Credentials for AWS deployment
In order to test this we can run serverless cli and deploy this. However, the serverless runs under the context of a user who needs certain roles. To read more details about managing [permission for the Serverless Framework user you can read here](https://www.serverless.com/blog/abcs-of-iam-permissions/). 

> IAM permissions for the Serverless Framework user is any permissions that are required when you run a command with the Serverless Framework, such as `sls deploy` or `sls logs`. 

> The Framework is making its calls to AWS using the Node aws-sdk. This means credentials are generally loaded from a file in `~/.aws/credentials` (for Mac/Linux users) or `C:\Users\USERNAME\.aws\credentials` for Windows users.




Lets commit and push this initial version by running following commands:

``` bash
$ cd static-site-serverless
$ git init --initial-branch=main
$ git remote add origin git@github.com:<YOUR-ACCOUNT>/static-site-serverless.git
$ git add .
$ git commit -m "initial commit"
$ git push --set-upstream origin main
```


## GitHub Actions 





## Resources to learn more about Serverless Framework and GitHub Actions
1. [Serverless Framework: Getting started](https://www.serverless.com/framework/docs/getting-started)
1. [GitHub Actions](https://docs.github.com/en/actions)
1. [Awesome GitHub Actions](https://github.com/sdras/awesome-actions)





#### Remove this 
other keywords to use 

GitHub Actions environment varialbes
GitHub Actions status
GitHub Actions marketplace