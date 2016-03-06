---
layout: post
date: 2013-09-20
author: Marc van Eijk
title: Virtual machine minimum bandwidth weight in a Format-Table output
tags: Bandwidth, ExpandProperty, Marc van Eijk, Minimum, Powershell, QoS, Weight
---
During my research for the last part of my [blog series on Bare Metal Deployment](/2013/10/05/bmd4) I wanted to get an overview of the weights that were assigned to all virtual machines on a host. What looked like a simple thing quickly became a little complex.

Let’s assume you have a Windows Server 2012 (R2) host running Hyper-V. On this host you have configured a NIC team with a vSwitch on top of that, either manually or with a logical switch from System Center Virtual Machine Manager 2012 SP1 or R2. The vSwitch is configured with the MinimumBandwidthMode set to Weight. In a weighted mode each virtual network adapter that has a weight assigned to it will have its minimum bandwidth mode calculated in percentage of the total bandwidth. For example if you have two virtual machines running on a vSwitch with the MinimumBandwidthMode set to Weight and each virtual machine has a weight of 3 they both have a minimum of 50% of the total bandwidth guaranteed. If you have fifty virtual machines running on a vSwitch with the MinimumBandwidthMode set to Weight and each virtual machine has a weight of 3, each virtual machine will have a minimum of 2% of the total bandwidth guaranteed.

If every virtual machine has the same weight assigned to it, the calculation is quite simple. It will become more complex if you have virtual machines with different weights. For example you have seven virtual machines with a minimum weight of 5, twenty-four virtual machines with a weight of 3 and eleven virtual machines with a weight of 1.

- High bandwidth adapter 7*5 = 35
- Medium bandwidth adapter 24*3 = 72
- Low bandwidth adapter 11*1 = 11
- Total weight 35 + 72 + 11 = 118

To get the percentage of a single vNIC, divide the weight of a single vNIC by the sum of all vNIC weights and multiple by 100. A virtual machine with a High bandwidth adapter would have a minimum bandwidth percentage of 5/118 * 100 = 4.24%

Virtual machines tend to move all over the place and to make it even more complex it is also possible to create vNICs accessible for the MangementOS. These vNICs can also have a minimum bandwidth value.

<img src="/images/2013-09-20/00-QOS.png" width=720">

Luckily for us is there is an easier way of getting the bandwidth information by calling on our good friend PowerShell. You can use the Get-VMNetworkAdapter cmdlet to retrieve a list of information about each vNIC (attached to a virtual machine or running in the MangementOS).

To get a list of all vNICs in the Management OS run the following cmdlet

```
Get-VMNetworkAdapter -ManagementOS
```

To get a list of all vNICs attached to virtual machines run the following cmdlet

```
Get-VMNetworkAdapter -VMName *
```

To get a list of all vNICs run the following cmdlet

```
Get-VMNetworkAdapter -All
```

Running any of these cmdlets with display the results in a table format with predefined set of properties. Nothing fancy yet. These predefined properties do not show any information about the bandwidth values. For this we will have to specify the properties we would like to have in our table. Usually you would then run the same command and pipe it to a Format-List.

```
Get-VMNetworkAdapter -VMName * | FL
```

The result of this cmdlet is a list properties for each virtual machine.

<img src="/images/2013-09-20/01-FL.png" width=720">

Near the bottom of the each list are a couple of bandwidth values. We are interested in MinimumBandwidthWeight and BandwidthPercentage. The name of the virtual machine might also come in handy. With this information we can run the command again, pipe it to a Format-Table and specify these properties.

```
Get-VMNetworkAdapter -VMName * | FT Name, MinimumBandwidthWeight, BandwidthPercentage
```

<img src="/images/2013-09-20/02-FT.png" width=720">

As you can see we now have a nice table displaying the calculated BandwidthPercentage values. But the MinimumBandwidthWeight is not showing any values. Despite the Format-List that did show the assigned weight.

So I tried to narrow down the Format-List command by specifying the same properties.

```
Get-VMNetworkAdapter -VMName * | FL Name, MinimumBandwidthWeight, BandwidthPercentage
```

<img src="/images/2013-09-20/03-FL-with-Properties.png" width=720">

This doesn’t make any sense. I asked [Jeff Wouters (MVP Powershell)](https://twitter.com/JeffWouters) if he had some pointers for me. His quick and effective answer:

> Sounds like the display name and the real name of the properties aren’t the same. To retrieve the real name run the cmdlet 
> `Get-VMNetworkAdapter -VMName * | Get-Member`

<img src="/images/2013-09-20/04-Get-Member.png" width=720">

Jeff was right, the property BandwidthPercentage was in the list of available properties but the MinimumBandwidthWeight was not. Another property called BandwidthSetting caught my attention so I tried to run the cmdlet by using this property.

```
Get-VMNetworkAdapter -VMName * | FT Name, BandwidthSetting, BandwidthPercentage
```

<img src="/images/2013-09-20/05-Alias.png" width=720">

Mm.. That was not the result I hoped for. I decided to do some PowerShell reading again. Finally I found a parameter called [–ExpandProperty](http://technet.microsoft.com/en-us/library/hh849895.aspx).

> if the specified property is an array, each value of the array is included in the output

That sounds like the result I’m looking for. But the -ExpandProperty is a parameter of the Select-Object cmdlet. Let’s try to pipe the Get-VMNetworkAdapter -VMName * cmdlet to the Select-Object cmdlet referencing the BandwidthSetting property from the Get-member cmdlet and then pipe that to a Format-Table.

```
Get-VMNetworkAdapter -VMName * | Select-Object Name, bandwidthpercentage -ExpandProperty Bandwidthsetting | FT –AutoSize
```

<img src="/images/2013-09-20/06-ExpandProperty-and-other-properties.png" width=720">

And there we have the result we were looking for. You can add additional properties. For example:

```
Get-VMNetworkAdapter -VMName * | Select-Object name, MacAddress, SwitchName, bandwidthpercentage -ExpandProperty Bandwidthsetting | FT –AutoSize
```

<img src="/images/2013-09-20/07-ExpandProperty-and-other-properties.png" width=720">

For a Powershell MVP like Jeff this is probably not a complicated thing, but all the blogs if have read with a Get-VMNetworkAdapter referencing the MinimumBandwidthWeight in a Fortmat-Table display no values. Thought this one might come in handy.

<img src="/images/2013-09-20/Twitter.png" width=720">
