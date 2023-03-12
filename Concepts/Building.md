---
title: Builds and Deployments
layout: home
parent: Concepts
---

Almost always when operating infrastructure for various applications (validators, nodes, keepers, other bots, etc.) you are running opensource code. This is both a good and a bad thing. It is likely that when you first start operating the code you have not done a complete walkthrough and do not fully understand all the inctricies of the code (especially if you are a small team).

However there are there are some things you can do to make sure you are running the code you expect in a secure, reproducible and compatible way.

# Building
Generally you can build the code in a few different ways, either building from source, or running in a container. 

Building from source is the most flexible, but also the most time consuming. While various blockchain applications are generally developed in go (geth, cosmos, etc.) or rust (substrate, solana, etc.) building the applications is not standardized and thus leads to having to maintain many differentiated workflows and pipelines (not to mention the non-standardized deployment and operation of the applications). 

Often developers package their application into a container, publish it to a registry and allow operators to easily pull and run the application in an isolated enviroment. This is extremely helpful for both reproducible builds along with consistent operation and deployment. A developer packaging their application into a container (even if automated) does not mean that it is secure, and is the application specification that you are looking to operate. 

[Docker](https://www.docker.com/) is a containerization platform that allows you to package an application and all of its dependencies into a standardized unit for software development. Docker containers support signing of each container build but generally the signature verification is not enforced by default leaving a potential for malicious or unintended functions of the application. 

When deciding to run your infrastructure with a container first approach you should 
- Operate your own container registry
- Always build and sign your own containers from the source 
- After pulling the container, verify the signature of the container

# SAST and DAST Scanning 
It is important to understand the security of the application you are running. For many new protocols or applications AppSec might be an overlooked part of their CI/CD pipeline (or you might just want to verify on your own). Given all applications you run you build the containers from source it is easy to integrate an automated AppSec Pipeline. For a super light and simple way to get started with workflow you can utlize the Adjacent [build-template](https://github.com/adjacentresearchxyz/build-template) which will build, sign, and publish your container along with performing automated software composition testing and DAST scans.

Static Application Security Testing (SAST) and Dynamic Application Security Testing (DAST) are two types of security testing that can be used to identify security vulnerabilities in your application. 

SAST scans the source code of your application and looks for common security vulnerabilities. SAST is a great way to identify security vulnerabilities early in the development process, but it is not able to identify vulnerabilities that are introduced by the application’s runtime environment. 

DAST scans the running application and looks for common security vulnerabilities. DAST is a great way to identify security vulnerabilities that are introduced by the application’s runtime environment, but it is not able to identify vulnerabilities that are introduced by the source code.

These are both helpful tools to integrate into your pipeline whether you developed the application or it is opensource.

## Mirroring and Forking 
In order to maintain your own pipeline and build process you will need to fork/mirror or otherwise maintain the application you are looking to operate. Often this is not nesseacry. You generally fork an application if you are looking to contribute or change the application in a sigifigant way. You might want to mirror an application if you simply want to ensure that you will have access to the code if location is is hosted is to be taken down (an easy way to do this is by simply rehosting on ipfs with [git-protocol-rehost](https://github.com/adjacentresearchxyz/git-protocol-rehost)). 

Often it makes the most sense to create your workflow/pipeline and include the application you are looking to operate as a submodule. This allows you to 
- Maintain your own pipeline and build process
- Freeze the commit/version of the application you are looking to operate

Note: while this approach does allow you to maintain your own pipeline and build process it does not mirror the application repository. If the application repository is deleted you will no longer be able to fetch it (again why you should rehost with something like [git-protocol-rehost](https://github.com/adjacentresearchxyz/git-protocol-rehost) if you do not maintain your own git instance)
