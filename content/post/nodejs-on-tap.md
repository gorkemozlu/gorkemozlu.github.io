---
title: Developing and Deploying NodeJS with Data Services on TAP
date: 2022-12-10
tags:
  - "development"
  - "TAP"
  - "NodeJS"
  - "Data"
  - "Redis"
  - "MongoDB"
  - "PostgreSQL"
  - "MySQL"
categories:
  - "development"
  - "TAP"
  - "NodeJS"
  - "Data"
thumbnail: img/nodejs-on-tap/19-tap-gui-02.png
---
On this post, we'll go through a high level experience with a NodeJS app on Tanzu Application Platform. We're going to cover attaching Data Services on top of NodeJS app as well.
<!--more-->

Tanzu Application Service is a platform for building and running modern applications. It is built on top of Kubernetes and provides a range of tools and services for building, deploying, and managing applications. It allows developers to focus on writing code, while the platform handles the underlying infrastructure and support for scaling and rolling out applications. It also provides a variety of services and integrations for building and deploying applications, such as databases, messaging systems, and other common building blocks for modern applications.

This NodeJS application has multiple capabilities like:

- Show Kubernetes Node, pod, serviceaccount name, namespace etc. on main page.
- Create a file on PVC, add content and do backup&restore operations with Velero
- Have built-in interaction with stateful applications like:
  - Redis
  - MongoDB
  - MySQL
  - PostgreSQL

![image info](/img/nodejs-on-tap/01-acc-01.png)

As a developer, i'll go through my application catalog and choose my project which is "Sample NodeJS App with Data Bindings"

Then it will ask me several options.

![Alt text](/img/nodejs-on-tap/01-acc-01.png?raw=true "App Accelerator")

For example, my dev namespace is "my-apps" and i would like to use existing "Dockerfile" to build my application.

![Alt text](/img/nodejs-on-tap/03-acc-03.png?raw=true "App Accelerator")

It will ask for you to choose the git repo and branch name to pull the source code.

![Alt text](/img/nodejs-on-tap/04-acc-04.png?raw=true "App Accelerator")

Then, i'll choose App-SSO to bind a simple AuthN & AuthZ mechanism.

![Alt text](/img/nodejs-on-tap/05-acc-05.png?raw=true "App Accelerator")

Then, i'll integrate with several Data Services.

![Alt text](/img/nodejs-on-tap/06-acc-06.png?raw=true "App Accelerator")

After choosing whatever my application needs, i can either explore the file like Spring Initalizor or directly download the file to develop&run.

![Alt text](/img/nodejs-on-tap/07-acc-07.png?raw=true "App Accelerator")

After launching the vscode i can develop and run my app on dev cluster.
On the next posts, i'll share more on development capabilities of TAP, for now, i'll just deploy the workload.

```sh
$ kubectl apply -f config/tap
```
![Alt text](/img/nodejs-on-tap/08-app-01.png?raw=true "vscode")

Then, we'll see that Secure Supply Chain is running step by step.

![Alt text](/img/nodejs-on-tap/09-workload-01.png?raw=true "workload")

And, we'll see that source code and image has been scanned part of pipeline. It also gives us what are the CVEs that are affecting the workload. Since this is just a demo, my scan policy is not stopping the pipeline because of the CVEs like Critical, High level.

![Alt text](/img/nodejs-on-tap/09-workload-01.png?raw=true "workload")

![Alt text](/img/nodejs-on-tap/10-workload-02.png?raw=true "workload")

And a result, we can see that app is deployed through the pipeline.
s
![Alt text](/img/nodejs-on-tap/11-workload-03.png?raw=true "workload")

Also, we can see my k8s resources of that application and check pod logs etc from GUI.

![Alt text](/img/nodejs-on-tap/12-tap-gui-01.png?raw=true "tap-gui")

On top of that, we can see the application's architecture, like , what are the dependencies i have or what APIs do i have, etc.

![Alt text](/img/nodejs-on-tap/19-tap-gui-02.png.png?raw=true "tap-gui")

After i access to endpoint of the application, i can see that TAP App-SSO is asking my credentials. After logging in, app is ready to use with data bindings.

![Alt text](/img/nodejs-on-tap/13-app-01.png?raw=true "app")

![Alt text](/img/nodejs-on-tap/14-app-02.png?raw=true "app")

Mongo Connection:

![Alt text](/img/nodejs-on-tap/15-app-03.png?raw=true "app")

PostgreSQL Connection:

![Alt text](/img/nodejs-on-tap/16-app-04.png?raw=true "app")

MySQL Connection:

![Alt text](/img/nodejs-on-tap/17-app-05.png?raw=true "app")

Redis Connection:

![Alt text](/img/nodejs-on-tap/18-app-06.png?raw=true "app")

Of course, there are a lot more to cover for TAP, we'll continue on following posts.