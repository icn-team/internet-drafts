---
title: "Anchorless mobility management through hICN (hICN-AMM): Deployment options"
abbrev: "hICN mobility: deployment options"
docname: draft-auge-dmm-hicn-mobility-deployment-options-03

date:
category: info

ipr: trust200902
area: general
workgroup: DMM Working Group
keyword: Internet-Draft
keyword: Information-Centric Networking
keyword: Hybrid ICN
keyword: Anchorless mobility

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:

-
    ins: J. Augé
    name: Jordan Augé
    org: Cisco Systems
    abbrev: Cisco
    street:
    - 11, rue Camille Desmoulins
    city: Issy-les-Moulineaux
    code: '92130'
    country: France
    email: augjorda@cisco.com
-
    ins: G. Carofiglio
    name: Giovanna Carofiglio
    org: Cisco Systems
    abbrev: Cisco
    street:
    - 11, rue Camille Desmoulins
    city: Issy-les-Moulineaux
    code: '92130'
    country: France
    email: gcarofig@cisco.com
-
    ins: L. Muscariello
    name: Luca Muscariello
    org: Cisco Systems
    abbrev: Cisco
    street:
    - 11, rue Camille Desmoulins
    city: Issy-les-Moulineaux
    code: '92130'
    country: France
    email: lumuscar@cisco.com
-
    ins: M. Papalini
    name: Michele Papalini
    org: Cisco Systems
    abbrev: Cisco
    street:
    - 11, rue Camille Desmoulins
    city: Issy-les-Moulineaux
    code: '92130'
    country: France
    email: micpapal@cisco.com

informative:
  TS.23.501-3GPP:
    title: "System Architecture for 5G System; Stage 2, 3GPP TS 23.501 v2.0.1"
    author:
      - org: 3rd Generation Partnership Project (3GPP)
    date: December 2017
  TR.29.892-3GPP:
    title: "Study on User Plane Protocol in 5GC, 3GPP TR 29.892 (study item)"
    author:
      - org: 3rd Generation Partnership Project (3GPP)
    date: 2017
  VPP:
    title: Vector Packet Processing (VPP)
    author:
    - ins: "FD.io (Linux Foundation)"
    seriesinfo:
        url: https://wiki.fd.io/view/VPP
  WLDR: DOI.10.1145/2984356.2984361
  CICN:
    title: CICN project
    author:
    - ins: "Linux Foundation fd.io"
    date: 2018


--- abstract

A novel mobility management approach is described in {{?I-D.auge-dmm-hicn-mobility}},
that leverages routable location-independent identifiers (IDs) and an
Information-Centric Networking (ICN) communication model integrated in IPv6,
also referred to as Hybrid ICN (hICN) {{?I-D.muscariello-intarea-hicn}}.

Such approach belongs to the category of pure ID-based mobility management
schemes whose objective is (i) to overcome the limitations of traditional
locator-based solutions like Mobile IP, (ii)
to remove the need for a global mapping system as the one required by
locator-identifier separation solutions like LISP
{{?I-D.ietf-lisp-introduction}} or ILA {{?I-D.herbert-intarea-ila}}.

ID-based networking as proposed by ICN architectures allows to disentangle
forwarding operations from changes of network location, hence removing tunnels
and user plane or control plane anchors. In virtue of its anchorless property,
we denote this approach as hICN-AMM (hICN Anchorless Mobility Management)
hereinafter.

This document discusses hICN-AMM deployment options and related tradeoffs in
terms of cost/benefits. Particular attention is devoted to the insertion in the
recently proposed 5G Service Based Architecture under study at 3GPP where
an hICN-AMM solution might present a more efficient alternative to the traditional
tunnel-based mobility architecture through GTP-U.

--- middle

# Introduction

The steady increase in mobile traffic observed in the recent years has come with
a growing expectation from next generation 5G networks of a seamless
latency-minimal service experience over multiple heterogeneous access networks.
Motivated by stringent next-generation applications requirements and by the
convergence of diverse wireless radio accesses, 5G standardization bodies have
encouraged re-consideration of the existing Mobile IP and GTP-based architecture
aiming at a simplified and sustainable mobility solution.

Specifically, enhanced user plane solutions are currently investigated where
GTP-U tunnels would be replaced by more flexible and dynamic forwarding schemes
tailored to application needs and enabling novel use cases such as
caching/computing at the edge (MEC/FOG), coordinated use of multiple network
accesses in parallel, etc.

It is commonly accepted that the inefficiencies of tunnels in terms of state and
network overhead, as well as the presence of anchors in the data plane are a
direct consequence of using locators as identifiers of mobile
nodes/services. This has motivated the introduction of a class of approaches
distinguishing locators and identifiers, known as Loc/ID split, and more
recently of pure ID-based approaches inspired from name-based
Information-Centric Networking (ICN) architectures.

The focus of this document is on the latter category of solutions and more
precisely on the hICN-AMM proposal in {{?I-D.auge-dmm-hicn-mobility}}, that aims at a
simplified anchorless mobility management through Hybrid ICN (hICN), a
fully-fledged ICN architecture in IPv6 {{?I-D.muscariello-intarea-hicn}}.

hICN-AMM benefits result both from the flexibility of pure ID-based mobility
solutions, that completely decouple data delivery from underlying network
connectivity, and from the native mobility support of ICN architectures.

Forwarding operations are performed directly based on location-independent IDs
stored in routers' FIBs and no mapping of ID into locators is required. In this
way, purely ID-based architectures remove the need to maintain a global mapping
system at scale, and its intrinsic management complexity.

Additional benefits brought by ICN communication model include:

- the flexibility of multi-source/multi-path connectionless pull-based
transport. An example is the native support for consumer mobility, i.e. the
transparent emission of data requests over multiple and varying available
network interfaces during node mobility;

- the opportunity to define fine-grained per-application forwarding and
security policies.

An overview of ICN principles and advantages for a simplified mobility
management resulting from name-based forwarding can be found in {{!RFC7476}},
while a detailed description of the proposal is presented in
{{?I-D.auge-dmm-hicn-mobility}}.

In this document, we discuss the integration of hICN-AMM proposal within an
existing mobile network architecture and analyze the resulting tradeoffs in
terms of cost/benefits. The 3GPP 5G architecture is used here as a reference as
the envisaged target for hICN-AMM insertion.

After a short overview of 5G architecture, we first discuss hICN-AMM insertion
with no modification to the existing 3GPP architecture. We then consider the
additional benefits emerging from a tighter integration involving optimization
of data plane interfaces. Finally, we consider the impact of different degrees
of hICN penetration, ranging from end-to-end to fully network-contained
deployments.


# Overview of a mobile access network and the 5G architecture

{{fig-mobile-network}} schematizes a typical mobile network composed of  edge,
backhaul and core network segments. In the rest of this document, we assume an
IPv6-based network.

~~~~
                 , - ,              , - ,              , - ,
Cell site    , '       ' ,      , '       ' ,      , '       ' ,
    __     ,               ,__,                ,__,               ,
 __/ A\   ,                /X/|                /X/|                ,
/ A\__/  __     IP edge    |_|/  IP backhaul   |_|/    IP core     __
\__/ A\ /X/|               , ,                 , ,                /X/|
/ A\__/ |_|/    network    ,__     network     ,__     network    |_|/
\__/ A\  |,                /X/|                /X/|                |
   \__/  | ,               |_|/,               |_|/,              ,|
         |   ,          , ' |   ,          , '' |   ,          , ' |
         |     ' - ,  '     |     ' - ,  '      |     ' - ,  '     |
       +-+-+              +-+-+               +-+-+              +-+-+
       |   |              |   |               |   |              |   |
       |DDC|              |DDC|               |DDC|              |DDC|
       |   |              |   |               |   |              |   |
       +---+              +---+               +---+              +---+
~~~~
{: #fig-mobile-network title="Overview of a typical mobile network architecture" }

With the adoption of virtualization and softwarization technologies, services
are increasingly hosted in Distributed Data Centers (DDC) deployed at the
network edge (Mobile Edge Computing, MEC).

5G network has been designed around such evolved Service Based Architecture
(SBA) {{TS.23.501-3GPP}}, exploiting SDN and NFV capabilities. It consists in a
set of modular functions with  separation of user and control
planes. {{fig-5g-sba}} gives the traditional representation of this architecture
(as of Release 15) in its service-based representation.

~~~~
                    Service Based Interfaces
+-------+ +-------+ +-------+ +-------+ +-------+ +-------+ +-------+
| NSSF  | |  NRF  | |  DSF  | |  UDM  | |  NEF  | |  PCF  | |  AUSF |
+---+---+ +---+---+ +---+---+ +---+---+ +---+---+ +---+---+ +---+---+
    |         |         |         |         |         |         |
----+------+--+---------+---------+-------+-+---------+---------+-----
           |                              |
  +--------+---------+          +---------+--------+      MOBILITY
  |        AMF       |          |        SMF       |     MANAGEMENT
  +-+--------------+-+          +++--------------+-+
    |              |              |              |        control
    | N1           | N2           | N4           | N4      plane
 ...|..............|..............|..............|....................
    |              |              |              |       user plane
    |              |              |              |
+---+---+      +---+---+  N3  +---+---+  N9  +---+---+  N6  +-------+
|  UE   +------+  gNB  +------+  UPF  +------+  UPF  +------+  DN   |
+-------+      +-------+      +---+---+      +-------+      +-------+

~~~~
{: #fig-5g-sba title="5G reference architecture" }

The mobility management subsystem corresponds to the bottom part of the picture
and distinguishes user and control planes.

- the __user plane__ (UP) represents the path followed by user traffic, between
the mobile node or User Equipment (UE), and the Data Network (DN) corresponding
to the egress point interfacing the ISP network and the Internet. The UP
consists of the Radio Access Network (RAN), terminated by the gNB function, and
of a series of User Plane Functions (UPFs) which are generic functions to
transport/ process the traffic up to the egress point. In particular, UPFs serve
as Layer 2 and Layer 3 anchors respectively in the RAN, and at the interface
with the DN (to advertise IP prefixes). The interfaces between gNB and UPF (N3),
and in-between UPFs (N9) are those where GTP tunnels are currently established,
and where there are opportunities for optimization.

- the principal __control plane__ functions are the Access Management Function
(AMF) which is responsible for the radio access, and the Session Management
Function (SMF) which is taking care of the sessions established between a UE and
a DN.

The resulting protocol stack between user plane components is represented in
{{fig-protocol-stack}}. It illustrates the encapsulation overhead of this
solution at the various UPFs, and well as tunnel inefficiencies due to the
transport of all mobile traffic up to the anchor in the core (N6), the latter
preventing convergence with non-3GPP network accesses.

~~~~
    UE            5G-AN        N3         UPF        N9   UPF    N6
                               |                     |            |
+--------+                     |                     |            |
|  App.  |--------------------------------------------------------|
+--------+                     |                     | +--------+ |
| IP PDU |---------------------------------------------| IP PDU | |
+--------+ +-----------------+ | +-----------------+ | +--------+ |
|        | |\     relay     /| | |\     relay     /| | |        | |
|        | | \_____________/ |-|-| \_____________/ |-|-|        | |
|        | |        | GTP-U  | | | GTP-U  |  GTP-U | | |  GTP-U | |
|        | |        +--------+ | +--------+--------+ | +--------+ |
|   5G   | |   5G   |  UDP   |-|-|  UDP   |   UDP  |-|-|   UDP  | |
|   AN   |-|   AN   +--------+ | +--------+--------+ | +--------+ |
|protocol| |protocol|   IP   |-|-|   IP   |   IP   |-|-|   IP   | |
| layers | | layers +--------+ | +--------+--------+ | +--------+ |
|        | |        |   L2   |-|-|   L2   |   L2   |-|-|   L2   | |
|        | |        +--------+ | +--------+--------+ | +--------+ |
|        | |        |   L1   |-|-|   L1   |   L1   |-|-|   L1   | |
+--------+ +-----------------+ | +-----------------+ | +--------+ |
                               |                     |            |
~~~~
{: #fig-protocol-stack title="5G protocol stack" }


# Transparent integration (option #1)

This section discusses the insertion of hICN-AMM in an unmodified 3GPP 5G
reference architecture, where GTP tunnels are preserved. As previously stated,
maintaining GTP tunnels does not allow to overcome limitations of anchor-based
approaches. However, a transparent integration of hICN-AMM limits to the minimum
deployment costs and already brings advantages over the baseline architecture.
We consider two cases: a) hICN-AMM deployment within Mobile Edge Computing (MEC)
platforms, exploiting the local breakout capabilities of 5G networks, b)
hICN-AMM deployment as User Plane Function (UPF) inside mobile user plane.

## MEC deployment (option #1a)

Option #1a refers to the deployment of an hICN forwarder within Mobile Edge
Computing platforms. It relies on the local breakout capability introduced in
5G, i.e. a specific UPF denoted UL/CL (uplink classifier) that locally diverts
specific flows to an alternative DN, filtering packets based for instance on
information carried in packet headers.

In the hICN-AMM case, this function is used to realize the hICN punting function
described in {{?I-D.muscariello-intarea-hicn}}, i.e. to identify hICN traffic
(Interest and Data packets) and forward it to the local MEC hICN instance.

The MEC deployment is illustrated in {{fig-hicn-sba-mec}}. It is worth noticing
that option #1a shares similarities with the deployment approach for ID/Loc
solutions proposed in {{?I-D.homma-dmm-5gs-id-loc-coexistence}}.

~~~~
  +------------------+          +------------------+
  |        AMF       |          |        SMF       |
  +-+--------------+-+          +++--------------+-+
    |              |             .|              |
    | N1           | N2       N4 .| N4           | N4
    |              |             .|              |
+---+---+      +---+---+  N3  +---+---+  N9  +---+---+  N6  +-------+
|  UE   +------+  gNB  +------+ UL/CL +------+  UPF  +------+  DN   |
+-------+      +-------+      +---+---+      +-------+      +-------+
                                 .|
                                 .| N9
                                 .|                            _
                              +--++---+  N9  +-------+  MEC  _( )___
                              |  UPF  +------+   DN  +------(  hICN )_
                              +-------+      +-------+      (_________)
                                 ^               ^              ^
                                 |_______________|              |
                                   local breakout        hICN insertion
~~~~
{: #fig-hicn-sba-mec title="hICN insertion in MEC" }

Although it preserves tunnels and anchor points, option #1a permits an early
termination of tunnels and the distribution of hICN capabilities close to the
edge like in path caching and rate/loss/congestion control which may be
leveraged for efficient low-latency content distribution, especially in presence
of consumer mobility. Let us analyze more in details such advantages.

__Pro: Benefits of edge caching__

The ability for the UE to communicate with an endpoint close to the access is a
pre-requisite of latency-sensitive and interactive applications such as AR/VR.
Option #1a avoids forwarding of high-throughput traffic into the core, when it
can be terminated locally at MEC.

hICN ability to take dynamic hop-by-hop forwarding decisions facilitates
edge/core traffic load-balancing. In addition, the pull-based transport model
enables request aggregation removing traffic redundancy and facilitates the use
of dynamically discovered sources. This is the case of requests directly
satisfied from locally buffered temporary copies. In this way, hICN implicitly
realizes opportunistic multicast distribution of popular content with benefits
for improved users' QoE for services such as VoD or live streaming as well as
reduced traffic cost by offloading the mobile core.

We remark that for caching of data or of requests to be effective, a sufficient
level of aggregation is required: one can thus expect hICN instances to be
useful in the backhaul and core locations (with an eventual hierarchy of
caches), while low-latency applications will benefit from deployments
close to the cell site.

__Pro: Consumer mobility improvements__

Consumer mobility is natively supported in ICN. It is sufficient for a mobile UE
to keep sending interests as soon as it connects to a new network interface. The
combined use of identifiers and of a pull-model in hICN-AMM allows for seamless
mobility across heterogeneous wired and wireless networks, as well as their
simultaneous use for multi-homing and bandwidth aggregation.

{{fig-hicn-wifi}} illustrates a mobility scenario considering a 3GPP access and
a non-3GPP WiFi access tunneled to a N3 Inter-networking Function (N3IWF)
offering a N3 interface for connection to the first UPF. In this setting, both
wireless accesses are assumed to be connected through the same backhaul link,
and thus to converge into the same first UPF.

Placing an hICN instance at the aggregation of the two accesses would be
beneficial both in terms of mobility and of local loss recovery, as a lost
packet would be fast retransmitted directly from local hICN instance.

~~~~
tunnelled non-3GPP radio
(e.g. WiFi)   +------+  N3
       +------+ N3IWF+-------+                         hICN
      /       +------+        \                          +
     /                         \                         |
+---+---+     +-------+ N3 +---+---+ N9 +---+---+ N6 +---+---+
|  UE   +-----+  gNB  +----+ UL/CL +----+  UPF  +----+  DN   +---+ hICN
+-------+     +-------+    +---+---+    +-------+    +-------+
~~~~
{: #fig-hicn-wifi title="Example of a UE connection to two local radio accesses"}

__Pro: Multihoming and multi-source communications__

A specific feature of hICN transport is to control the use of multiple
path/sources of content at the same time, for instance caches located at the
edge of different fixed/mobile network accesses. The consumer is able to use
both network accesses in parallel adapting load-balancing on a per-packet
granularity, based on network conditions in order to exploit the full aggregated
bandwidth and/or to compensate variations induced by UE mobility or by random
fluctuations of individual radio channel conditions.

{{fig-hicn-multisource}} presents the case of a mobile consumer fetching data
simultaneously from two distinct sources, for instance local caches behind each
radio access.

~~~~
tunnelled non-3GPP radio
(e.g. WiFi)  +-------+ N3 +-------+ N9 +-------+ N6 +-------+
       +-----+ N3IWF +----| UL/CL +----+  UPF  +----+  DN   +----+ hICN
      /      +-------+    +-------+    +-------+    +-------+
     /
+---+---+    +-------+ N3 +-------+ N9 +---+---+ N6 +---+---+
|  UE   +----+  gNB  +----+ UL/CL +----+  UPF  +----+  DN   +----+ hICN
+-------+    +-------+    +---+---+    +-------+    +-------+
~~~~
{: #fig-hicn-multisource title="Multi-source communication with hICN"}


<!--
_Limitations_

Because the hICN forwarders are placed in MEC DN outside 5G mobile core, this solution is
less adapted for handling producer mobility : coordination between hICN
forwarders would then be required involving interaction with SMF.

Offloading of local communications is limited by the positioning of the edge
cloud, and it would not be possible for instance to have two users of the same
cell site to directly communicate together due to the lack of hICN support on
cell side (device-to-device communication with cell support).
-->

__Con: Remaining IP anchors hinder producer mobility__

hICN-AMM MEC deployments are not suitable for handling producer mobility
effectively due to the presence of an anchor node in the core at the N6
interface which traffic has to pass through. An alternative would be to put in
place dedicated control mechanisms to update routers FIB following mobility
events, but such coordination between hICN forwarders would involve interaction
with SMF.

__Con: Loss of 3GPP control plane__

More generally, traffic in between forwarders in the MEC will be out of reach
for the mobile core. At the time of this writing, MEC interfaces are not well
standardized; slicing and QoS functions will only be available up to the
breakout point.


## hICN deployment as a UPF (option #1b)

The design of hICN makes it a good candidate for being deployed as a UPF in
NFV environments, for instance within a container (see {{fig-hicn-sba-upf}}.
This solution has the advantage of preserving the interface with the 3GPP
control plane, including support for slicing or QoS.

~~~~
  +------------------+          +------------------+
  |        AMF       |          |        SMF       |
  +-+--------------+-+          +-+--------------+-+
    |              |              |              |
    | N1           | N2           | N4           | N4
    |              |              |              |
+---+---+      +---+---+  N3  +---+---+  N9  +---+---+  N6  +-------+
|  UE   +------+  gNB  +------+  UPF  +------+  UPF  +------+  DN   |
+-------+      +-------+      +-------+      +-------+      +-------+
                                 ^                ^
                                 |________________|
                                   hICN insertion
~~~~
{: #fig-hicn-sba-upf title="hICN insertion inside UPFs }


__Improved traffic engineering__

On-path deployments of hICN-AMM, as early as at the first UPF, bring additional
advantages in terms of opportunities to optimize traffic engineering, including
multipath and load balancing support.

As an example, one may consider hICN deployment at the PDU Session Anchor, with
dynamic congestion-aware and per-application control of multiple downstream
paths to the edge.

__Network-assisted transport__

hICN ability to provide in-network assistance for rate/loss/congestion control
is an additional advantage, e.g. to support rate adaptation in the case of
dynamic adaptive streaming, or to improve reliability of WiFi communications via
local wireless detection and recovery {{WLDR}}.

We illustrate the latter case in {{fig-hicn-wldr}}, where an hICN UPF is
deployed as the first UPF following the WiFi access. hICN UPF can exploit
network buffers to realize loss detection and recovery of the packet transiting
on the unreliable radio access, thus offering superior performance for WiFi
traffic with respect to a traditional end-to-end recovery approach. Such feature
could be fundamental for low-latency reliable services.

~~~~
                             +-- Loss Detection and Recovey
                             V
             +-------+ N3 +-------+
       +-----+ N3IWF +----| hICN  |
      /      +-------+    +---+---+
     /                     N9 |
+---+---+    +-------+ N3 +---+---+ N9 +---+---+ N6 +---+---+
|  UE   +----+  gNB  +----+  UPF  +----+  UPF  +----+  DN   +----+ hICN
+-------+    +-------+    +---+---+    +-------+    +-------+
~~~~
{: #fig-hicn-wldr title="In-network loss detection and recovery with hICN"}


__Producer mobility__

Option #1b is appropriate for handling producer mobility; however, it would be
supported through traditional DMM approaches for identifier-based mobility.


## Summary

This section has reviewed various insertion strategies for hICN, including
overlay deployments using local breakout to hICN instances situated in MEC, or
hICN forwarders deployed within an UPF. While those approaches have the merit of
allowing an easy or early integration of hICN and exploiting some of its
benefits, they do not fully exploit purely ID-based capabilities nor the dynamic
hICN forwarding.

# "Integrated" approaches: data-plane alternatives to GTP-U (option #2)

The section describes more integrated hICN-AMM/3GPP 5G approaches leveraging
hICN capabilities in the mobile backhaul network in alternative to GTP-U tunnels
over N9 (and possibly N3) interface(s), as shown in {{fig-5g-sba}} and
{{fig-protocol-stack}}.

It is proposed to replace GTP tunnels at N9 and optionally at N3 interfaces by
an hICN-enabled data plane operating on location-independent identifiers
advertised by the routing protocol and whose forwarding information is
stored/updated in routers' FIBs according to the hICN-AMM approach.

We remind that an hICN data plane does not require the presence of a mapping
system nor the enablement of all routers, since hICN traffic is transparently
forwarded by regular hICN-unaware IP routers.

## hICN insertion at N9 interface only (option #2a)

Option #2a answers the question of the ongoing 3GPP study of alternative
technologies for the N9 interface {{TR.29.892-3GPP}}. with no impact on gNB
as illustrated in {{fig-hicn-sba-n9}}. The corresponding protocol layering is
shown in {{fig-hicn-prot-n9}} where we assume hICN-enablement of the end-points
(an alternative in-network hICN enablement via proxies is possible but not
 considered by this document). In the protocol layer, hICN is associated to IPv6
PDU layer, transported over N9 directly over L2.

~~~~
  +------------------+          +------------------+
  |        AMF       |          |        SMF       |
  +-+--------------+-+          +-+--------------+-+
    |              |              |              |
    | N1           | N2           | N4           | N4
    |              |              |              |
+---+---+      +---+---+  N3  +---+---+  N9  +---+---+  N6  +-------+
|  UE   +------+  gNB  +------+  UPF  +------+  UPF  +------+  DN   |
+-------+      +-------+      +-------+      +-------+      +-------+
                                         ^
                                         |
                                   hICN insertion
~~~~
{: #fig-hicn-sba-n9 title="Replacement of N9 interface" }

~~~~
    UE            5G-AN        N3         UPF        N9   UPF    N6
                               |                     |            |
+--------+                     |                     |            |
|  App.  |--------------------------------------------------------|
+--------+                     |                     | +--------+ |
| IP PDU |                     |                     | | IP PDU | |
| (hICN) |---------------------------------------------| (hICN) | |
+--------+ +-----------------+ | +-----------------+ | |        | |
|        | |\     relay     /| | |\     decap     /  | |        | |
|        | | \_____________/ |-|-| \_____________/   | |        | |
|        | |        | GTP-U  | | | GTP-U  |          | |        | |
|        | |        +--------+ | +--------+          | |        | |
|   5G   | |   5G   |  UDP   |-|-|  UDP   |          | |        | |
|   AN   |-|   AN   +--------+ | +--------+          | |        | |
|protocol| |protocol|   IP   |-|-|   IP   |          | |        | |
| layers | | layers +--------+ | +--------+--------+ | +--------+ |
|        | |        |   L2   |-|-|   L2   |   L2   |-|-|   L2   | |
|        | |        +--------+ | +--------+--------+ | +--------+ |
|        | |        |   L1   |-|-|   L1   |   L1   |-|-|   L1   | |
+--------+ +-----------------+ | +-----------------+ | +--------+ |
                               |                     |            |
~~~~
{: #fig-hicn-prot-n9 title="Replacement of N9 interface - Protocol layers" }

__Anchorless consumer and producer mobility__

Option #2a realizes a fully anchorless solution as the traffic does not need to
go up to the anchor point in the core, nor to depend on the resolution performed
by a mapping node. As illustrated in {{fig-hicn-producer}}, communication
between two UEs can take place by forwarding traffic between their first UPFs,
and thus avoids unnecessarily overload of links up to the core. In this way
option #2a provides an effective traffic offloading and increased resiliency of
network operations in the event of a failure that would disconnect the edge from
the mobile core (e.g. disaster recovery scenario).

~~~~
    +---+---+      +---+---+  N3  +-------+
    |  UE   +------+  gNB  +------+  UPF  +------ ...
    +-------+      +-------+      +---+---+
                                      | N9    hICN domain
    +---+---+      +---+---+  N3  +-------+
    |  UE   +------+  gNB  +------+  UPF  +------ ...
    +-------+      +-------+      +---+---+
~~~~
{: #fig-hicn-producer title="Offloading of inter-UE communications" }

__Low state and signaling overhead__

Unlike traditional 3GPP GTP-based mobility management, hICN-AMM does not involve
signaling/ additional state to be maintained to handle static
consumers/producers or mobile consumers. Forwarding updates are required only in
the event of producer mobility to guarantee real-time re-direction of consumer
requests without triggering routing updates.  

__Dynamic forwarding strategies / UPF selection__

Dynamic selection of next hop or exit point is simplified by hICN-AMM as it can
be performed locally based on names and/or name-based locally available
information (e.g. measurements of congestion status or residual latency up to
the first data source).


## hICN deployment at both N9 and N3 interfaces (option #2b)

Option #2b proposes to additionally remove GTP tunnels between the RAN and the
first UPF (N3 interface). It is illustrated in {{fig-hicn-sba-n9n3}} and
{{fig-hicn-prot-n9n3}}.

~~~~
  +------------------+          +------------------+
  |        AMF       |          |        SMF       |
  +-+--------------+-+          +-+--------------+-+
    |              |              |              |
    | N1           | N2           | N4           | N4
    |              |              |              |
+---+---+      +---+---+  N3  +---+---+  N9  +---+---+  N6  +-------+
|  UE   +------+  gNB  +------+  UPF  +------+  UPF  +------+  DN   |
+-------+      +-------+  ^   +-------+  ^   +-------+      +-------+
                          |              |
                          |              |
                           hICN insertion
~~~~
{: #fig-hicn-sba-n9n3 title="Replacement of N3 and N9 interfaces" }

~~~~
    UE            5G-AN        N3         UPF        N9   UPF    N6
                               |                     |            |
+--------+                     |                     |            |
|  App.  |--------------------------------------------------------|
+--------+                     | +--------+--------+ | +--------+ |
| IP PDU |                     | | IP PDU | IP PDU | | | IP PDU | |
| (hICN) |-----------------------| (hICN) | (hICN) |-|-| (hICN) | |
+--------+ +-----------------+ | |        |        | | |        | |
|        | |\     decap     /  | |        |        | | |        | |
|        | | \_____________/   | |        |        | | |        | |
|        | |        |          | |        |        | | |        | |
|        | |        |          | |        |        | | |        | |
|   5G   | |   5G   |          | |        |        | | |        | |
|   AN   |-|   AN   |          | |        |        | | |        | |
|protocol| |protocol|          | |        |        | | |        | |
| layers | | layers +--------+ | +--------+--------+ | +--------+ |
|        | |        |   L2   |-|-|   L2   |   L2   |-|-|   L2   | |
|        | |        +--------+ | +--------+--------+ | +--------+ |
|        | |        |   L1   |-|-|   L1   |   L1   |-|-|   L1   | |
+--------+ +-----------------+ | +-----------------+ | +--------+ |
                               |                     |            |
~~~~
{: #fig-hicn-prot-n9n3 title="Replacement of N3 and N9 interfaces : Protocol
    layers" }

__Full tunnel removal; No internetworking between N3 and N9__

A clear advantage is the elimination of all GTP tunnels and a similar
specification of both N3 and N9 interfaces as recommended by 3GPP, so removing
challenges of inter-networking between N3 and N9. Also, it leads to a flat and
simpler architecture, allowing convergence with other non-3GPP accesses (and
related management and monitoring tools).

__State and signaling reduction__

A direct consequence is the extension to N3 of the property of hICN-AMM above
described, namely the absence of signaling and additional state to be maintained to
handle static consumers/producers or mobile consumers.

__Dynamic first UPF selection__

Dynamic forwarding capabilities are extended to the selection of the first UPF,
hence closer to the edge w.r.t. option #2a.

The advantage can be significant in the case of dense deployments scenarios:
here hICN-AMM makes possible to isolate the core network from local edge
mobility (a design objective of 5G mobile architecture), while still permitting
distributed  selection of ingress UPFs, and load-balancing of traffic across
them.

## Control plane considerations

By operating directly on router FIBs for mobility updates and for dynamic
policy-based forwarding strategies etc., hICN inherits the simplicity of IP
forwarding and can reuse legacy IP routing protocols to advertise/route ID
prefixes.  In this way it remains compatible with the exiting control plane
architecture as proposed in the 3GPP standard, with no change required to N1, N2
or N4.

hICN-AMM producer mobility management does not require SMF interaction, but
could be implemented by means of SMF signaling, at the condition to follow the
same procedure described for MAP-Me protocol in hICN-AMM. In the analysis of
pros and cons of SMF interaction, it is worth noticing that the absence of SMF
interaction might be beneficial in case of dense deployments or of failure of
the central control entities (infrastructure-less communication scenarios) to
empower distributed control of local mobility within an area.


# Deployment considerations

The benefits previously described can be obtained by an upgrade of only a few
selected routers at the network edge. The design of hICN allows the rest of the
infrastructure to remain unmodified, and to leverage existing management and
monitoring tools.

## hICN in a slice

The use of hICN does not impose any specific slicing of the network as the
architecture uses regular IPv6 packets, and is designed to coexist with regular
traffic. In that, hICN contrasts with previous approaches integrating ICN in IP
networks by using separate slices and/or different PDU formats as reviewed in
{{?I-D.irtf-icnrg-deployment-guidelines}}.

Although not required, slicing can ease a transition of services towards hICN,
and/or the coexistence of different mobility management solutions with hICN-AMM
or of different hICN deployment options.

An example use of slicing would be to create multiple slices for optimizing the
delivery of services having different requirements such as a low-latency
communication slice with hICN instances deployed very close to the edge, and a
video delivery service with caches deployed in locations aggregating a
sufficient number of users to be effective.

## End-to-end deployment

Deployment of the hICN stack in the endpoints is the preferred insertion option
and offers the full range of benefits. hICN client and network stacks are
available through two reference implementations based on the CICN project
{{CICN}}. They share the objective of smooth deployment in existing devices, and
are fully user-space based.

The first one is built on top of existing IP primitives and proposed as an
application/library for all major OS vendors including iOS, Android, Linux, macOS
and Windows. The second one targets high-performance routers and servers, and
leverages the VPP technology {{VPP}}.

## Network-contained deployment

In case it is not possible to insert hICN stack at endpoints, an hICN deployment
fully contained in the network can be envisaged through the deployment of
proxies. An overview of different options implemented at the network, transport
or application level is provided in {{I-D.irtf-icnrg-deployment-guidelines}}.
An example would be the deployment of HTTP (or RTP) proxies (as made available through
the CICN project)  at the ingress and egress (resp. first and last UPFs), in
order to benefit from content awareness in the network. Such configuration
however reduces the flexibility and dynamic forwarding capabilities in
endpoints. In particular, existing transport protocols have limited support for
dynamically changing paths/discovered sources or network conditions.

In such configuration, traffic that is not handled through hICN-AMM mechanisms
can still benefit from the lower overhead and anchorless mobility capabilities
coming from the removal of GTP tunnels, as well as dynamic forwarding
capabilities that are inherent to the forwarding pipeline. This results from the
ability to assign location-independent identifiers to endpoints even without the
deployment of a full hICN stack. The advantages of removed identifier/location
mapping and of a lightweight FIB update process are maintained. No encapsulation
is required and packet headers are not modified, which may allow the network to
have visibility of the source and/or destination identifiers.

## Alternative data planes: joint hICN/SRv6 deployment

hICN is designed to operate inside an IPv6 network by means of an enriched
communication layer supporting ICN primitives. The targeted deployment of a few
hICN-empowered nodes leads to the tradeoff between incremental deployment and
benefits which are proportionally related to the degree of hICN penetration. The
association of hICN with other data planes technologies is investigated as a
possibility to overcome the above-mentioned tradeoff yielding to a selective,
yet fully beneficial insertion of hICN in IP networks.

To this aim, we focus on hICN insertion in a Segment Routing (SR) enhanced data
plane, specifically considering SRv6 instantiation of SR
{{?I-D.ietf-spring-segment-routing}}.

hICN/SRv6 combination inherits all SRv6 advantages presented in SR-dedicated
section of this document, namely "underlay" management (fast reroute, etc.),
service chaining or fine-grained TE, for instance.

In addition, it allows extending the reach of hICN on regular IP routers with
SRv6 functionality. One realization being to create SRv6 domains in between hICN
nodes. The hICN router (through forwarding strategies) would then act as a
control plane for SRv6 by specifying the list of SIDs to insert in the packet.

SRv6 forwarding of packets between hICN hops would permit to enforce dynamic
per-application hICN forwarding strategies and their objectives (path steering,
QoS, etc.), which would be otherwise not possible over not hICN-enabled IP
network segments. It would also allow dynamic multi-path and load balancing in
hICN-unaware IP network segments and enforcing the symmetry of the request/reply
path for more accurate round trip delay measurements and rate/congestion
control).

<!--
Path symmetry
: in ICN and hICN, path symmetry is an essential property ensuring robust load
balancing and congestion control. The very nature of the underlying IP network
in between hICN nodes does not guarantee full path symmetry, but only partial
symmetry where intermediate hICN nodes define landmarks. SRv6 allow the use of a
"Record Route" like option able to reflect the path taken by the UL traffic
(path steering for instance) back into the returning packets. This will ensure
full path symmetry for the traffic in between hICN nodes. This might also be
used to support the reflective QoS specified by the 5G standard with proper
configuration.

Multipath/Multi-homing
: this feature exploits the path steering functionality offered by SRv6 to
transport traffic through different parts of the network, exploiting the full
path diversity and capacity offered by the transport network. This might also be
used to spread traffic over non-overlapping paths, or offer fast recovery
options in case of node or link failure.

Load balancing
: load balancing is a direct consequence of the previous option.

Anchorless mobility
: As previously discussed, anchorless mobility assumes the presence of
hICN-enabled routers at each hop, or at least the possibility by the control
plane to affect the FIBs of intermediate routers to reflect UE's mobility. If
this is not the case, SRv6 might be an alternative option allowing to steer the
traffic in between hICN nodes, thus bypassing possibly outdated IP forwarding
data in not-hICN IP routers.

Inter-domain forwarding:
-->


# Summary

hICN proposes a general purpose architecture that combines the benefits of a
pure-ID (name-based) architecture with those of ICN.

An hICN enabled network offers native offloading capabilities in virtue of the
anchorless properties resulting from the pure-ID communication scheme. It does
so without the need for a mapping system, and further requires no change in the
5G architecture nor in its control plane. The architecture will further leverage
the incremental insertion of Information-Centric functionalities through proxies
or direct insertion in user devices as the technology gets adopted and deployed.

While a full deployment is recommended to make efficient use of available
network resources, it is still possible to opt for a partial or phased
deployment, with the associated tradeoffs that we summarize in {{tab-summary}}.

This table reviews the various insertion options in the 3GPP architecture
earlier introduced -- namely options #1a (MEC), #1b (UPF), #2a (N9) and #2b
(N9/N3) --, assuming endpoints are hICN-enabled. Various levels of hICN
penetration are then considered : (UE) the UE is hICN-enabled, (proxy) hICN
processing is available through proxies, (None) no hICN function, the traffic is
purely forwarded using endpoint identifiers rather than content identifiers.

~~~~
                                       +-----------------------+------+
                     hICN penetration: |       UE      | Proxy | None |
  Benefit:          Deployment option: | 1a 1b | 2a 2b |   2b  |  2b  |
+--------------------------------------+-------+-------+-------+------+
| Additional state for static consumer | Y  Y  | Y  N  |   N   |  N   |
| Additional state for static producer | Y  Y  | Y  N  |   N   |  N   |
| Additional state for mobile consumer | Y  Y  | Y  N  |   N   |  N   |
| Additional state for mobile producer | Y  Y  | Y  N  |   N   |  N   |
+--------------------------------------+-------+-------+-------+------+
| Signalization for static consumer    | Y  Y  | N  N  |   N   |  N   |
| Signalization for static producer    | Y  Y  | N  N  |   N   |  N   |
| Signalization for mobile consumer    | Y  Y  | N  N  |   N   |  N   |
| Signalization for mobile producer    | Y  Y  | Y  Y  |   Y   |  Y   |
+--------------------------------------+-------+-------+-------+------+
| Removed tunnel/encap overhead        | N  N  | P  Y  |   Y   |  Y   |
| Preserve perf. of flows in progress  | N  N  | Y  Y  |   Y   |  Y   |
+--------------------------------------+-------+-------+-------+------+
| Local offload                        | P  N  | Y  Y  |   Y   |  Y   |
| Anchorless for UP                    | N  N  | Y  Y  |   Y   |  Y   |
| Anchorless for CP                    | N  N  | Y  Y  |   Y   |  Y   |
| Dynamic egress UPF selection         | N  N  | Y  Y  |   Y   |  Y   |
| Dynamic ingress UPF selection        | N  N  | N  Y  |   Y   |  Y   |
+--------------------------------------+-------+-------+-------+------+
| Edge caching : low-latency           | Y  Y  | Y  Y  |   P   |  N   |
| Edge caching : multicast             | Y  Y  | Y  Y  |   P   |  N   |
| Seamless mobility across het. RAT    | Y  Y  | Y  Y  |   Y   |  P   |
| Multi-homing ; Bandwidth aggregation | Y  Y  | Y  Y  |   Y   |  P   |
| Multi-source                         | Y  Y  | Y  Y  |   Y   |  N   |
| In-network assistance                | N  Y  | Y  Y  |   P   |  N   |
+--------------------------------------+-------+-------+-------+------+
| Integration with 3GPP CP             | N  Y  | Y  Y  |   Y   |  Y   |
+--------------------------------------+-------+-------+-------+------+

LEGEND: (Y) Yes - (N) No - (P) Partial
~~~~
{: #tab-summary title="Tradeoff in benefits for various deployment options }

[modeline]: # ( vim: set cc=72: )
