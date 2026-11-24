---
title : "Amazon Lightsail Containers"
weight : 1
chapter : false
---

# Amazon Lightsail Containers

#### Overview

A **Lightsail container service** runs Docker containers for you at a fixed monthly price. You choose a size (the *power*) and a number of nodes (the *scale*), give Lightsail one or more container images, and Lightsail runs them behind a load balancer with an HTTPS public domain. You do not manage servers, clusters or load balancers.

In this workshop, in the **Singapore (ap-southeast-1)** Region, you:

1. Create a container service.
2. Deploy the official **nginx** image from a public registry.
3. Build your own image in **AWS CloudShell** and push it to the container service.
4. Deploy your image as a new version.
5. Clean up.

![Container service](/images/2/4-deploying.png)

#### Architecture

The diagram shows the flow of the lab. Visitors reach the container through the HTTPS public domain of the container service. Version 1 of the deployment pulls nginx from Docker Hub. You then build your own image in CloudShell, push it to the container service, and deploy it as version 2.

![Workshop architecture](/images/architecture.png)

#### What you need

- An AWS account and a user or role that can use **Amazon Lightsail** and **AWS CloudShell**.
- About 60 minutes.

You do **not** need a build server, an IAM user or access keys: CloudShell already has Docker, the AWS CLI and the permissions of your console session.

{{% notice warning %}}
The container service is billed from the moment you create it until you delete it, even if no container is running. This lab uses the **Nano** power (USD 7 per month, about USD 0.01 per hour). Do the [Clean up](5-clean-up/) chapter as soon as you finish.
{{% /notice %}}
