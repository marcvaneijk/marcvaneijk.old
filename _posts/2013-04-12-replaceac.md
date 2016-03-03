---
layout: post
date: 2013-04-12
author: Marc van Eijk
title: Will Windows Azure for Windows Server replace System Center App Controller? 
tags: App Controller 2012, KATAL, Management Summit, Windows Azure, Windows Azure for Windows Server
---
The Microsoft Management Summit 2013 just ended and all recorded sessions are already available on [Channel9](http://channel9.msdn.com/Events/MMS/2013). Whether you are interested in the latest developments, lessons learned by early adopters or in-depth demos these session recordings will provide you with great insight into Private, Hosted and Public Cloud solutions by Microsoft. I did not attend the MMS 2013 and therefore I am very grateful to have access to all this magnificent content online.

As you might have noticed from my previous blogs I have a great interest in Windows Azure for Windows Server. In session [WS-B303 Windows Server Virtual Machine: Adding Windows Azure Services](http://channel9.msdn.com/Events/MMS/2013/WS-B303) Program Managers Marc Umeno and Anjli Chaudhry explain the components, lessons learned (some of them looked very familiar :)) and some demos.

One slide caught my attention.

<img src="/images/2013-04-12/AD-Integration.png" width="720">

In this slide Marc Umeno talks about an upcoming development in Windows Azure For Windows Server. In the current version user accounts are stored in an ASP.NET membership SQL database. This is a great solution for Service Providers, but (except for the Admin Site) there is no integration with Active Directory.

The product team is working on Active Directory integration for a future release.

What users will logon to the portal with Active Directory accounts? Users from the internal organization.

If you think about it, it is a logical step. Whether self-service users manage their services in Windows Azure, in a hosted cloud or in their own private cloud, they can access them through a uniform portal. It also fits in the roadmap to drive the adoption of Windows Azure in a great way.

Where does this leave System Center App Controller? Maybe the product team working on Windows Azure for Windows Server might be reinforcement with the System Center App Controller product team. At the end of the session Marc Umeno specifies that at [TechEd (taking place June 3-6, this year)](http://northamerica.msteched.com/) more information will be disclosed. So stay tuned.
