---
layout: post
title: "VXLAN Study 2"
categories:
  - networking
tags:
  - topology
  - network
  - VXLAN
---

### VXLAN Study 2

-----
VXLAN 개념 공부 할 겸 RFC 번역해보기  
[RFC 7348](https://datatracker.ietf.org/doc/rfc7348/?include_text=1)

2. 생략

3. VXLAN Problem Statements
 - Limitation STP & VLAN Ranges
```
  3.  VXLAN Problem Statement

   This section provides further details on the areas that VXLAN is
   intended to address.  The focus is on the networking infrastructure
   within the data center and the issues related to them.

3.1.  Limitations Imposed by Spanning Tree and VLAN Ranges

   Current Layer 2 networks use the IEEE 802.1D Spanning Tree Protocol
   (STP) [802.1D] to avoid loops in the network due to duplicate paths.
   STP blocks the use of links to avoid the replication and looping of
   frames.  Some data center operators see this as a problem with Layer
   2 networks in general, since with STP they are effectively paying for
   more ports and links than they can really use.  In addition,
   resiliency due to multipathing is not available with the STP model.
   Newer initiatives, such as TRILL [RFC6325] and SPB [802.1aq], have
   been proposed to help with multipathing and surmount some of the
   problems with STP.  STP limitations may also be avoided by
   configuring servers within a rack to be on the same Layer 3 network,
   with switching happening at Layer 3 both within the rack and between
   racks.  However, this is incompatible with a Layer 2 model for inter-
   VM communication.

   A key characteristic of Layer 2 data center networks is their use of
   Virtual LANs (VLANs) to provide broadcast isolation.  A 12-bit VLAN
   ID is used in the Ethernet data frames to divide the larger Layer 2
   network into multiple broadcast domains.  This has served well for
   many data centers that require fewer than 4094 VLANs.  With the
   growing adoption of virtualization, this upper limit is seeing
   pressure.  Moreover, due to STP, several data centers limit the
   number of VLANs that could be used.  In addition, requirements for
   multi-tenant environments accelerate the need for larger VLAN limits,
   as discussed in Section 3.3.

```
```
IEEE 802.1D STP 는 루프를 방지하기 위한 프로토콜이다.

STP를 이용해, 루프를 방지하지만 사실, STP 때문에 네트워크 장비의 물리 포트를 100% 사용할 수 없다. 게다가, STP 에서는 Multipathing model을 지원하지 않는다.

STP의 결함을 보완하기 위해서 TRILL 이나 SPB 등의 multipathing이 되는 표준 규격도 있고,

또 Layer 3 네트워크를 이용해 STP 탈피가 가능하다. 어쨌거나 저쨌거나, Inter-VM 통신에서 Layer2 Model은 썩 쓸모 있어 보이진 않는다.

VLAN은, 12bit VLAN ID를 이용해, 브로드케스트 도메인을 나누는 기술이다. 알겠지만, 4094 개의 Limitation을 가지고 있다. 관련해선 3.3 섹션에서 자세히 다루도록 하겠다.

```
- Multi-Tenant Environments
```
3.2.  Multi-tenant Environments

   Cloud computing involves on-demand elastic provisioning of resources
   for multi-tenant environments.  The most common example of cloud
   computing is the public cloud, where a cloud service provider offers
   these elastic services to multiple customers/tenants over the same
   physical infrastructure.

   Isolation of network traffic by a tenant could be done via Layer 2 or
   Layer 3 networks.  For Layer 2 networks, VLANs are often used to
   segregate traffic -- so a tenant could be identified by its own VLAN,
   for example.  Due to the large number of tenants that a cloud

   provider might service, the 4094 VLAN limit is often inadequate.  In
   addition, there is often a need for multiple VLANs per tenant, which
   exacerbates the issue.

   A related use case is cross-pod expansion.  A pod typically consists
   of one or more racks of servers with associated network and storage
   connectivity.  Tenants may start off on a pod and, due to expansion,
   require servers/VMs on other pods, especially in the case when
   tenants on the other pods are not fully utilizing all their
   resources.  This use case requires a "stretched" Layer 2 environment
   connecting the individual servers/VMs.

   Layer 3 networks are not a comprehensive solution for multi-tenancy
   either.  Two tenants might use the same set of Layer 3 addresses
   within their networks, which requires the cloud provider to provide
   isolation in some other form.  Further, requiring all tenants to use
   IP excludes customers relying on direct Layer 2 or non-IP Layer 3
   protocols for inter VM communication.
```
```
클라우드 기술이 발전함에 따라, Multi-tenant 의 시대가 도래하게 되었다.
클라우드 service provider 는, Multi-tenant 서비스를 동일한 물리 인프라에서 서비스 하는데, 이는 Multi-tenant 환경의 대표적인 사례다.

Tenant 네트워크의 분리는, Layer2 단에서, 혹은 Layer3 단에서 구분한다.
Layer2는 4094개의 물리적인 한계가 명확한 솔루션인, VLAN이 대표적인 분리 방법이다. 
그렇다면, Tenant 네트워크 분리는 Layer3 단에서 하는 것이 가장 좋은 솔루션이냐? 하면 그것도 아니다. 두 개의 테넌트는 Layer3 단에서, 같은 네트워크를 가지는 경우가 있다. Layer3 를 이용한 분리는 IP 대역을 무조건 다르게 가져가야 하기 때문에, Layer 2 확장을 필요로 하는 고객들의 니즈를 만족하지 못한다.
-> Hyper Visor 별로 IP 대역이 다르게 되면, 이는 진정한 의미의 가상화가 될 수 없다는 뜻으로 이해
```
- Tor 스위치의 Table Size 한계
```
3.3.  Inadequate Table Sizes at ToR Switch

   Today's virtualized environments place additional demands on the MAC
   address tables of Top-of-Rack (ToR) switches that connect to the
   servers.  Instead of just one MAC address per server link, the ToR
   now has to learn the MAC addresses of the individual VMs (which could
   range in the hundreds per server).  This is needed because traffic
   to/from the VMs to the rest of the physical network will traverse the
   link between the server and the switch.  A typical ToR switch could
   connect to 24 or 48 servers depending upon the number of its server-
   facing ports.  A data center might consist of several racks, so each
   ToR switch would need to maintain an address table for the
   communicating VMs across the various physical servers.  This places a
   much larger demand on the table capacity compared to non-virtualized
   environments.

   If the table overflows, the switch may stop learning new addresses
   until idle entries age out, leading to significant flooding of
   subsequent unknown destination frames.
```
```
3.3 ToR 스위치의 Table Size 한계

하이퍼바이저의 스위치(ToR-Switch) 는, 각각의 VM들의 MAC을 모두 수용할 Mac Table 사이즈가 필요하다. VM 들 간의 통신을 하기 위해서, 하이퍼바이저가 연결 되어 있는 ToR 스위치의 Mac table은, 가상화 되어 있지 않은 환경보다 훨씬 많은 테이블 사이즈가 필요하다. 물리 서버 하나당 하나의 Mac이 아닌, 물리서버 하나에 올라간 여러 VM들의 mac주소들을 학습해야 하기 때문이다.

만약 Mac table이 넘치게 된다면, 스위치는 새로운 mac 주소에 대한 Learning을 하지 않을 것이고, 어마어마한 양의 unknown destination frame에 대한 flooding이 발생하게 될 것이다.
```