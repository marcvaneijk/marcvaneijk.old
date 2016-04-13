---
layout: post
date: 2016-04-13
author: Marc van Eijk
title: Updated guidelines for Azure Resource Manager templates
tags: Azure Resource Manager, ARM, Guidelines, Best Practices
published: false
---
With the introduction of Azure Resource Manager at [Build in 2014](https://channel9.msdn.com/Events/Build/2014/2-607), 
Microsoft defined the next generation foundation for its public cloud platform. 
One of the core capabilities of Azure Resource Manager is template orchestration for all resources types in the platform. 
Enabling you to deploy any type of application (IaaS, PaaS or a combination) from a single template. 

To provide you with a range of sample templates, Microsoft started a community initiative for creating, 
maintaining and contributing templates in a public repository on GitHub. 
While this repository started with a modest set of templates, 
over time it grew to the most used reference and starting point for Azure Resource Manager Templates, containing over 330 templates. 

The templates in Azure Resource Manages are configured in the JavaScript Object Notation (JSON) language. 
Although these templates  must comply to a schema, there still is room to make design choices in your templates. 
What should be parameterized, how to use template functions, when to use nested templates. 
These are just a few choices you need decide on when you are creating your templates. 
To improve the standardization and reusability of the templates the repository contains a README file that is automatically displayed at the bottom the start page. 
The README file contains guidelines on contributing to the repository. 

## Learnings

With more people contributing to the repository new guidelines were added. 
The contributing process (Pull Requests) was integrated with Travis CI to improve the Pull Requests. 
Common issues with deployments of the templates were identified and guidelines to prevent these issues were also added to the README. 
Technical Preview 1 of Microsoft Azure Stack was introduced. 
The majority of the templates use hardcoded endpoint, requiring minor adjustments to the template before deploying them to Microsoft Azure Stack. 

## Guidelines and best practices

Together with a lot of passionate people from Microsoft that are involved with this topic, we were able to settle on an updated set of guidelines. 
These guidelines address the challenges we have learned so for. 
Improving the standardization and reusability of contributions to the repository. 
But more importantly it contains guidelines that help you create and maintain any ARM templates, whether you prefer to store them on GitHub or not.

The guidelines are split into three documents.

+ [**Contribution guide**](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/README.md). Describes the minimal guidelines for contributing.
+ [**Best practices**](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md). Best practices for improving the quality of your template design.
+ [**Git tutorial**](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/git-tutorial.md). Step by step to get you started with Git.

I can't stress this enough. You will gain benefit from creating templates that are compliant to these guidelines and best practices. The guidelines and best practices are based on the learnings from all the community members that contributed over 330 templates to the repository and from the result of the deployments of these templates on the Microsoft Azure platform. 
