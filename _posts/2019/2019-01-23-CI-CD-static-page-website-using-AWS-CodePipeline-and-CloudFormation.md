---
layout: post
title: "CI/CD for static website using CodePipeline and CloudFormation"
date: 2019-01-23 16:30
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [AWS, CloudFormation, S3, CodePipeline, CodeBuild, CodeDeploy]
---

Useful links https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/continuous-delivery-cod epipeline-basic-walkthrough.html
MyRepository https://github.com/ahmad2x4/aws-taskcat-sample should be renamed to something like aws-static-page

```
{
  "name": "aws-taskcat-sample",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "echo \"Warning: nothing happens here\""
    
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/ahmad2x4/aws-taskcat-sample.git"
  },
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/ahmad2x4/aws-taskcat-sample/issues"
  },
  "homepage": "https://github.com/ahmad2x4/aws-taskcat-sample#readme"
}
```

## Steps 

# Create stack placeholder 

- Go to cloudformation and create a new stack this is the placeholder
- Design template 
- Drop a simple S3 block 
- Choose YAML as your prefered template language
- Save the template on Amazon S3 bucket
- Copy the URL in clipboard that is prompted when it is saved
- Go back to CloudFormation in AWS Console 

- Create a stack 
- Use `Specify an Amazon S3 template URL and the paste the ULR you have copied in previous step
- Click next and in the next step choose a name for the stack e.g. `simple-static-page` 

 -For this walkthrough, you don't need to add tags or specify advanced settings, so choose Next.

- Ensure that the stack name and template URL are correct, and then choose Create.

- Click next and then create

Now you have a CloudFormation stack. The idea is to be able to update this stack as we develop our application and keep the template in our source code 

If you download the CloudFormation generated buy the desiner stored in your S3 bucket, you can open it with a text editor of your choise and see the content 

After a bit of clean up the template looks like this :

```
Resources:
  S3B3AIZH:
    Type: 'AWS::S3::Bucket'
    Properties: {}

```

# Create a repository in github for your source code
files:

- index.html (Simple html)
- your cloudformation template (static-page.template.yml)
- and a package.json file which does not do anything at the moment but we are going to add more later




# Create a CodePipeline in AWS 
- Go to CodePipeline and choose `Create Pipeline`
- Give the new pipeline a name like `simple-static-page`
- Keep the option ofr Service Role as `New Service Role`
- Go to the next step and for your Srouce provided choose GitHub 
- Then you need to grant AWS access to your GitHub Repository. To do this you need to go through OAuth flow and agree on proposed concents 
- Once you have connected your github account to AWS you can select your repository and the branch name.
- Keep GitHub Webhook selectedn and go to the next step

In the next step we are going to define a simple build step to genereate build artifact. At this step our website is very simple and doing almost nothing  however once you introduce javascript frameworks it can get more complext. Lets assume our build is based on npm package.json and we are going to use AWS CodeBuild as our build system 


# CodeBuild
one of the most important configuation is 

```
version: 0.2

phases:
  install:
    commands:
      - echo Installing npm packages...
      - npm install
  pre_build:
    commands:
      - echo Pre-build phase
  build:
    commands:
      - echo Build started on `date`
      - echo Compiling the Node.js code
      - npm run build
  post_build:
    commands:
      - echo Build completed on `date`
artifacts:
  files:
    - '**/*'
```



# CodeDeploy (CloudFormation)

# CodeDeploy (Content)