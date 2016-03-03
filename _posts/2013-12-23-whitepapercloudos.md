---
layout: post
date: 2013-12-23
author: Marc van Eijk
title: Whitepaper Series on the CloudOS 
tags:
---
Windows Azure Pack is getting the attention it deserves. Microsoft is doing a great job in blogging extensively about the endless possibilities with this crucial piece in the CloudOS vision. Since the initial preview of Windows Azure Services for Windows Server, which was recently succeeded by Windows Azure Pack, I have done a couple of series on some inner working.

<img src="/images/2013-12-23/CloudOS.jpg" width="720">

These series were born because of personal interest or the amount of questions I got on a particular subject. I have worked closely with a lot of folks from Microsoft to get the most out of the solution. Their input, enthusiasm and willingness to help is just great. 

About two months ago I had a couple of chats with Kristian Nese and Flemming Riis, who recently released the [Hybrid Cloud with NVGRE (WSSC 2012 R2)](http://gallery.technet.microsoft.com/Hybrid-Cloud-with-NVGRE-aa6e1e9a) whitepaper. Somewhere in those chats we spoke about a follow up on their whitepaper. The idea quickly became a little more concrete in a series of whitepaper about the CloudOS. As a result, I have been working on the next whitepaper for the last couple of weeks. This whitepaper describes the installation steps for different deployment scenarios. After the initial deployment a lot of post deployment tasks can/must be completed. The whitepaper describes the following subjects.

- Introduction
- Components
- Designs
- Prerequisites
- Installation
- Authentication
- Administration
- Troubleshooting

Is this documentation not already available somewhere, you ask? Well, yes and no. Microsoft has done a good job with a lot of documentation on TechNet and the folks from the program team are blogging like maniacs. My dear friend [Hans Vredevoort](http://twitter.com/hvredevoort) had a great idea to start the [WAP Wiki](http://social.technet.microsoft.com/wiki/contents/articles/20689.windows-azure-pack-wapack-and-related-blogs-videos-and-technet-articles.aspx), that is the entry portal to all this information.

But we have been working with this product in actual deployments and have gained a lot of knowledge from that. The next whitepapers will be in line with the already released ones and will describe resource providers that can be configured in Windows Azure Pack. 

The last weeks I have done a lot of reading, testing and configuring with ADFS in combination with Windows Azure Pack. Every part that is unraveled is tested in a complete production ready environment. With High Availability at all levels, separate domains for security reasons, etc. This last piece of information was necessary for the whitepaper. To give you a small teaser for the whitepaper, a screenshot from the ADFS portal for Windows Azure Pack for Inovativ.

<img src="/images/2013-12-23/Inovativ-Portal.png" width="720">
