---
title: "Anchorless mobility through hICN"
docname: draft-auge-dmm-hicn-mobility-04
date: 2020-07-08
category: info

ipr: trust200902
area: general
workgroup: DMM Working Group
keyword: Internet-Draft
keyword: Information-Centric Networking
keyword: Hybrid ICN

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:

-
    ins: J. Auge
    name: Jordan Auge
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
  MAPME: DOI.10.1109/TNSM.2018.2796720
  SURVEY1: DOI.10.1109/INFCOMW.2016.7562050
  SURVEY2: DOI.10.1186/s13638-016-0715-0
  SURVEYICN: DOI.10.1145/2248361.2248363
  DATAPLANE:
    title: "Ensuring connectivity via data plane mechanisms."
    author:
      -
        surname: "Liu"
        fullname: "Junda Liu"
        ins: "J."
      -
        surname: "Panda"
        fullname: "Aurojit Panda"
        ins: "A."
      -
        surname: "Singla"
        fullname: "Ankit Singla"
        ins: "A."
      -
        surname: "Godfrey"
        fullname: "Brighten Godfrey"
        ins: "B."
      -
        surname: "Schapira"
        fullname: "Michael Schapira"
        ins: "M."
      -
        surname: "Shenker"
        fullname: "Scott Shenker"
        ins: "S."
    date: 2013
  SEC: DOI.10.1145/3125719.3125725
  TS29.274:
    title: 3GPP Evolved Packet System (EPS); Evolved General Packet Radio Service (GPRS) Tunnelling Protocol for Control plane (GTPv2-C); Stage 3
    author:
      - surname: 3gpp-29.274
    series:
      name: 3GPP
      value: ""

--- abstract

This document first discusses the use of locators and identifiers in mobility
management architectures, and their implication on various anchorless
properties. A new architecture is then proposed that is purely based on
identifiers, and more specifically names as defined in Hybrid-ICN (hICN). The
document then focuses on two main cases: the end-point sends data (data
producer) or the end-point receives data (data consumer). These two cases are
taken into account entirely to provide anchorless mobility management in hICN.

--- middle

# Introduction

New usages of the network and the rapid growth of the Mobile Internet calls to
reconsider the way we deploy and operate IP networks, where mobility is not
built into the design, but rather added as an afterthought. Notable examples are
IETF Mobile IP and its variants {{!RFC5944}} and {{!RFC2275}} 3GPP GTP-based
architecture {{TS29.274}}, both based on tunnelling and encapsulation.

One identified difficulty in proposing mobility models for IP lies in the
semantic overloading of IP addresses which are both host identifiers, and
locators used for routing. Starting with LISP, the identifier/locator split
paradigm has shown promising results in virtue of its scalability properties with respect
to routing tables entries, and the possibilities it offers in terms of mobility.
Several solutions have been proposed around this concept, namely ILNP, ILSR, and
ILA. One common facet of these proposals is to embrace the current trend of a
clear separation between the control and data planes, which both allows for
distribution of the control infrastructure, as well as anchorless operations in
the data plane, including facilitated local breakout.

The counterpart of these architectures is an increased dependency on the control
plane which is responsible for binding the identifiers used by the application
at the edge to the locators used to forward the traffic in the network. Device
mobility will typically induce a change of IP address, which makes performance
of flows in progress dependent on interactions with those control elements which
have to remain globally consistent.

In this document, we first propose to clarify protocol descriptions by adopting
a new terminology of control-plane and data-plane anchors to characterize
anchorless operations, and show their tight coupling with the use of locators
and identifiers. This definition serves to position Loc/ID-split architectures
with respect to the traditional use of tunnels, before advocating to push this
step further and perform mobility management purely based on identifiers. We
introduce a mobility approach based on Hybrid ICN (hICN) as described in
{{?I-D.muscariello-intarea-hicn}}, for which we perform an in-depth analysis of
mobility considerations. We show how this proposal can help addressing further
challenges faced by networks today, such as multihoming and multipath, while
preserving the simplicity and end-to-end design of the current Internet.

# Locators, identifiers and anchorless mobility management

## Terminology

The consideration of mobility in network design shares the challenges raised for
routing for instance in {{!RFC6115}}, where the terminology for locators and
identifiers is presented.

Because they are intrinsically bound to the topological location of a node,
session established through locators require additional mechanisms to support
mobility, including the presence of anchor points in the network. The notion of
*anchor* and the resulting *anchorless* properties of mobility schemes are prone
to different interpretation depending on the context. The following definitions
attempt to reconcile those interpretations and propose a unifying terminology
used throughout this document.

Anchor
: An anchor refers to a specialized node or service, possibly distributed,
functionally required by a network architecture for forwarding or mobility. This
is in contrast to decentralized architectures. Configuration of these
anchors, including their location, will directly affect the overall scheme
performance. We follow the classic distinction between control plane and user
(or data, or forwarding) plane to distinguish control-plane anchors and
user-plane anchors.

User-plane anchor
: A user-plane anchor is a node through which traffic is forced to pass.
An example of such anchor is the indirection point in Mobile IP.

Control-plane anchor
: A control-plane anchor refers to a node that is not responsible for carrying
traffic, but is needed for the operation of the forwarding and/or the
mobility architecture. An example of such anchor is the resolution or mapping
service of LISP. We remark that while not being on path, such anchors might
affect the performance of the user-plane due to resolution delays or indirection
(following a mapping cache miss for instance).

Anchorless
: This term qualifies approaches that do not involve any user-plane not
control-plane anchor. The challenge for such full anchorless approaches are
exacerbated during mobilty of both end points at the same time, as they cannot
rely on any stable anchor point to preserve connectivity. Nor they can rely on
routing mechanism that would be the source of overhead and instability.

## Towards locator-independent network architectures

We distinguish network architectures based on their use of location independent
identifiers (or names) and locators for forwarding.

__Locator-based architectures__

As mentioned earlier, IP architectures are typically operated based on locators
corresponding to the IP addresses of the host interfaces in their respective
network attachment, used also as session identifiers. This results in a complex
mobility architecture built on top, involving traffic anchors and tunnels to
preserve the identifier exposed to the transport layer: IP/IP or GRE tunnels in
Mobile IP, and GTP tunnels in 3GPP architectures.

The limitations of locator-based schemes in terms of complexity, overhead and
efficiency are well-recognized and led to other alternatives to be considered.

__Locator-ID separation architectures__

LISP {{!RFC6830}} was the first proposal to distinguish between the usage of IP
addresses as locators or identifiers by explicitly defining two namespaces,
respectively used for endpoint identification and forwarding. A mapping service
is further used to bind an identifiers to a given location, and updated after
mobility. From there, several approaches have been defined, either host-based
like SHIM6 {{!RFC5533}} or HIP {{!RFC4423}}), or network-based, like LISP {{!RFC6830}},
ILSR, ILNP {{!RFC6740}} or ILA {{?I-D.herbert-intarea-ila}} to cite a few.

An overview of these approaches and their use in mobility is presented in
{?I-D.bogineni-dmm-optimized-mobile-user-plane}}.

The main challenge consists in maintaining a (distributed) mapping service at
scale, including the synchronization of local caches required for scalability
and efficiency.

__ID-based architectures__

A third class of approaches exists that redefines IP communication principles
(i.e. network and transport layers) around location-independent network
identifiers of node/traffic. The interest of such architectures is highlighted
in {{?I-D.vonhugo-5gangip-ip-issues}} (referred to as ID-oriented Networking)
for it removes the limitations introduced by locators and simplifies the
management of mobility. The draft however question the possibility to realize
such an architecture where node status and mobility would not affect routing
table stability.

The work done around Information-Centric Networking (ICN) falls into such class
of approaches that we refer to as purely ID-based, also known as name-based
{{?I-D.irtf-icnrg-terminology}}, although as we will see, mobility management
often departs from this principle.


## Information-Centric Networking (ICN)

ICN is a new networking paradigm centering network communication around
named data, rather that host location. Network operations are driven by
location-independent data names, rather than location identifiers (IP
addresses) to gracefully enable user-to-content communication.

Although there exist a few proposals, they share the same set of core
principles, resulting in several advantages including a simplified mobility
management {{!RFC7476}}. For clarify, this section we focus
on hICN {{?I-D.muscariello-intarea-hicn}} an ICN implementation for IPv6.

## Hybrid-ICN overview

Hybrid ICN (hICN) is an ICN architecture that defines integration of ICN
semantics within IPv6.
The goal of hICN is to ease ICN insertion in existing IP infrastructure by:

1. selective insertion of hICN capabilities in a few network nodes at the edge
(no need for pervasive fully hICN network deployments);
2. guaranteed transparent interconnection with hICN-unaware IPv6 nodes, without
using overlays;
3. minor modification to existing IP routers/endpoints;
4. re-use of existing IP control plane (e.g. for routing of IP prefixes carrying
ID-semantics) along with performing mobility management and caching operations
in forwarding plane;
5. fallback capability to tradition IP network/transport layer.

hICN architecture is described in {{?I-D.muscariello-intarea-hicn}}.

# Hybrid-ICN Anchorless Mobility Management (hICN-AMM)

hICN, together with MAP-Me {{?I-D.irtf-icnrg-mapme}}, forms the basis for the mobility management
architecture we describe in the rest of this document. Due to the pull based
nature of hICN architecture, we distinguish consumer and producer nodes, for
which mobility is handled differently

## Consumer mobility in hICN

The consumer end-point is the logical communication termination that receives data.
Due to the pull-based and connection-less properties of hICN
communications, consumer mobility comes natively with ICN. It is indeed
sufficient that the consumer reissues pending interests from the new
point-of-attachment to continue the communication. Consumer mobility is
anchorless by design, and managed without any impact on the transport session.
It is however necessary to have an appropriate transport layer on top able to
cope with eventual disruptions and path variations caused by the mobility event.

# Producer mobility architectures

The producer end-point is the logical communication termination that sends data.
Producer mobility is not natively supported by the architecture, rather handled
in different ways according to the selected producer mobility management scheme,
some of which diverge from the concept of pure ID-based architecture through
their use of locators. Additional procedures have to be performed to maintain
reachability as it moves in the network.

In fact, many schemes proposed for ICN are adaptations to the vast amount of
work made in IP over the last two decades {{!RFC6301}}. Surveys for the ICN
family, resp. for CCN/NDN-specific solutions, are available in {{SURVEYICN}},
respectively {{SURVEY1}} and {{SURVEY2}}. There has been however a recent trend towards
anchorless mobility management, facilitated by ICN design principles, that has
led to new proposals and an extension of previous classifications in
{{?I-D.irtf-icnrg-mapme}} and {{MAPME}} to the four following categories:

- _Resolution based_ solutions rely on dedicated rendez-vous nodes (similar to
DNS) which map content names into routable location identifiers. To maintain
this mapping updated, the producer signals every movement to the mapping system.
Once the resolution is performed, packets can be correctly routed directly to
the producer.
- _Anchor-based_ proposals are inspired by Mobile IP, and maintain a mapping at
network-layer by using a stable home address advertised by a rendez-vous node,
or anchor. This acts as a relay, forwarding through tunneling both interests
to the producer, and data packets coming back.
- _Tracing-based_ solutions allow the mobile node to create a hop-by-hop
forwarding reverse path from its RV back to itself by propagating and keeping
alive traces stored by all involved routers. Forwarding to the new location is
enabled without tunneling.
- _Anchorless_ approaches allow the mobile nodes to advertise their mobility to
the network without requiring any specific node to act as a rendez-vous point.

## Producer mobility in hICN

In an hICN network, regular routing protocols such as BGP, ISIS or OSPF can be
used for propagating all prefix announcements and populate routers' FIBs.
However, these protocols are not appropriate and should not be used to manage
name prefix mobility, for scalability and consistency reasons.

The default mobility management for hICN is designed following the same
principles and protocols as MAP-Me, an anchorless producer mobility management
protocol initially proposed for ICN {{?I-D.irtf-icnrg-mapme}} {{MAPME}}. It
builds on an initial forwarding state bootstrapped by the routing protocol, and
performs a lightweight path repair process as the producer moves. For
simplicity, we refer to it as simply MAP-Me in the rest of the document.

In the rest of this section, we describe the specific realization of the
protocol in an hICN context. Additional background and details are available in
{{?I-D.irtf-icnrg-mapme}} and {{MAPME}}. The solution is based on a path repair
mechanism following mobility events, using dynamic FIB updates. Using data plane
mechanisms for ensuring connectivity has been previously proposed in
{{DATAPLANE}} to handle link failures, and has been proven more reactive than
relying on typical control plane messaging.

# Protocol description

## Signalization messages; acknowledgements and retransmission

Signalization messages follow hICN design principles and use data plane packets
for signalization. Signalization messages and acknowledgement are respectively
Interest and Data packet (requests and replies) according to hICN terminology
{{?I-D.muscariello-intarea-hicn}}.
Upon processing of those advertisements, the network will send an
acknowledgement back to the producer using the name prefix as the source and
the locator of the producer as the destination, plus the sequence number
allowing the producer to match which update has been acknowledged.

Pending signalization interests that are not acknowledged are retransmitted
after a given timeout.

## Dynamic face creation and producer-triggered advertisements

The producer is responsible for mobility updates and should be hICN-enabled. It
stores a sequence number incremented at each mobility event.

Faces in the producer are assumed synchronized with layer 2 adjacencies, upon
joining a new point-of-attachment, a new face should be created. Face creation
will trigger the increase of the sequence number, and per-prefix advertisement
packets to be sent to the joined network.

Those advertisements should contain:

- the name prefix as the destination address, plus a field indicating the
associated prefix length;
- the locator of the producer as the source address, that will serve for
receiving acknowledgements;
- a sequence number sequentially increased by the producer after each movement;
- a security token (see {{sec-security}}).

Upon producer departures from a PoA, the corresponding face is destroyed. If
this leads to the removal of the last next hop, then faces that were previously
saved are restored in FIB to preserve the original forwarding tree and thus
global connectivity.

## Update protocol

Based on the information transmitted in the packet, and its local the network's
local policy, the network might decide to update its forwarding state to reflect
this change.

The update process consists in updating a few routers on the path between the
new and a former point-of-attachment. More precisely, this is done in a purely
anchorless fashion by sending a signalling message from the new location towards
the name prefix itself. This packet will be forwarded based on the now-stale
FIB entries, and will update forwarding entries to point to the ingress
interfaces of each traversed router.

### Illustration

{fig-mapme-update}} illustrates the situation of a P moving between access
routers AR1 and AR2 while serving user requests:

1. A first interest towards the producer originates from remote. The producer
   answers with a Data packet.
2. Following this, the producer detaches from AR1 and moves to AR2.
3. As soon as it is attached to AR2, the producer sends an Interest Update
   towards its own prefix, which is forwarded from AR2 following the FIB towards
   AR1 (one of its former positions in the general case).
4. At each hop, the message fixes the FIB to point towards the ingress face,
   until the message cannot be further forwarded in AR1.
5. A subsequent message from remote will follow the updated FIB and correctly
   reach the producer's new location in AR2.

~~~~
    P (radio link) AR1             AR2             GW        Internet
    |               |               |               |              |
1   |               |               |               |<~~~~~~~~~~~~~|
    |               |<~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~|  Interest    |
    |<~~~~~~~~~~~~~~|               |               |              |
    |-------------->|               |               |              |
2  []    Data       |---------------|-------------->|              |
  / |               |               |               |------------->|
  | |(attach to AR2)|               |               |              |
  -X]...............|...............|               |              |
3   |===============|==============>o FIB update    |              |
    | Update        |               |==============>o FIB update   |
4   |               o<==============|===============|              |
    |    FIB update |               |               |              |
5   |               |               |               |<~~~~~~~~~~~~~|
    |               |               |<~~~~~~~~~~~~~~|              |
    |<~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~|               |              |
    |---------------|-------------->|               |              |
    |               |               |-------------->|              |
    |               |               |               |------------->|
    |               |               |               |              |
~~~~
{: #fig-mapme-update title="Signalization during producer mobility (MAP-Me)"}

Through this process, MAP-Me trades-off path optimality for a fast, lightweight
and loop-free forwarding update. It has been shown in {{MAPME}} that stretch is
bounded and typically kept low in scenarios of interest. This results from the fact that MAP-Me
operations preserve the initial structure of the forwarding tree/DAG (direct acyclic graph), by only
flipping its edges toward the new producer location.

MAP-Me does not perform a routing update. The protocol is complementary to the
routing protocol in that it can run in between routing updates to handle
producer mobility, at a faster timescale, and with a lower overhead. Routing
updates can be performed at a lower-pace, to re-optimize routes once a node has
relocated for instance.

### Message content

The update messages should contain :

- the name prefix as the destination address, plus a field indicating the
associated prefix length;
- the locator of the producer as the source address, that will serve for
receiving acknowledgements;
- a sequence number sequentially increased by the producer after each movement;
- an optional security token.

### Processing at network routers

At the reception of advertisement/update packets, each router performs a
name-based Longest Prefix Match lookup in FIB to compare sequence number from
the received packet and from FIB}. According to that comparison:

- if the packet carries a higher sequence number, the existing
  next hops associated to the lower sequence number in FIB are used to
  forward it further and temporarily stored to avoid loss of such information before completion of the
  acknowledgement process.

- If the packet carries the same sequence number as in the FIB, its originating
  face is added to the existing ones in FIB without additional packet processing
  or propagation. This may occur in presence of multiple forwarding paths.

- If the packet carries a lower sequence number than the one in the FIB, FIB
  entry is not updated as it already stores 'fresher information'.  To advertise
  the latest update through the path followed by the packet, this one is re-sent
  through the originating face after having updated its sequence number with the
  value stored in FIB.

## Notifications and scoped discovery

The update protocol is responsible for reestablishing global connectivity with
minimal changes to the FIBs. In order to further improve the reactivity of the
scheme and better support QoS constraints of latency-sensitive traffic, we
propose an additional mechanism named __Notifications__. It assumes hICN-enabled
routers at the edge, and the existence of links between access routers, which
are typically used for handover, and proposes to exploit them during mobility
events, or to delay updates when possible.

### Notification processing

Upon receiving a valid advertisement, the point-of-attachment will remember the
presence of the producer, update its corresponding FIB entry but send no update.
Previous next hops should be saved and restored upon face deletion so as to
preserve the forwarding tree/DAG structure.

The rationale is that during mobility events, the producer will move access
connected and neighbouring base stations. It will be then sufficient to make a
scoped discovery around the last position known to forwarding to find the
producer within a few hops.

### Illustration

{{fig-mapme-discovery}} illustrate such as situation where an Interest is sent
to the producer before it had the time to complete the update.

~~~~
    P (radio link) AR1             AR2             GW        Internet
    |               |               |               |              |
1   |               |               |               |<~~~~~~~~~~~~~|
    |               |<~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~|   Interest   |
    |<~~~~~~~~~~~~~~|               |               |              |
    |-------------->|               |               |              |
2  [].....Data.....-)---------------|-------------->|              |
  / |               |               |               |------------->|
  | |(attach to AR2)|               |               |              |
  -Y]...............|...............+               |              |
3   |               |               |               |<~~~~~~~~~~~~~|
4   |            X--|<~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~|              |
    |               |               |               |              |
5   |===============|==============>o FIB update    |              |
    | Notification  |               |               |              |
    |               |~~~~~~~~~~~~~~>| (scoped discovery)           |
6   |<~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~|               |              |
7   |---------------|-------------->|               |              |
    |               |<--------------|               |              |
    |               |---------------|-------------->|              |
    |               |               |               |------------->|
    |               |               |               |              |
~~~~
{: #fig-mapme-discovery title="Scoped discovery during producer mobility" }

1. A first interest towards the producer originates from remote. The producer
   answers with a Data packet.
2. Following this, the producer detaches from AR1 and moves to AR2.
3. A subsequent message from remote arrives before the update could take place,
   and is thus forwarded to AR1.
4. AR1 has no valid face anymore towards the producer, and enters discovery mode
   by broadcasting the interest to neighbours.
5. In the meantime, we assume the notification reaches AR2.
6. Because the producer is attached to AR2, the producer can be directly
   reachable. Otherwise, AR2 would iterate the discovery to neighbours only if
   it had more recent information about the producer location than its
   predecessor (based on sequencing of regular Interests and Interest Updates).
7. The Data packet follows the reverse path to the consumer as usual.

<!--
## Algorithm

The operations in the forwarding pipeline for IU/IN processing are reported in
Alg. {{alg-update}}.

~~~~~~~~~~
   | Algorithm 1 : ForwardSpecialInterest(SpecialInterest SI, Ingress face F)
   |
   |  CheckValidity()
   |  // Retrieve the FIB entry associated to the prefix
   |  e, T <- FIB.LongestPrefixMatch(SI.name)
   |  if SI.seq >= e.seq then
   |  .   // Acknowledge reception
   |  .   s <- e.seq
   |  .   e.seq <- SI.seq
   |  .   SendReliably(F, SI.type + Ack, e)
   |  .   //Process special interest
   |  .   if F in e.TFIB then
   |  .   .   // Remove outdated TFIB entry (eventually cancelling timer)
   |  .   .   e.TFIB = e.TFIB \ F
   |  .   if SI.seq > s then
   |  .   .   if SI.type == IU then
   |  .   .   .   // Forward the IU following the FIB entry
   |  .   .   .   SendReliably(e.NextHops, SI.type, e
   |  .   .   else
   |  .   .   .   // Create breadcrumb and preserve forwarding structure
   |  .   .   .   e.TFIB = e.TFIB U {( f -> NULL) : for all f in e.NextHops}
   |  .   .   e.NextHops = {}
   |  .   e.NextHops = e.NextHops U { F }
   |  else
   |  .   // Send updated IU backwards
   |  .   SI.seq = e.seq
   |  .   SendReliably(F, SI.type, e)
~~~~~~~~~~
{: #alg-update}
-->


<!--
More details about the realization of the protocol can be found in
{{?I-D.irtf-icnrg-mapme}}, including optimizations allowing to improve the
support of latency-sensitive applications, and further reducing overhead.

+ sequence numbers
-->

# Benefits

We now review the potential benefits of the general architecture we have
presented using features from the hICN data plane for supporting consumer and
producer mobility.

## Overview

__Native mobility__ : This mobility management process follows and exploits
hICN design principles. It makes producer mobility native in the
architecture by preserving all benefits of hICN even when consumers and/or
producers are moving.

__Anchorless mobility__ : Such approach belongs to the category of pure ID-based
mobility management schemes whose objective is (i) to overcome the limitations
of traditional locator-based solutions like Mobile IP (conf)using locators as
identifiers and requiring tunnels, and (ii) to remove the need for a global
mapping system as the one required by locator-identifier separation solutions.
The result is a fully anchorless solution both in the data plane and in the
control plane.

__Local and decentralized mobility management__ : Mobility updates are handled
locally and the routers that are affected are those on path between successive
positions of the producer. In particular, remote endpoints are not affected by
the event. Mobility is managed in a fully distributed manner and no third party
is required. This does not prevent any centralized control (as discussed in
{{?I-D.auge-dmm-hicn-mobility-deployment-options}}), but makes the network
robust to disconnectivity events.

The next section will discuss more in depth the following advantages

## Simplicity, scalability, efficiency

As emerges from the points raised in the previous section, consumer mobility is
transparently supported by an hICN network in virtue of the pull-based model and the
way the forwarding path works.
After moving, a consumer can just reissue pending interests once attached to the
new access router at layer 2, without requiring any more information from L3 and
above. This ensures a fast and simple handover, which can be further enhanced
through additional mechanisms such as in-network caching. Consumer mobility is
fully anchorless with hICN, and does not incur any signalization nor tunneling
overhead.

This is particularly interesting considering that most mobile users are
consumers only (e.g. linear video distribution, or large scale video conferencing
where we typically have few presenters and most users are simply consumers).

Another aspect of using the unifying hICN architecture in replacement of the
traditional tunnel-based mobile core is that it removes the need to maintain
state for consumers and producers which are not mobile (eg. IoT sensors), or not
currently moving.


## Reduced latency through caching

While this is not strictly linked to mobility, we first illustrate the benefits
of caching content close to the edge to reduce user latencies. This is
particularly important for wireless networks such as WiFi or LTE which have
non-negligible latencies especially during connection setup, or after an idle
period. The characteristics of those radio networks are thus of interest for the
performance of the mobility architecture as a whole.

The example in {{fig-twoaccess}} considers a mobile node that can move access
two accesses linked to Access Routers (AR) AR1 and AR2, both connected to the
Internet though a common gateway (GW). This same setup will be later used to
illustrate the flow of packets during mobility events, eventually specializing AR
into AP or eNB when it makes more sense.

~~~~
             +-----+                      .--.
         _,--+ AR1 +--,            .-~ ~-(    )_ _
+-----+ /    +-----+   \ +----+   |                ~-.        +-----+
|  C  +=                =+ GW +---+     Internet       \--//--+  P  |
+-----+ \_   +-----+   / +----+    \                  .'      +-----+
          '--+ AR2 +--'              ~-.__________.-~
             +-----+
~~~~
{: #fig-twoaccess title="Simple topology with a multi-homed consumer"}

We represent in {{fig-latency}} a simple data flow between the different entities involved in the
communication between the mobile node M, and a remote producer available over
the Internet.


~~~~
    C              AR1             AR2             GW        Internet
    |               |               |               |              |
1   |~~~~~~~~~~~~~~>|               |               |              |
    |  Interest X   |~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~>|              |
    |               |               |               |~~~~~~~~~~~~~>|
    |               |               |               |             ...
    |               |               |               |<-------------|
    |     Data X    |<--------------|---------------|              |
    |<--------------|               |               |              |
    |               |               |               |              |
2   |~~~~~~~~~~~~~~>|               |               |              |
3   |  Interest Y   |~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~>X Cache hit    |
    |               |<--------------|---------------X              |
    |<--------------|               |               |              |
    |    Data Y     |               |               |              |


LEGEND:
~~~~>: Interest     <---- Data     ====> Signalization
....-: L2 detach    .....+ L2 attach         X failure    o event

~~~~
{: #fig-latency title="Reduced latency through caching" }

Numbers on the left refer to the following comments relative to the data flow:

1. The consumer issues a first interest towards name prefix X, which is transported
   up to the producer. A Data packet comes back following the reverse path and
   populating intermediate caches. Latency of the exchange can be seen by the
   distance between lines on the vertical axis.

2. The consumer now requests content B, which has previously been requested by
   another consumer located on the same gateway. Interest B thus hits the cache,
   refreshes the entry, and allows a lower round-trip latency to content both for
   consumer C and any subsequent request of the same content.


## Improved reliability through caching

Mobile networks might consist in unreliable access technologies, such as WiFi,
responsible for packet losses. {{fig-loss-cache}} considers a similar scenario
with a lossy channel (eg. WiFi) between the mobile and the first access router.

~~~~
    C (radio link) AR1             AR2             GW        Internet
    |               |               |               |              |
1   |~~~~~~~X       |               |               |              |
2   |~~~~~~~~~~~~~~>|               |               |              |
    |  Interest     |~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~>|              |
    |               |               |               |~~~~~~~~~~~~~>|
    |               |               |               |             ...
    |               |               |               |<-------------|
    |     Data      |<--------------|---------------|              |
3   |       X-------|               |               |              |
    |               |               |               |              |
4   |~~~~~~~~~~~~~~>X Cache hit     |               |              |
    |<--------------X               |               |              |
    |               |               |               |              |
~~~~
{: #fig-loss-cache title="IU propagation example"}

1. The first issued interest is lost due to bad radio conditions.

2. Upon detection (either after a timeout, the consumer can just reissue the lost
   Interest.

3. This time the remote producer successfully replies but the Data packet is
   lost on the radio link.

4. Upon a similar detection, the consumer can reissue the lost Interest that
   will hit a locally stored copy at the router where the loss occurred (or again
   use a detection mechanism to retransmit the Data packet).

## Local mobility and recovery from common cache

The same mechanism can be used to recover from mobility losses after the
consumer reconnects to the new network as the content will already be available
in the cache at the junction router between the two accesses. This is
particularly interesting in case of micro-mobility between access routers that
are topologically close in the network.

This is also the opportunity to manage those losses on behalf of the transport
protocol, so that only losses due to congestion are exposed to it, and don't
perturb the feedback loop by unnecessarily reducing the transfer rate.

~~~~
    C (radio link) AR1             AR2             GW        Internet
    |               |               |               |              |
1   |~~~~~~~~~~~~~~>|               |               |              |
    |  Interest     |~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~>|              |
    |               |               |               |~~~~~~~~~~~~~>|
    |(detach fm AR1)|               |               |              |
2 ..|...............-               |               |<-------------|
    |               |<--------------|---------------|              |
3   |        Data X-|               |               |              |
    |               |               |               |              |
    |(attach to AR2 |               |               |              |
4 ..|...............|...............+               |              |
5   |~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~>|               |              |
6   |               |               |~~~~~~~~~~~~~~>X Cache hit    |
    |               |               |<--------------X              |
    |---------------|---------------|               |              |
    |               |               |               |              |
~~~~
{: #fig-mobility-cache title="IU propagation example"}

1. The consumer issues an interest before moving to a new location
2. It detaches from the first Access Router AR1
3. This causes the returning data packet to be lost as there is no more a valid
   face towards the consumer.
4. The consumer has finished attachment to the new access router AR2.
5. It can retransmit pending interests immediately.
6. The interests will hit the cache at the first junction point between AR1 and
   AR2 which have previously seen the data packet coming back.

## Additional reliability through consumer multihoming

Because mobility is implemented at layer 3 and is thus agnostic to the physical
layer, this allows a mobile consumer to seamlessly switch to a different access
layer, eg. WiFi to LTE, following the unreachability of the preferred radio
access.

{{fig-mobility-seamless}} illustrates a fallback to LTE which can be performed
transparently for the application.

~~~~
  C          WiFi          GW         LTE enB        PGW     Internet
  |            |            |            |            |            |
1 |~~~~~~~~~~~>|            |            |            |            |
  |            |~~~~~~~~~~~>|            |            |            |
  |            |            |~~~~~~~~~~~~|~~~~~~~~~~~~|~~~~~~~~~~~>|
  |            |            |            |            |            |
  |            |            |<-----------|------------|------------|
  |            |<-----------|            |            |            |
  |<-----------|            |            |            |            |
2 |            X            |            |            |            |
3 |~~~~~~~~~~~~X~~~~~~~~~~~~|~~~~~~~~~~~>|            |            |
  |            X            |            |~~~~~~~~~~~>|            |
  |            X            |            |            |~~~~~~~~~~~>|
  |            |            |            |            |            |
  |            |            |            |            |<-----------|
  |            |            |            |<-----------|            |
  |<-----------|------------|------------|            |            |
4 |~~~~~~~~~~~>|...         |            |            |            |
  |            |            |            |            |            |
~~~~~
{: #fig-mobility-seamless title="Seamless mobility between a WiFi and a LTE access point}

1. First interest is sent on the WiFi link.
2. WiFi unavailability due to failure for instance.
3. The application seamlessly switches to the LTE link.
4. Upon recovery, the traffic can be brought back on the WiFi interface.

hICN handles consumer mobility from one access to the other (e.g.  WiFi to LTE
or vice-versa) without any a-priori knowledge of the multiple networks to use as
it is the case of MPTCP or QUIC approaches. Moving rate and congestion control
at the receiver end results to be a significant advantage w.r.t. all existing
alternatives in controlling dynamically multiple and new discovered network
accesses in presence of mobility.

This use case highlights the importance of having a compatible transport, as the
WiFi and LTE paths will have much different characteristics in terms of delay,
jitter and capacity.

Evaluations of the scheme have shown this scheme to preserve the performance of
flows like mobile video distribution up to very high switching rates in the
order of a second, not even considering optimization occurring from network
support.

Mobility is handled transparently at the network layer with very
fast handover times, due to the connection-less property of hICN. This makes it
possible to offer reliable WiFi connectivity, besides its lossy nature as
observed in previous use case and besides the frequency of mobility from one
network access to the other.


## Bandwidth aggregation with consumer multihoming

Bandwidth aggregation can be realized dynamically through a congestion-aware
load-balancing forwarding strategy at the client, with no a priori knowledge of
paths. This is done similarly in the network by hICN forwarders, allowing a
combination of multi-homing, multipath and multi-source data transfers. As all
the paths are used at the same time, hICN offers the full network capacity to the
users and tends to smooth fluctuations due to the radio channels.

Over an heterogeneous network access, hICN also offers a simple and
cost-effective realization of heterogeneous channel bonding allowing an user to
seamlessly roam across different radios or fixed lines (for increased
reliability or reduced costs), or aggregate their bandwidth for high-throughput
applications such as video streaming.

We illustrate a simplified data flows with one mobile consumer C alternating
interests between a WiFi and LTE access points. The load balancing strategy
would in that case optimize the split ratio between the two access to realize an
optimal split. Note that in that case the relative distances on the vertical
axis have not been respected here for readability.

~~~~
  C          WiFi          GW         LTE enB        PGW     Internet
  |            |            |            |            |            |
  |~~~~~~~~~~~~|~~~~~~~~~~~~|~~~~~~~~~~~>|            |            |
  |~~~~~~~~~~~>|            |            |~~~~~~~~~~~>|~~~~~~~~~~~>|
  |            |~~~~~~~~~~~>|            |            |            |
  |            |            |~~~~~~~~~~~~|~~~~~~~~~~~~|~~~~~~~~~~~>|
  |            |            |            |            |            |
  |            |            |            |            |<-----------|
  |            |            |            |<-----------|            |
  |<-----------|------------|------------|            |            |
  |            |            |<-----------|------------|------------|
  |            |<-----------|            |            |            |
  |<-----------|            |            |            |            |
  |            |            |            |            |            |
  |            |            |            |            |            |
~~~~

Fine grained control from the application allows fully exploiting available
bandwidth, resulting in an aggregated throughput equal to the sum of access
throughputs, which is hard to achieve with existing solutions.


## Traffic and signalization offload

As a natural consequence of its anchorless behavior, an hICN network can
continue to operate in disconnected mode, for local mobility, even though no
upstream entity is available.

In order to illustrate this, we slightly extend the previous topology in
{{fig-disconnected-topology}} with an additional access network. To show that
local mobility induces no traffic on upstream links, we further assume a failure
on the link between the gateway (GW) and the Internet.

~~~~
             +-----+
         _,--+ AR1 +--,
+-----+ /    +-----+   \                         .-~~~-.
|  P  +=                \                .- ~ ~-(       )_ _
+-----+ \_   +-----+     +----+         |                     ~ -.
          '--+ AR2 +-----+ GW +----X----+        Internet         \
             +-----+     +----+   FAIL   \                       .'
                        /                 ~- . _____________ . -~
+-----+      +-----+  /
|  C  +------+ AR3 +-'
+-----+      +-----+
~~~~
{: #fig-disconnected-topology title="Access network disconnected from core"}

The data flow represented in {{fig-disconnected}} illustrates the communication
between a consumer connected to AR3, and a mobile producer moving from AR1 to
AR2.

~~~~
C         P         AR1        AR2        AR3        GW  X   Internet
|         |          |          |          |          |  X         |
|~~~~~~~~~|~~~~~~~~~~|~~~~~~~~~~|~~~~~~~~~>|          |  X         |
|         |          |          |          |~~~~~~~~~>|  X         |
|         |          |<~~~~~~~~~|~~~~~~~~~~|~~~~~~~~~~|  X         |
|         |<~~~~~~~~~|          |          |          |  X         |
|         |--------->|          |          |          |  X         |
|         |          |----------|----------|--------->|  X         |
|         |          |          |          |<---------|  X         |
|<--------|----------|----------|----------|          |  X         |
|         |          |          |          |          |  X         |
|         |..........-          |          |          |  X         |
|         |..........|..........+          |          |  X         |
|         |==========|=========>o          |          |  X   NO    |
|         | Update   |          |==========|=========>o  X TRAFFIC |
|         |          o<=========|==========|==========|  X         |
|         |          |          |          |          |  X         |
|~~~~~~~~~|~~~~~~~~~~|~~~~~~~~~~|~~~~~~~~~>|          |  X         |
|         |          |          |          |~~~~~~~~~>|  X         |
|         |          |          |<~~~~~~~~~|~~~~~~~~~~|  X         |
|         |<~~~~~~~~~|~~~~~~~~~~|          |          |  X         |
|         |----------|--------->|          |          |  X         |
|         |          |          |----------|--------->|  X         |
|         |          |          |          |<---------|  X         |
|<--------|----------|----------|----------|          |  X         |
|         |          |          |          |          |  X         |
~~~~
{: #fig-disconnected title="Anchorless mobility in network disconnected from core"}

We see that both the data and the signalization remain local to the zone where
the mobility occurs, and that communications during mobility are not affected by
the failure of the link upstream. This is a sign that the mobile core is not
loaded with unnecessary traffic, and that communications remain local, thus
improving user flow latencies. The offload of both data and signalization allows
reducing the cost of the infrastructure by increasing the diversity of resources
used at edge, mutualizing their capacity, and lower requiring network and
compute capacity in the mobile core. A direct consequence is also a more robust
and reliable network.


# Implementation considerations

## Interaction with non-hICN enabled routers

The realization of the architecture in a partial hICN deployment where some
routers are not extended to support hICN mechanisms requires either to introduce
additional functionalities or protocol support, or to reuse existing protocols
achieving similar objectives (following hICN design).

One such example is the combination of ICMP redirect messages and Neighbour
Discovery Proxies (NDProxy), that partially realizes the objectives of the update process:

- ICMP packets do not include sequence numbers however they can be transported
as part of the payload; verification is deferred to the next hICN node which
should send the packet backwards in case of verification failure to fix the
incorrect path update.
- multipath is not supported in the pure-IP part of the network (which is
the expected behaviour)

Identified concerns might be about the unexpected use of such protocols, the
lack of available implementation for NDProxy, and security aspect related to
redirect messages. The latter shares the fate of source routing, which has long
been advocated against, and has recently gained popularity within the SPRING
context. A proper security scheme is certainly the right way to address this
problem, and we believe the set of benefits that we have listed are worth
reconsidering such aspects.

## Security considerations {#sec-security}

As indicated in previous sections, signalization messages transmitted across
trust boundaries must be secured. The choice of the solution will intimately
depend on the selected protocols.

The use of ICMP packet might allow reusing existing security schemes such as AH
headers {{!RFC4302}}, or SEND {{!RFC3971}} (and its proxy extensions {{!RFC6496}},
{{!RFC5909}}).

Alternatively, {{SEC}} has reviewed standard approaches from the
literature and proposes a fast, lightweight and distributed approach that
can be applied to MAP-Me and fits its design principles.

## Discussion

Both consumer and producer mobility support multiple paths, however the support
of mobility for a multihomed producer, is left for future updates of the present
document.

Similarly, the proposed producer mobility solution is appropriate for the
management of micro-mobility; its extension to multiple domains is out of scope.


# IANA Considerations {#iana}

This memo includes no request to IANA.

