---
layout: post
title: How to construct ECS service with EFS volume using CDK
published: true
author: ahmadreza
comments: false
categories: [blog]
tags: [AWS, CDK, ECS, ECR, EFS]
excerpt_separator: <!--more-->
---

In this post, I will demonstrate how to use Amazon Elastic File System (EFS) to persist user input data in a Node.js application. Specifically, I will show how to use an EFS volume as a local storage for a container and how data is retained even after the container is restarted. <!--more--> Additionally, I will utilise the Cloud Development Kit (CDK) to create an ECS Fargate service and connect it to an EFS volume for persistent storage.

## Introduction

ECS is a fully managed container orchestration service that makes running, scaling, and securing containerised applications easy. A key feature of ECS is the ability to define and run tasks, which are units of work that are made up of one or more container instances. Tasks can be used to run a single container or multiple containers that work together as a group. With ECS tasks, you can quickly deploy and manage your containerised applications and take advantage of built-in features such as automatic scaling, load balancing, and service discovery. By default, the Amazon ECS task gets 20 GiB of ephemeral storage. The size of ephemeral storage is configurable and can be increased up to a maximum of 200 GiB. Ephemeral storage is temporary storage that is not persistent and is not retained after the instance or container is terminated or rebooted. You can use this storage as temporary storage when you don't care if the container restarts and you lose the data. This type of storage is suitable for local caching. Services are often limited by memory size and can move infrequently used data into storage slower than memory with little impact on overall performance. Another use case is to store read-only input data to be present in files, like configuration data or secret keys.

Applications often connect to a database outside the container with proper persistence storage like RDS or DynamoDB. Still, if your application requires a volume to persist data permanently and reliably, you need a storage service like EFS volumes. EFS is fully managed and scalable. EFS allows you to create a file system and mount it to multiple instances simultaneously, making it easy to share files and data between instances. With EFS, you can quickly scale your storage needs up or down based on usage, and you only pay for the storage you actually use, with no upfront costs or commitments.


## What is CDK?

CDK is an open-source software development framework that enables developers to create and provision cloud infrastructure using familiar programming languages such as TypeScript, JavaScript, Python, C#, and Java. With CDK, you can define your cloud infrastructure as code and use the same programming constructs and tools you already know and love, making it easy to build, deploy, and update cloud applications.

To read more about CDK, use this link ([Getting started with CDK](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html)).


## The sample web app:

We need an actual application to demonstrate the persistence requirements for a container. I'll be using a simple Node.js application in ECS. It would help if you had basic knowledge about Node.js, npm and `Express.js` to follow the rest of the blog post.

In this sample code, I created a folder called docker_web_app and initiated npm. I also installed `Express.js` node module. Then created the follwoing `server.js` file. I set the `main` entry to `server.js` In the `package.json`.

This simple Node.js application uses the Express web framework to create a server that listens for incoming HTTP requests. When a request is received, the application reads and updates a counter stored in a file called `db.txt`. The application uses the File System (fs) module to read and write the file. The counter is incremented each time a request is received, and the current count is returned to the user in the response. 
This server also exposed an [GET] endpoint for a health check which we will use later in AWS.


server.js

``` javascript
'use strict';
const fs = require('fs');
const path = require('path');
const express = require('express');

// Constants
const PORT = process.env.PORT || 8080;
const HOST = process.env.HOST || '0.0.0.0';
const filePath = process.env.FILE_PATH || '.';

// App
const app = express();
app.get('/', (req, res) => {
    const dbPath = path.join(filePath, 'db.txt')
    if (!fs.existsSync(dbPath)) 
      fs.writeFileSync(dbPath, "0");
    let count = Number(fs.readFileSync(dbPath));
    fs.writeFileSync(dbPath, (++count).toString());
    res.send(`Refresh count ${count}.`);
  });

app.get('/health', (req, res) => { res.send('Ok'); });

app.listen(PORT, HOST, () => {
  console.log(`Running on http://${HOST}:${PORT}`);
});
```

You can run this node web application locally by running the following command. This command sets an environment variable that is used by the `Node.js'` application. `FILE_PATH` is the path of the file storage. The second part of the command is simply running the application.

``` bash
export FILE_PATH=./ && npm start
```


Once the app starts, you can navigate to the [http://0.0.0.0:8080/](http://0.0.0.0:8080/)URL. Every time you refresh this page, it increases the count of page visits by one. If you stop this Node.js app and start again, it will continue from the last number since it stores the last number on the local path. However, if we dockerize this node application and restart the container, the counter starts from 1 again unless we map a volume to the container. Now let's dockerize this web application by following [this guide](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/).

Add `db.txt` line to `.dockerignore` to ignore in docker container.

``` text
node_modules
npm-debug.log
db.txt
```

Build the docker image using the following command (use your own docker hub username):

``` bash 
docker build . -t <your username>/node-web-app
```

Run the docker using the following command. Pass the `FILE_PATH` environment variable as `./` which is the app folder:

``` bash
docker run -p 8080:8080 -e FILE_PATH=./ -d <your username>/node-web-app 
```

Once running, navigate to the [http://localhost:8080](http://localhost:8080) URL. The visit counter starts from one, and it increases if you refresh the page a few times. Now, let's stop the docker container and rerun it.

``` bash 
docker stop <IMAGE_ID>
docker run -p 8080:8080 -e FILE_PATH=./ -d <your username>/node-web-app 
```



Now if you navigate to the [http://localhost:8080](http://localhost:8080) URL you'll notice it starts from one again. To retain the counter value we need to either run named container or mount a volume. Lets mount a local volume and run the application again.

``` bash 
docker stop <IMAGE_ID>
docker run -p 8080:8080 -v data:/data -d <your username>/node-web-app   
```

Now if we navigate to the app URL and refresh a few times, even if we stop and start, the counter will resume from the last value. Ok, this is good for the local testing; however, we need to run this in AWS ECS service. So let's get started with CDK.

## Create an Elastic Container Registry

First we need to initialise CDK. Lets create a folder called `iac` (Infrastructure as code). In this blog post we will use TypeScript as scripting language for CDK. Install `aws-cdk-lib` globally:

```
npm install aws-cdk-lib -g
npm install -g typescript
cd iac
cdk init app --language typescript
```

Installing the packages required for the CDK templated project will take a couple of minutes. So far, we have installed aws-cdk-lib and TypeScript globally and then inside `iac` folder, we have initialised a CDK template project with TypeScript. The template has a specific folder structure.

- `lib/iac.ts` is where your CDK application’s main stack is defined.
- `bin/iac.ts` is the entrypoint of the CDK application. It will load the stack defined in `lib/iac.ts`.

To simplify the CDK we will define all code in `bin/iac.ts`. The following code defines an ECR repository called web-app. 


./bin/iac.ts

```  typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import * as ecr from 'aws-cdk-lib/aws-ecr';
import { Construct } from 'constructs';
import { CfnOutput } from 'aws-cdk-lib';

const app = new cdk.App();

class EcrStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const repository = new ecr.Repository(this, "web-app", {
      repositoryName: "web-app"
    });

    new CfnOutput(this, "repository-uri", { value: repository.repositoryUri });
  }
}

new EcrStack(app, 'EcrStack', {});
```

The above code creates an ECR Stack using `aws-cdk-lib` and its sub-libraries like `aws-cdk-lib/aws-ecr`, AWS CDK library for Amazon ECR.

First, an instance of the App class from the `aws-cdk-lib` is created in the code, which defines and runs the CDK application.

The EcrStack class extends the Stack class from the `aws-cdk-lib` and is used to create an Amazon ECR repository. The repository is created by instantiating the Repository class from `aws-cdk-lib/aws-ecr`, with the name of "web-app".


And the final step is to create an output using the `CfnOutput` class. It will print out the repository URI of the ECR repository after deploying.

Finally, an instance of the EcrStack class is created, passing in the app instance and a string "EcrStack" as the stack's ID. This creates and runs the CDK stack.

### Deploy Ecr Stack

Ensure your local machine has access to your AWS account with sufficient privileges. To deploy EcrStack you need to run the following command.

```
cdk deploy EcrStack
```



After successful CDK deployment, if you log in to AWS console and navigate to ECR you'll notice a new registry called `web-app`.The output of the CDK is the repository URI. We need this URI later to connect to and push our container image.

```
EcrStack.repositoryuri = xxxxxxxxxxx.dkr.ecr.ap-southeast-2.amazonaws.com/web-app
```

To push the docker image, we need to get an authentication token and authenticate the docker client to this registry:


```
aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin xxxxxxxxxxx.dkr.ecr.REGION.amazonaws.com
```

If the above command prompted `Login Succeeded`, your local docker client is now authenticated to interact with your remote Ecr.

Tag the image you previously built with the Amazon ECR registry, repository, and optional image tag name combination to use. The registry is xxxxxxxxxxx.dkr.ecr.ap-southeast-2.amazonaws.com. If you omit the image tag, AWS assume that the tag is the latest. Then you can push the image to the registry.

```
docker tag <your username>/node-web-app  xxxxxxxxxxx.dkr.ecr.ap-southeast-2.amazonaws.com/web-app:latest`
docker push xxxxxxxxxxx.dkr.ecr.ap-southeast-2.amazonaws.com/web-app:latest
```


## Create an Fargate ECS Stack

Now with the container image pushed to the container registry, we can create our Fargate ECS stack. Let's extend our `iac.ts` code with another stack called `EfsStack`. We added a new class called EfsStack which extends the `cdk.Stack` class. In this class, we create a new VPC named `web-vpc`. Then we create a security group within this VPC which is important to allow our container access to the EFS via port `2049` (NFS Service port) within the same security group.

The next step is to create the file system using `efs` construct and assign the security group we created in the previous step. Then we create an access point with the required permission for the file system. Then create efs volume configuration with the access point and file system, which is used for the volume configuration in our ECS task definition.

Then we create a new ECS cluster and a task definition for our web application. In the task definition, we set the `FILE_PATH` to `/data` so that our node.js app will write the counter to that path. Then we need to define our container, which uses the image we pushed to the ECR. As part of this step, we also enable logging, which is handy for troubleshooting. 

Then mount the volume we created previously so that the container would have access to the volume to persist the counter value in `/data`. 

<!--reviewed up to here -->

If you look at the container image we built earlier, it is listening to port `8080` so we need to map this port on TCP protocol. Then we create a service and assign it to the cluster we created earlier and the above task definition. We need to expose this service so that we can call this service over the internet. 

In the next step, we create an application load balancer in the VPC and add a port listener to port `80`. The last step is to add a target to port `80`, our ECS service. In this step, we also configure the health check endpoint, which we defined earlier in our Node.js application. The load balancer uses this endpoint to check the application's health status. If the application is not healthy, it will restart the container. We wanted to avoid pointing the health check to the homepage as it would increase the number of visits per health check every 30 seconds.

This CDK code will output the DNS for the applciation load balance which you can use to access the application over internet.

./bin/iac.ts

``` typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import * as ecr from 'aws-cdk-lib/aws-ecr';
import { Construct } from 'constructs';
import { CfnOutput } from 'aws-cdk-lib';

import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import * as ecr from 'aws-cdk-lib/aws-ecr';
import * as ecs from 'aws-cdk-lib/aws-ecs'; // ecs constructs
import * as ec2 from 'aws-cdk-lib/aws-ec2'; // ec2 constructs
import * as efs from "aws-cdk-lib/aws-efs"; // efs constructs
import elbv2 = require('aws-cdk-lib/aws-elasticloadbalancingv2'); // load balancing constructs 

import { Construct } from 'constructs';
import { CfnOutput, Duration } from 'aws-cdk-lib';

const app = new cdk.App();

...  //ECR CDK code

class EfsStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const vpc = new ec2.Vpc(this, "web-vpc");

    const webSecurityGroup = new ec2.SecurityGroup(this, "web-sg", {
      securityGroupName: `Web-app`,
      vpc,
    });

    webSecurityGroup.addIngressRule(
      webSecurityGroup,
      ec2.Port.tcp(2049) // Enable NFS service within security group
    );

    const fileSystem = new efs.FileSystem(this, "EfsFileSystem", { 
      vpc,
      securityGroup: webSecurityGroup
     });

    var accessPoint = new efs.AccessPoint(this, "volumeAccessPoint",  {
      fileSystem: fileSystem,
      path: "/data",
      createAcl: {
       ownerGid: "1000",
       ownerUid: "1000",
       permissions: "755"
      },
      posixUser: {
       uid: "1000",
       gid: "1000"
      }
   })    

    const efsVolumeConfiguration: ecs.EfsVolumeConfiguration = { 
      authorizationConfig: {
        accessPointId: accessPoint.accessPointId,
        iam: 'ENABLED',
      },      
      fileSystemId: fileSystem.fileSystemId,
      transitEncryption: 'ENABLED',
    };

    const ecsCluster = new ecs.Cluster(this, 'DefaultEcsCluster', { vpc });

    // Create Task Definition
    const webappTaskDef = new ecs.FargateTaskDefinition(
      this,
      "WebAppTaskDefinition",
      {
        family: `webapp`,
      }
    );

    const volume = {
      // Use an Elastic FileSystem
      name: "data",
      efsVolumeConfiguration
    };

    webappTaskDef.addVolume(volume)

    const repository = ecr.Repository.fromRepositoryName(
      this,
      "webapp-repository",
      "web-app"
    );

    const logging = new ecs.AwsLogDriver({ streamPrefix: 'webapp' });

    const webappContainer = webappTaskDef.addContainer("webappContainer", {
      image: ecs.ContainerImage.fromEcrRepository(repository),
      environment: {
        FILE_PATH: "/data"
      },
      logging
    });

    webappContainer.addMountPoints({
      readOnly:false,
      containerPath: '/data',
      sourceVolume: volume.name
    })


    webappContainer.addPortMappings({
      containerPort: 8080,
      protocol: ecs.Protocol.TCP
    });

    // Create Service
    const service = new ecs.FargateService(this, "Service", {
      cluster: ecsCluster,
      taskDefinition: webappTaskDef,
      securityGroups: [webSecurityGroup],
    });

    // Create ALB
    const lb = new elbv2.ApplicationLoadBalancer(this, 'LB', {
      vpc,
      internetFacing: true
    });

    const listener = lb.addListener('PublicListener', { 
      port: 80, 
      open: true,       
      // Default Target Group
      defaultAction: elbv2.ListenerAction.fixedResponse(200)
    });

    // Attach ALB to ECS Service
    listener.addTargets('ECS', {
      port: 80,
      healthCheck: {
        path: "/health",
        interval: Duration.seconds(30),
        timeout: Duration.seconds(3),
      },
      targets: [service]
    });

    new CfnOutput(this, 'LoadBalancerDNS', { value: lb.loadBalancerDnsName, });
  }
}

new EfsStack(app, 'EfsStack', {});
```

To deploy the `EfsStack` you need to run the following command 


```
cdk deploy EfsStack
```

It takes a few minutes to deploy all resources, and the output will be public DNS to the load balancer, which we can use to browse the web application.

Open that URL in the browser, and it should display the `Refresh count 1.` text on the first browse. If you refresh the page a few times, it will increase the number. Now to prove that this is persisted outside of the container, you can log in to your AWS console and navigate to the EFS, Cluster, then tasks and then stop the running task. You don't need to rerun the task. The cluster will automatically run a new task based on the task definition, as the desired count for this task is 1. If you attempt to refresh the page while the app is down, you'll get `503: Service Unavailable`. After a couple of minutes, a new task will start, and you can refresh the page. The counter will start counting from the last number it saved to EFS.

### Clean up

To avoid extra costs for running test resources in the AWS, you can run the following command to destroy the stack you've created. 

```
cdk deploy EfsStack
```
