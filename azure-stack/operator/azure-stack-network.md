---
title: Network integration planning for Azure Stack | Microsoft Docs
description: Learn how to plan for datacenter network integration with Azure Stack integrated systems.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''

ms.assetid: 
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/07/2019
ms.author: mabrigg
ms.reviewer: wamota
ms.lastreviewed: 06/04/2019
---

# Network integration planning for Azure Stack

This article provides Azure Stack network infrastructure information to help you decide how to best integrate Azure Stack into your existing networking environment. 

> [!NOTE]
> To resolve external DNS names from Azure Stack (for example, www\.bing.com), you need to provide DNS servers to forward DNS requests. For more information about Azure Stack DNS requirements, see [Azure Stack datacenter integration - DNS](azure-stack-integrate-dns.md).

## Physical network design

The Azure Stack solution requires a resilient and highly available physical infrastructure to support its operation and services. Uplinks from ToR to Border switches are limited to SFP+ or SFP28 media and 1 GB, 10 GB, or 25-GB speeds. Check with your original equipment manufacturer (OEM) hardware vendor for availability. The following diagram presents our recommended design:

![Recommended Azure Stack network design](media/azure-stack-network/recommended-design.png)


## Logical Networks

Logical networks represent an abstraction of the underlying physical network infrastructure. They're used to organize and simplify network assignments for hosts, virtual machines (VMs), and services. As part of logical network creation, network sites are created to define the virtual local area networks (VLANs), IP subnets, and IP subnet/VLAN pairs that are associated with the logical network in each physical location.

The following table shows the logical networks and associated IPv4 subnet ranges that you must plan for:

| Logical Network | Description | Size | 
| -------- | ------------- | ------------ | 
| Public VIP | Azure Stack uses a total of 31 addresses from this network. Eight public IP addresses are used for a small set of Azure Stack services and the rest are used by tenant VMs. If you plan to use App Service and the SQL resource providers, 7 more addresses are used. The remaining 15 IPs are reserved for future Azure services. | /26 (62 hosts) - /22 (1022 hosts)<br><br>Recommended = /24 (254 hosts) | 
| Switch infrastructure | Point-to-point IP addresses for routing purposes, dedicated switch management interfaces, and loopback addresses assigned to the switch. | /26 | 
| Infrastructure | Used for Azure Stack internal components to communicate. | /24 |
| Private | Used for the storage network and private VIPs. | /24 | 
| BMC | Used to communicate with the BMCs on the physical hosts. | /26 | 
| | | |

## Network infrastructure

The network infrastructure for Azure Stack consists of several logical networks that are configured on the switches. The following diagram shows these logical networks and how they integrate with the top-of-rack (TOR), baseboard management controller (BMC), and border (customer network) switches.

![Logical network diagram and switch connections](media/azure-stack-network/NetworkDiagram.png)

### BMC network

This network is dedicated to connecting all the baseboard management controllers (also known as BMC or service processors) to the management network. Examples include: iDRAC, iLO, iBMC, and so on. Only one BMC account is used to communicate with any BMC node. If present, the Hardware Lifecycle Host (HLH) is located on this network and may provide OEM-specific software for hardware maintenance or monitoring.

The HLH also hosts the Deployment VM (DVM). The DVM is used during Azure Stack deployment and is removed when deployment completes. The DVM requires internet access in connected deployment scenarios to test, validate, and access multiple components. These components can be inside and outside of your corporate network (for example: NTP, DNS, and Azure). For more information about connectivity requirements, see the [NAT section in Azure Stack firewall integration](azure-stack-firewall.md#network-address-translation).

### Private network

This /24 (254 host IPs) network is private to the Azure Stack region (doesn't expand beyond the border switch devices of the Azure Stack region) and is divided into two subnets:

- **Storage network**: A /25 (126 host IPs) network used to support the use of Spaces Direct and Server Message Block (SMB) storage traffic and VM live migration.
- **Internal virtual IP network**: A /25 network dedicated to internal-only VIPs for the software load balancer.

### Azure Stack infrastructure network
This /24 network is dedicated to internal Azure Stack components so that they can communicate and exchange data among themselves. This subnet can be routable externally of the Azure Stack solution to your datacenter, we do not recommend using Public or Internet routable IP addresses on this subnet. This network is advertised to the Border but most of its IPs are protected by Access Control Lists (ACLs). The IPs allowed for access are within a small range equivalent in size to a /27 network and host services like the [privileged end point (PEP)](azure-stack-privileged-endpoint.md) and [Azure Stack Backup](azure-stack-backup-reference.md).

### Public VIP network

The Public VIP Network is assigned to the network controller in Azure Stack. It's not a logical network on the switch. The SLB uses the pool of addresses and assigns /32 networks for tenant workloads. On the switch routing table, these /32 IPs are advertised as an available route via BGP. This network contains the external-accessible or public IP addresses. The Azure Stack infrastructure reserves the first 31 addresses from this Public VIP Network while the remainder is used by tenant VMs. The network size on this subnet can range from a minimum of /26 (64 hosts) to a maximum of /22 (1022 hosts). We recommend that you plan for a /24 network.

### Switch infrastructure network

This /26 network is the subnet that contains the routable point-to-point IP /30 (two host IPs) subnets and the loopbacks, which are dedicated /32 subnets for in-band switch management and BGP router ID. This range of IP addresses must be routable outside the Azure Stack solution to your datacenter. They may be private or public IPs.

### Switch management network

This /29 (six host IPs) network is dedicated to connecting the management ports of the switches. It allows out-of-band access for deployment, management, and troubleshooting. It's calculated from the switch infrastructure network mentioned above.

## Next steps

Learn about network planning: [Border connectivity](azure-stack-border-connectivity.md).
