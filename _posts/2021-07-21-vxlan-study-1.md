---
layout: post
title: "VXLAN Study 1"
categories:
  - networking
tags:
  - topology
  - network
  - VXLAN
---

### VXLAN Study 1

-----
VXLAN 개념 공부 할 겸 RFC 번역해보기  
[RFC 7348](https://datatracker.ietf.org/doc/rfc7348/?include_text=1)


1. Abstract  
```
  This document describes Virtual eXtensible Local Area Network
   (VXLAN), which is used to address the need for overlay networks
   within virtualized data centers accommodating multiple tenants.  The
   scheme and the related protocols can be used in networks for cloud
   service providers and enterprise data centers.  This memo documents
   the deployed VXLAN protocol for the benefit of the Internet
   community.
```
> 해당 문서는 Virtual eXtensible Local Area Network (VXLAN) 에 대한 문서로, 가상화된 데이터센터의 multiple 테넌트 수용의 니즈를 해결하기 위한 주소 체계다.   
Cloud service 제공자와, enterprise 데이터센터에서 사용 될 프로토콜이며, VXLAN protocol 도입의 장점을 서술한 문서다.

  
2. Introduction
```
   Server virtualization has placed increased demands on the physical
   network infrastructure.  A physical server now has multiple Virtual
   Machines (VMs) each with its own Media Access Control (MAC) address.
   This requires larger MAC address tables in the switched Ethernet
   network due to potential attachment of and communication among
   hundreds of thousands of VMs.

   In the case when the VMs in a data center are grouped according to
   their Virtual LAN (VLAN), one might need thousands of VLANs to
   partition the traffic according to the specific group to which the VM
   may belong.  The current VLAN limit of 4094 is inadequate in such
   situations.

   Data centers are often required to host multiple tenants, each with
   their own isolated network domain.  Since it is not economical to
   realize this with dedicated infrastructure, network administrators
   opt to implement isolation over a shared network.  In such scenarios,
   a common problem is that each tenant may independently assign MAC
   addresses and VLAN IDs leading to potential duplication of these on
   the physical network.

   An important requirement for virtualized environments using a Layer 2
   physical infrastructure is having the Layer 2 network scale across
   the entire data center or even between data centers for efficient
   allocation of compute, network, and storage resources.  In such
   networks, using traditional approaches like the Spanning Tree
   Protocol (STP) for a loop-free topology can result in a large number
   of disabled links.

   The last scenario is the case where the network operator prefers to
   use IP for interconnection of the physical infrastructure (e.g., to
   achieve multipath scalability through Equal-Cost Multipath (ECMP),
   thus avoiding disabled links).  Even in such environments, there is a
   need to preserve the Layer 2 model for inter-VM communication.

   The scenarios described above lead to a requirement for an overlay
   network.  This overlay is used to carry the MAC traffic from the
   individual VMs in an encapsulated format over a logical "tunnel".

   This document details a framework termed "Virtual eXtensible Local
   Area Network (VXLAN)" that provides such an encapsulation scheme to
   address the various requirements specified above.  This memo
   documents the deployed VXLAN protocol for the benefit of the Internet
   community.
```
  
> 서버의 가상화로 인해, 물리적인 네트워크 인프라의 확장이 불가피해졌다. (가상화도 하나의 자원이므로, 인프라의 확장이 꼭 필요해짐을 의미하는 듯) 하나의 물리서버는, 각각의 MAC을 가진 VM 들을 가질 수 있게 되었다. 이는, 훨씬 많은 MAC을 학습할 수 있는 MAC 테이블이 필요하게 되었다는 뜻이다.

> VM들은 VLAN 내에서 그룹화 된다. 즉 하나의 VLAN은 많은 VM 들이 속해있을 수 있는데, 아시다시피, VLAN은 4094 개의 물리적 갯수 제한이 있다.

> 데이터센터는 이제, 여러 tenant들의 수용이 불가피해졌다.
-> tenant는 VM들의 네트워크 단위로 이해함
물리적인 4094개의 제한된 L2 네트워크는, 결국 한계에 달하게 될 것이다. 즉, 물리적인 Layer2 네트워크를, 물리 환경을 넘은 확장된 Layer2 네트워크로 만드는 것이 중요해졌다는 뜻이다.

> 이 Document 는 Overlay network 에 대한 시나리오를 설명할 것이다. Overlay, VM들의 MAC Address를 논리적인 "Tunnel"을 만들어 통신이 가능하게 할, VXLAN(Virtual eXtensible Local Area Network) 에 대해 알아보자.


3. 목차  
  
```
Mahalingam, et al.            Informational                     [Page 3]
RFC 7348                          VXLAN                      August 2014

1.1.  Acronyms and Definitions

   ACL      Access Control List

   ECMP     Equal-Cost Multipath

   IGMP     Internet Group Management Protocol

   IHL      Internet Header Length

   MTU      Maximum Transmission Unit

   PIM      Protocol Independent Multicast

   SPB      Shortest Path Bridging

   STP      Spanning Tree Protocol

   ToR      Top of Rack

   TRILL    Transparent Interconnection of Lots of Links

   VLAN     Virtual Local Area Network

   VM       Virtual Machine

   VNI      VXLAN Network Identifier (or VXLAN Segment ID)

   VTEP     VXLAN Tunnel End Point.  An entity that originates and/or
            terminates VXLAN tunnels

   VXLAN    Virtual eXtensible Local Area Network

   VXLAN Segment
            VXLAN Layer 2 overlay network over which VMs communicate

   VXLAN Gateway
            an entity that forwards traffic between VXLANs
```