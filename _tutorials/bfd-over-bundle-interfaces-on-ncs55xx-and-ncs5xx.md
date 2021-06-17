---
published: true
date: '2021-06-16 17:54 +0530'
title: BFD over Bundle Interfaces on NCS5500 and NCS500
author: Tejas Lad
excerpt: Understanding the BFD over Bundle Interfaces on NCS5500 and NCS500
tags:
  - iosxr
  - NCS 5500
  - NCS 500
  - BFD
  - Bundle
  - Bundle Ethernet
  - BoB
  - BLB
position: top
---
{% include toc icon="table" title="BFD over Bundle Interfaces on NCS55xx and NCS5xx" %} 

## Introduction

We started the NCS5500 and NCS500 BFD technotes with [BFD architecture](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/#:~:text=BFD%20Feature%20Support,-BFD%20is%20supported&text=Static%2C%20OSPF%2C%20BGP%20and%20IS,are%20supported%20in%20IPv4%20BFD.&text=BFD%20over%20BVI%20is%20supported,dampening%20for%20IPv4%20is%20supported.). Then we looked a bit deeper with the [Hardware Programming](https://xrdocs.io/ncs5500/tutorials/understanding-the-bfd-hardware-programming-on-ncs55xx-and-ncs5xx/). We confirmed the behaviour as per [RFC 5880](https://datatracker.ietf.org/doc/html/rfc5880). We also checked the OAM engine programming in the hardware and saw a few verification commands for quick verification. In this tech-note we will discuss the BFD over Bundles and BFD over Logical Bundles. What are their use cases and see the programming and behaviour w.r.t [RFC 7130](https://datatracker.ietf.org/doc/html/rfc7130) and [RFC 5883](https://datatracker.ietf.org/doc/html/rfc5883).


