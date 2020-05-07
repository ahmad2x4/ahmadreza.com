---
layout: post
title: "Creating a secure deployment of AWS RDS and Elastic Beanstalk with pulumi"
date: 2020-05-06
author: ahmadreza
comments: true
categories: [Blog]
tags: [IaC, Pulumi, AWS, AWS RDS, AWS Elastic Beanstalk, TypeScript, Node.js, DevOps]
---

This blog post is part two of two blog posts about Infrastructure as Code with Pulumi:

1. Introduction to Infrastructure as Code with Pulumi
2. Creating a secure deployment of AWS RDS and Elastic Beanstalk with Pulumi

## Introduction
In the previous blog post, we have described Infrastructure as Code and differences between different types of IaC. Also, described what Pulumi is and created a simple Pulumi project which creates an S3 bucket in AWS.

In this blog post, we are going to create a secure deployment of AWS RDS and Elastic Beanstalk web application with Pulumi.

We used javascript and Node.js runtime for the web application. This web app simply tries to connect to the database server hosted in private subnet when an endpoint is accessed.

Following is the AWS component diagram for our application:

![AWS Beanstalk connected to RDS in private subnet](images/AWS-Secure-RDS.png)

We have one VPC, with two subnets: public, private subnet. We want to create our web application in our public subnet using Elastic Beanstalk and create our database in the private subnet, so it is not accessible from outside of the VPC. This network design is a recommended practice to keep your database secure. Our application is a Node.js application running in a docker container in Elastic Beanstalk. 


Let's have a look at the folder structure first:

![AWS Beanstalk Stack Folder structure](images/vpc-working-folder-strcuture.png)

`infra` folder is for infrastructure for Pulumi and source code for the application written Node.js is in `src`. 

## Application code

Let's start with the application source code. It is a simple application that tests the connection to the database. The following code snippet is in the `app.js` file. 

```typescript
//app.js
const port = process.env.PORT || 3000,
  sql = require("mssql"),
  http = require("http"),
  fs = require("fs"),
  html_disconnected = fs.readFileSync("index_disconnected.html"),
  html_connected = fs.readFileSync("index_connected.html");

const try_connect_sql = async (connectionString) => {
  try {
    console.log("trying to connect to server", connectionString);
    const connection = await sql.connect(connectionString);
    console.log("Connection to sql server was successful", connection);
    connection.close();
    return true;
  } catch (err) {
    console.log(err);
    return false;
  }
};
```

First, we are assigning a few variables like Port and importing a few packages like MSSQL, HTTP, fs. Then we are defining a function called `try_connect_sql`. This function attempts to connect to the database with the connection string passed to it. If it is successful, it returns `true`; otherwise, it returns `false`.  


```typescript
// app.js (continues) 
var server = http.createServer(async (req, res) => {
  if (req.method === "GET") {
    res.writeHead(200, "OK", { "Content-Type": "text/html" });
    const result = await try_connect_sql(process.env.CONNECTION_STRING);
    res.write(result ? html_connected : html_disconnected);
  } else {
    res.writeHead(405, "Method Not Allowed", { "Content-Type": "text/plain" });
  }
  res.end();
});

// Listen on port 3000, IP defaults to 127.0.0.1
server.listen(port);

// Put a friendly message on the terminal
console.log("Server running at http://127.0.0.1:" + port + "/");
```

The next block of code creates a simple HTTP server that responds to `GET` requests only. For any `GET` request, it attempts to connect to the database, and if the connection is successful, it returns a page with a green background saying _Congratulations application is connected to RDS!_. If it could not connect to the database, it returns a page with a red background saying _Unfortunately application is not connected to RDS!_.  

In the end, it starts the server by listening to the Port and logs that the server has started.


You can run this simple app locally. If you want to test this, you can run `npm start` in the `src` folder. When you browse http://127.0.0.1:3000, if there is no database connection, it returns a red page, but if you have a database running, it returns a green page. You can run a SQL database in docker to test the successful path by following this  [instruction](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash).


In the `package.json` file, there are two scripts. The first one is the `start` script, and the second one is `package`.  This script, package up the app folder in zip format, later, our infrastructure code deploys that zip file to Elastic BeansTalk.


```json
  "scripts": {
    "start": "node app.js",
    "package": "cross-zip ../src deployment.zip "
  }
```

## Infrastructure code

The infrastructure code is in the `infra` folder. It is written in `typescript` using Pulumi library.

### Adding configuration 
Pulumi encrypts secrets in the configuration file using a `PASS_PHRASE` you choose, Pulumi also adds a random salt to the encryption. You can add a secure configuration using `pulumi config set ... --secrets`. Sensitive configurations like database password should be added as secret. I didn't commit dbPassword configuration to the YAML file. So you can set database password using the following Pulumi command.  


```bash
$ pulumi config set vpc_rds_dmz:dbPassword "[STRONG_DB_PASSWORD]" --secret
```

This command asks you to choose a passphrase for your secrets and confirm it. 


Now, let's have a look at the infrastructure code. The main file is in `./infra/index.ts`. The programming language for Pulumi code is typescript. Let's go through different blocks of code to describe each block and what they mean.


```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

const config = new pulumi.Config();

const dbPassword = config.require("dbPassword");

// Allocate a new VPC with the default settings:
const vpc = new awsx.ec2.Vpc("custom");
```

At the top it just imports different libraries `@pulumi/pulumi`, `@pulumi/aws` and `@pulumi/awsx` then, initialize the config object to get some values from the configuration file, in this case, the configuration value is `dbPassword` which we set up earlier. Then we create a VPC called `custom` using `awsx.ec2.Vpc` object. This class encapsulates a complete configuration of an AWS network, including the actual VPC itself, in addition to public and private subnets, route tables, and gateways, across multiple availability zones. But in this example, we use private, and public subnet addresses to use for our DB and Elastic Beanstalk, respectively.

```typescript
const rdsSecurityGroup = new aws.ec2.SecurityGroup(`dbsecgrp`, {
  vpcId: vpc.id,
  ingress: [
    {
      protocol: "tcp",
      fromPort: 1433,
      toPort: 1433,
      cidrBlocks: [vpc.vpc.cidrBlock],
    },
  ],
});

const dbSubnets = new aws.rds.SubnetGroup("dbsubnets", {
  subnetIds: vpc.privateSubnetIds,
});
```

This block of code creates a security group using `aws.ec2.SecurityGroup` class to enable TCP ingress for SQL Port. Later, we use this security group for our database. The next step is to create a subnet group with private subnet ids using `aws.rds.SubnetGroup`. 


```typescript
const rds = new aws.rds.Instance(`database-dev`, {
  engine: "sqlserver-ex",
  username: "admin",
  password: dbPassword,
  instanceClass: "db.t2.micro",
  allocatedStorage: 20,
  skipFinalSnapshot: true,
  publiclyAccessible: false,
  // For a VPC cluster, you will also need the following:
  dbSubnetGroupName: dbSubnets.id,
  vpcSecurityGroupIds: [rdsSecurityGroup.id],
});
```

In this step, we create the RDS itself using `aws.rds.Instance` class. There are many parameters for creating rds. However, there are 3 essential parameters. First, `dbPassword` which we pulled from our configuration. Second is `dbSubnetGroupName` which is set by the subnetGroup created for the private subnet. It means our RDS in the private subnet. And third, `vpcSecurityGroupIds` security group which we set to the security group created earlier with SQL Port ingress rule.


```typescript
const ebAppDeployBucket = new aws.s3.Bucket("eb-app-deploy", {});

const ebAppDeployObject = new aws.s3.BucketObject("default", {
  bucket: ebAppDeployBucket.id,
  key: "deployment.zip",
  source: new pulumi.asset.FileAsset("../deployment.zip"),
});
```

The next step is to upload our `webapp` artifacts to s3 and make them ready for deployment to AWS Elastic Beanstalk. To make the deployment package ready, we just need to run following command line command to create the zip package in `./src` folder.

```bash 
$ npm run package
```

This command packages up the whole `src` folder into a zip file called `deployment.zip`. Assuming we had already run this command and zip file is ready, the infrastructure code uploads this zip file to an S3 bucket called `eb-app-deploy` which is also created by our infrastructure code.


The next step is to create Elastic Beanstalk. Creating Elastic beanstalk is a bit more complected. There are a few things that should be ready before creating the application and environment.


```typescript
const instanceProfileRole = new aws.iam.Role("eb-ec2-role", {
  name: "eb-ec2-role",
  description: "Role for EC2 managed by EB",
  assumeRolePolicy: pulumi.interpolate`{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": "sts:AssumeRole",
                "Principal": {
                    "Service": "ec2.amazonaws.com"
                },
                "Effect": "Allow",
                "Sid": ""
            }
        ]
    }`,
});
```

*Instance Profile role* - Instance profile role used in instance profile. There is a class in Pulumi AWS SDK to create this profile role `aws.iam.Role`

```typescript
const instanceProfile = new aws.iam.InstanceProfile("eb-ec2-instance-profile", {
  role: instanceProfileRole.name,
});
```

*Instance profile* - An instance profile is an IAM role that is applied to instances launched in your Elastic Beanstalk environment. When creating an Elastic Beanstalk environment, you specify the instance profile that is used when your instances. In Pulumi SDK we can use `aws.iam.InstanceProfile` to create the instance profile.

```typescript
// CREATE BEANSTALK APPLICATION
const app = new aws.elasticbeanstalk.Application(`webapp`, {
  name: `webapp`,
});
```

Then we need to create the Elastic Beanstalk app itself. it is achievable by using `aws.elasticbeanstalk.Application` class. It needs at least `name` value, which we have provided and it is called `webapp`.

```typescript
const defaultApplicationVersion = new aws.elasticbeanstalk.ApplicationVersion(
  "default",
  {
    application: app,
    bucket: ebAppDeployBucket.id,
    description: "Version 0.1",
    key: ebAppDeployObject.id,
  }
);
```

*Application version* - Elastic Beanstalk environment needs an application version to deploy the environment. We can create an application version with the `Zip` file we uploaded to S3 and configure environment by setting the version parameter.


The next step is to create the connection string. We need to create the connection string based on values from our RDS and also database password (secure parameter we added to Pulumi configuration).
Before describing how we construct the connection string we should mention how inputs and outputs are working in Pulumi. Pulumi creates resources asynchronously, which means outputs of a given resource might not be available immediately in the next step in the sequence. 

Pulumi uses a special type called output. According to Pulumi documentation:

>Outputs are values of type `Output<T>` , which behave very much like promises; this is necessary because outputs are not fully known until the infrastructure resource has actually completed provisioning, which happens asynchronously. Outputs are also how Pulumi tracks dependencies between resources.

To construct connection string we need to wait for rds resource to be created then we can get the server address and Port. Pulumi has a utility function called [pulumi.all()](https://www.pulumi.com/docs/intro/concepts/programming-model/#all) _This function joins over an entire list of outputs, waiting for all of them to become available, and then provides them to the supplied callback. 

```typescript
// SQL CONNECTION STRING
export const connectionString = pulumi
  .all([rds.address, rds.port])
  .apply(
    ([serverName, port]) =>
      `Server=${serverName},${port};uid=admin;pwd=${dbPassword};`
  );
```

This block of code is waiting for both address and Port to become available, and when they are available runs `apply` function and return the connection string. 



```typescript
const tfenvtest = new aws.elasticbeanstalk.Environment("webapp-env", {
  application: app,
  version: defaultApplicationVersion,
  solutionStackName: "64bit Amazon Linux 2018.03 v4.13.1 running Node.js",
  settings: [
    {
      name: "VPCId",
      namespace: "aws:ec2:vpc",
      value: vpc.id,
    },
    {
      name: "Subnets",
      namespace: "aws:ec2:vpc",
      value: vpc.publicSubnetIds[0],
    },
    {
      name: "IamInstanceProfile",
      namespace: "aws:autoscaling:launchconfiguration",
      value: instanceProfile.name,
    },
    {
      name: `CONNECTION_STRING`,
      namespace: "aws:elasticbeanstalk:application:environment",
      value: connectionString,
    },
    {
      name: "SecurityGroups",
      namespace: "aws:autoscaling:launchconfiguration",
      value: rdsSecurityGroup.id,
    },
  ],
});

export const endpointUrl = tfenvtest.endpointUrl;
```

The final step is to create an environment for the Elastic Beanstalk application. This is achievable using `aws.elasticbeanstalk.Environment` class. We set the `app` parameter and `version` to previously created applications and versions. We also set the platform to "64bit Amazon Linux 2018.03 v4.13.1 running Node.js". The last property is setting which is an array of key/values. We set the `VPCId` to vpc and subnet to public subnet. `IamInstanceProfile` value is set to the instance profile name we created earlier. `SecurityGroups` is set to the security group id we created. `CONNECTION_STRING` is an environment variable and is set to the value of the connection string variable. 

Then the last line returns the endpoint URL as an output so Pulumi prints int out in console after running the Pulumi command line.

Now its time to run the Pulumi and create the stack. Make sure you have created the package.zip and then in `./infra` folder run following command:

```bash
$ pulumi up
```

After running this command, Pulumi asks for a passphrase. Provide passphrase you have selected earlier and then select `yes`. It'll take a few minutes to create the whole stack. After pulumi created the entire stack successfully, it'll output connection string and Elastic Beanstalk Endpoint. If you copy-paste this URL into your browser, it should return a green page with this message in it:

```text
Congratulations application is connected to RDS!
```

Congratulation we have creates an Elastic Beanstalk application backed by secure RDS. Now if you want to destroy this environment, you can simply run following command which removes all resources been created by Pulumi.

```bash
$ pulumi destroy
```

If you want to clone full example, you can access the repository here: https://github.com/ahmad2x4/vpc_rds_dmz_pulumi