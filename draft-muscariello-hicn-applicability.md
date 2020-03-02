---
title: Applicability of the Hybrid Information-Centric Networking Architecture
abbrev: hICN Applicability
docname: draft-muscariello-hicn-applicability-latest
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

   This document discusses the applicability of the Hybrid Information-Centric Networking (hICN)
   architecture by showing examples of the development of applications on top of hICN transport 
   and the deployment of hICN routers in the IPv6 Internet.
   Among the many
--- middle

# Introduction

## Notational Conventions

The words "MUST", "MUST NOT", "SHOULD", and "MAY" are used in this document.
It's not shouting; when these words are capitalized, they have a special
meaning as defined in {{RFC2119}}.
