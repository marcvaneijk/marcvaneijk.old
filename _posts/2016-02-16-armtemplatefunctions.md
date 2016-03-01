---
layout: post
date: 2016-02-16
author: Marc van Eijk
title: Azure Resource Manager – Validate all template functions
tags:
---
In your deployment templates for Azure Resource Manager, you can use all kind of template functions. These functions can perform a variety of actions, from retrieving information to calculating numbers and changing string values. You can find a description of these template functions (including some examples) [here](https://azure.microsoft.com/en-us/documentation/articles/resource-group-template-functions/)

The list of functions retrieved from the article at the time of writing this blogpost are grouped in the following categories.

**Numeric functions**

- add
- copyIndex
- div
- int
- length
- mod
- mul
- sub

**String functions**

- base64
- concat
- padLeft
- replace
- split
- string
- substring
- toLower
- toUpper
- trim
- uniqueString
- uri

**Array functions**

- concat
- length

**Deployment value functions**

- deployment
- parameters
- variables

**Resource functions**

- listkeys
- providers
- reference
- resourceGroup
- resourceId
- subscription

Based the template functions documentation I have created five ARM deployment templates, that will execute each function for a given category. The templates will not create any resources (unless necessary for the function). You can use the output section see the result of the functions. The templates can be found in my GitHub repository here:

<https://github.com/marcvaneijk/arm/tree/master/000-patterns>

Besides the documentation I have also used the GitHub repo by Ryan Jones with a lot of great examples on template functions, containing a template per function.

<https://github.com/rjmax/ArmExamples>

These five templates successfully deploy to Microsoft Azure and can be useful to test a function and see the result. Because the templates do not deploy any resources, you can see the output in a couple of seconds.
