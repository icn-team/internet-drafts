---
title: Transport Framework for Hybrid Information-Centric Networking Architecture
abbrev: hICN-transport
docname: draft-muscariello-hicn-transport-latest
date:
category: info

ipr: trust200902
area:
workgroup:
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:

-
    ins: L. Muscariello
    name: Luca Muscariello
    org: Cisco Systems Inc.
    email: lumuscar@cisco.com
-
    ins: G. Carofiglio
    name: Giovanna Carofiglio
    org: Cisco Systems Inc.
    email: gcarofig@cisco.com

normative: 
    RFC0793:
    RFC1081:
    RFC1624:
    RFC2119:
    RFC3031:
    RFC3587:
    RFC3550:
    RFC4291:
    RFC4302:
    RFC6830:
    RFC8200:
informative:
    I-D.irtf-icnrg-ccnxsemantics:
    I-D.irtf-icnrg-ccnxmessages:
    I-D.irtf-icnrg-mapme:
    I-D.irtf-icnrg-terminology:
    I-D.muscariello-intarea-hicn:
    CCN: DOI.10.1145/1658939.1658941
    NDN: DOI.10.1145/2656877.2656887
    MAN: DOI.10.1109/INFCOMW.2012.6193505
    WLD: DOI.10.1145/2984356.2984361
    MIR: DOI.10.1109/TMC.2017.2734658
    RAQ: DOI.10.1109/ICNP.2013.6733576

--- abstract

Information-Centric Networking (ICN) architectures introduce a location-independent communication model based on request/reply exchanges between a client and one or more a-priori unknown sources (ref to RFCs). Request packets carry a data indentifier which is not bound to any specific host location, hence is not affected by network mobility. They are routed in a hop-by-hop fashion over one or multiple paths until they are matched by requested data packets. Data packets follow the reverse path of requests to be conveyed to one or more requesting clients. 

Mobility, Multi-path, Multi-source and Many-to-many communication (M4) characterize ICN transport making it a promising candidate for a various number of applications (e.g. live-streaming, IoT, mobile edge computing, realtime multimedia communications, etc.)
The requirements of native M4 transport are not met by traditional host-to-host transport protocols which assume the knowledge of both client and server end-points before packet transmission and do not leverage dynamic forwarding/caching/control decisions taken by in-path nodes.

There have been various attempts in the literature to either adapt existing transport protocols to ICN or to define novel approaches, in all cases targeting a specific application (refs to papers)
A few attemtps have been made at addressing the novel challenges brought by ICN in terms of flow definition, path labelling, etc., but to the best of our knowledge no unified ICN transport framework has been proposed so far.
Without the ambition to design a specific ICN transport protocol, this document discusses a general M4 transport framework for Hybrid ICN, an IP-integrated open-source ICN architecture (ref to FD.io hICN) and  illustrates its applicability to a few relevant use cases.

--- middle

# Introduction


## Problem and Scope

Information-Centric Networking (ICN) identifies a networking paradigm  centering 
communication around named data, in contrast to host location. ICN offers a simplified 
and more efficient user-to-content communication that resolves the mismatch 
between content-centric usage and underlying host-to-host foundations of the Internet.

Despite important architectural differences in the proposed designs (REFS), a shared
idea characterizes ICN, that of abstracting data delivery from underlying point
to point connectivity by means of an information-aware network layer operating
directly on secured named data regardless of their location. 
It results a native support for mobility, storage and security as network features are integrated
by design, rather than as an afterthought.


Receiver-Driven Connectionless Transport – In contrast
with the current sender-based TCP/IP model, ICN transport
is receiver-controlled, does not require connection instantiation
and accommodates retrieval from possibly multiple
dynamically discovered sources. It builds upon the flow
balance principle guaranteeing corresponding request-data
flows on a hop-by-hop basis [9]. A large body of work has
looked into ICN transport (surveyed in [78]), not only to
propose rate and congestion control mechanisms [81, 100] –
especially in the multipath case [30] – but also to highlight
the interaction with in-network caching [29], the coupling
with request routing [30, 50], and the new opportunities
provided by in-network hop-by-hop rate/loss/congestion
control [32, 93] for a more reactive low latency response of
involved network nodes.


- Rephrase around M4 transport capabilities




## Related work
Rel work : a summary
There have been various attempts in the literature to either adapt existing transport protocols to ICN or to define novel approaches, in all cases targeting a specific application (refs to papers)


Rel work on ICN transport protocols


There has been huge amount of work on transport protocolsin the Internet. Here we focus on transport schemes proposedfor CCN. We give a discussion of such work in three aspects.RTT-based congestion control:Some designs, for instanceICP [3] and ICTP [19], rely on a single round trip time (RTT)estimator at the receiver to predict network status which arenot suitable for CCN because CCN flows may have multiplesources and multiple paths. To adapt to CCN’s multipathnature, authors in [5], [6] propose per-route transport control,in which a separate RTT estimation is maintained for eachroute at the receiver, similar to MPTCP [21]. Multiple RTTbased scheme works well for multipath transfer, however, itadds lots of complexity to the receiver and heavily relies onthe accuracy of timing. Our scheme, on the other hand, utilizesexplicit congestion signalling from network to effectivelynotify the receiver about network condition. Compared withmultiple RTT based scheme, our approach leads to a muchsimpler receiver design and is not restricted to limitations oftimers which have long been criticized [9], [11], [18], [24].hop-by-hop  Interest  shaping:Since in CCN one Interestpacket retrieves at most one Data chunk, a router can controlthe rate of future incoming data chunks by shaping the rate ofoutgoing Interests. In [4], [17], authors propose quota-basedInterest shaping scheme to actively control traffic volume.They assign a quota (in terms of the number of pendingInterests) to each flow, and if the number of pending Interestsfor a flow exceeds the quota, that flow’s Interests will bedelayed or dropped. In [22], authors use per interface rate limitto avoid congestion at an interface and per prefix-interfaceratelimit to control Interest rate of each content prefix. The quota-based Interest shaping and the rate limiting Interest controlrequire to assign an appropriate quota value for each flowor rate limit value for each content, which is challenging in practice, if not impossible. And they can be rather inefficientunder dynamic traffic setting. Our fair share Interest shapingscheme also considers about fairness, however, resources areshared by all flows and Interest from a flow is delayedtemporarily only when shared resources become limited andthe corresponding flow unfairly consumes resources.flow-aware traffic control:Authors in [17] propose a flow-aware network paradigm for CCN. They define content flowas packets bearing the same content name identified on the flyby parsing the packet headers. Moreover, routers impose per-flow fair bandwidth sharing by having multiple data queues ateach interface for active flows and using deficit round robin(DRR) [20] for fair scheduling among these queues. Differentfrom such scheme, our FISP realizes per-flow fair sharing byhaving per-flow Interest queues at an interface and utilizingmodified DRR to approximate max-min fairness.

The Information Centric Transport Protocol (ICTP) [REF]adapts the existing ideas of CCNx. ICTP implements thesame algorithm of TCP in its congestion control mechanism,adapting it to the receiver-driven context. In particular, itoperates using interest-data sequences. The loss detection relieson the expected set of data. An equivalent of duplicate ACKin ICTP is the out-of-order received segments, i.e. a gap inthe flow of data. In NetInf TP the order of reception is non-sequential and loss detection relies on the number of receivedsegments after a particular request has been issued.Carofiglioet al.propose the Interest Control Protocol (ICP)as a receiver-driven transport protocol for CCN [REF]. ICPimplements a window-based interest flow control mechanism.For reliability, ICP relies on delay measurements and timerexpirations only; this means, retransmissions are realized byre-issuing interest packets once the timers expire. This alsomeans that re-expression of an interest can happen even if apacket is only delayed. The authors used an analytic modelwith parameter setting to optimize the round-trip timers. Incontrast, NetInf TP uses an additional packet loss detectionmechanism that signals congestion more reliably. 

NetInf TP [REF] applies a cyclic retransmission strategy which savesresources in congested scenarios.

A few attemtps have been made at addressing the novel challenges brought by ICN in terms of flow definition, path labelling, etc., but to the best of our knowledge no unified ICN transport framework has been proposed so far.

Rel work on new challenges  brought by  ICN transport 

- Flow definition
- Path identification
- Out-of-order delivery or variations in inter-arrival inter-vals can no longer be used as an indication of networkcongestion. In fact, a packet might arrive out of ordersimply because it has been sent by a node located furtheraway than the source of the other packets.
- RTO estimation, as defined by Jacobson’s algorithm [6],does not span multiple data sources, and as such it cannotbe used reliably.



## Reference ICN architecture: hICN

Focus here is on defining a transport frameworkf for Hybrid ICN (REF to FD.io hICN)

We focus on hICN (refs) and on the open-sourced implementation in FD.io hICN project (ref)

Hybrid ICN is an architecture that brings ICN into IPv6 as described in [1]. By doing that, hicn allows to generalize IPv6 networking by using location-independent name-based networking. This is made either at the network layer and at the transport layer by also providing name-based sockets to applications.

hicn allows to reuse existing IPv6 protocol and architectures, to extend them and deploy hybrid solutions based on the use case and application needs. Moreover, hicn can exploit hardware and software implementations heavily based on IP, making much simpler insertion of the technology in current networks, current applications and design future applications reusing what the industry already provides in terms of on the shelf components. 


## Terminology

## Notational Conventions

The words "MUST", "MUST NOT", "SHOULD", and "MAY" are used in this document.
It's not shouting; when these words are capitalized, they have a special
meaning as defined in {{RFC2119}}.


# Transport framework architecture

M4 Transport framework provides segmentation and reassembly services to
the applications, and sits on top of hICN network layer which provides a 
request/reply service to the transport layer itself. End-points in TCP/IP can be 
described as acting as transmitters or receivers, or both at the same time, 
whereas hICN transport is oriented to the consumer/producer paradigm where a 
transmission is always triggered by a matching request. Moreover, a TCP connection 
is a bidirectional communication channel, while in hICN there are always two 
unidirectional sockets: the consumer and the producer. Transport protocols in 
hICN can be stream or datagram oriented, depending on the application (as TCP or UDP).
Currently, hICN prototype implements (i) the stream oriented ICN multi-path 
transport protocol with support for loss recovery, flow and congestion control 
introduced  in \cite{carofiglio2016optimal} and (ii) the in-network loss control 
mechanisms in \cite{Papalini-WLDR}. We plan to enrich in the future the set of 
supported transport protocols. As previously mentioned, a transport manifest is used 
by the producer to distribute meta-data useful for consumer to retrieve the actual 
data. It includes low level segmentation information (name suffixes) as well as 
integrity information, and it is signed with the producer's private key~\cite{Manifest}. 
Such approach is adopted as the default \hicn{} approach instead of per packet 
signature computation. The manifest also carries the expiration time of the associated data
which is an absolute quantity and requires reasonable synchronization to UTC.
In case the manifest is not used by the producer, the expiration time is carried
by the TCP timestamp option.


\subsection{Socket API}\label{subsec:socket-api}
We use substantially unaltered the INET6 socket API for \hicn{} with the twofold 
advantage to provide a known API for developers of new applications and simple 
integration of existing ones.
The system calls of \hicn{} socket API are based on the socket interface extensions 
for IPv6 \cite{rfc3493}. Both consumer and producer sockets bind to a socket address 
(\sa{}), which is initialized by specifying address family 
\af  and name prefix. The range of addresses stated by the name prefix 
tells the namespace in which a producer socket is allowed to publish data and a 
consumer socket is allowed to request data.

The \texttt{bind()} system call takes care of setting up a local face to the \hicn{} 
forwarder, which in the case of the producer also sets a FIB entry 
\texttt{(name\_prefix, socket\_id)}. The \texttt{recvmsg()} and the \texttt{recvfrom()} 
system calls are used by a consumer for retrieving data, while \texttt{sendmsg()} and 
\texttt{sendto()} are used by a producer for publishing a content and making it available 
for the consumers. 
After data is published under a given name,  subsequent requests for it can be 
satisfied by the producer socket without passing them to the application.

The consumer socket can use the \texttt{sendmsg()} for sending a single Interest 
with payload eventually triggering a \texttt{recvmsg()} on the remote producer 
to pass the payload to the application, e.g. to send and HTTP request in half RTT.
In general, push semantics are realized using reverse pull techniques, which require all end-points
to have a consumer and a producer socket open and bound to a valid name prefix.
An end-point can push data to one or several remote end-points by (i) calling a 
\texttt{sendto()} from the producer socket with actual data; (ii) passing the resulting 
transport manifest to a valid open consumer socket; (iii) calling a \texttt{sendmsg()} 
from the consumer to trigger an Interest packet carrying the data manifest as payload.
The remote producer (one or many) after reception of Interest can pass the manifest to
the local consumer socket to trigger the reverse pull procedure.
This technique can be used to send large HTTP requests to one or multiple servers, 
and has the advantage to enable multicast upload from one client to many servers.


## Application APIs
Application requirements to tailor underlying transport
## Consumer and Producer Sockets
We use substantially unaltered the INET6 socket API for \hicn{} with the twofold 
advantage to provide a known API for developers of new applications and simple 
integration of existing ones.
The system calls of \hicn{} socket API are based on the socket interface extensions 
for IPv6 \cite{rfc3493}. Both consumer and producer sockets bind to a socket address 
(\sa{}), which is initialized by specifying address family 
\af  and name prefix. The range of addresses stated by the name prefix 
tells the namespace in which a producer socket is allowed to publish data and a 
consumer socket is allowed to request data.

The \texttt{bind()} system call takes care of setting up a local face to the \hicn{} 
forwarder, which in the case of the producer also sets a FIB entry 
\texttt{(name\_prefix, socket\_id)}. The \texttt{recvmsg()} and the \texttt{recvfrom()} 
system calls are used by a consumer for retrieving data, while \texttt{sendmsg()} and 
\texttt{sendto()} are used by a producer for publishing a content and making it available 
for the consumers. 
After data is published under a given name,  subsequent requests for it can be 
satisfied by the producer socket without passing them to the application.

The consumer socket can use the \texttt{sendmsg()} for sending a single Interest 
with payload eventually triggering a \texttt{recvmsg()} on the remote producer 
to pass the payload to the application, e.g. to send and HTTP request in half RTT.
In general, push semantics are realized using reverse pull techniques, which require all end-points
to have a consumer and a producer socket open and bound to a valid name prefix.
An end-point can push data to one or several remote end-points by (i) calling a 
\texttt{sendto()} from the producer socket with actual data; (ii) passing the resulting 
transport manifest to a valid open consumer socket; (iii) calling a \texttt{sendmsg()} 
from the consumer to trigger an Interest packet carrying the data manifest as payload.
The remote producer (one or many) after reception of Interest can pass the manifest to
the local consumer socket to trigger the reverse pull procedure.
This technique can be used to send large HTTP requests to one or multiple servers, 
and has the advantage to enable multicast upload from one client to many servers.

- Transport semantics in L4 packet header 

## Flow identification
...
## Rate and congestion control at the receiver
AIMD-based receiver Interest control(RIC). The receiveradjusts its Interest window based on the AIMD (AdditiveIncrease Multiplicative Decrease) mechanism. Here, re-ceiver detects congestion mostly by marked packets fromupstream routers.

RAAQM paragraph

Other proposals

## Loss detection and control 

at receiver and in the network 

WLDR, MLDR

Interest shaping and control hop-by-hop

# Coexistance with non-ICN transport

# Examples of application
## Live-streaming video distribution
## Realtime Communications

# Security considerations

We also envision for future work, the design of a secure transport layer that 
provides confidentiality to the communication by extending the widely diffused 
point-to-point approach (e.g., as in TLS and DTLS), by means of secure group 
communications~\cite{baugher2005multicast,perrig1999efficient,rafaeli2003survey,sherman2003key} 
to cope and exploit \hicn{} in-network caching and multicasting.

## Consumer
## Producer





