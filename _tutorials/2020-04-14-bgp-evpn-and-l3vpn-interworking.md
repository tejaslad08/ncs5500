---
published: true
date: '2020-04-14 16:35 -0700'
title: BGP-EVPN and L3VPN Interworking
author: Ahmad Bilal
excerpt: BGP-EVPN and L3VPN Interworking Support on NCS5500
---
{% include toc %}

## BGP-EVPN and L3VPN Interworking Support on NCS5500

BGP EVPN and L3VPN interworking is a way to connect EVPN domain such as a DC or CO over an IPVPN network. This is generally needed when we want end-to-end connectivity of Hosts/CEs in EVPN domain to connect to other domains over an IPVPN network providing inter-subnet routing.
Below topology shows an EVPN domain connecting to L3VPN domain, where interworking is done on Boarder-Leaf/DCI routers. Boarder-Leaf/DCI routers are essential in these types of designs to keep prefixes local to domain and send summarized advertisement out.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-topology.png?raw=true)

In this post we will go over the configuration of EVPN and L3VPN interworking on NCS 5500 routers acting as DCI. When we complete the configuration, we will see the Host subnet (10.0.0.0/24) learnt on PE-1, and PE-1’s loopback-100 prefix learnt on Leafs. This will provide end-to-end reachability between Hosts and PE-1’s prefixes, that are advertised by different BGP address families. The configuration setup is based on single BGP AS 65001 but separate ISIS routing domains for EVPN and L3VPN. DCI is participating in both ISIS domains performing EVPN and L3VPN interworking.

As we look at the setup, there are four major tasks we have to implement to achieve end-to-end connectivity. Some of these tasks are already covered in previous posts, their details will not be covered here. Click on the links below visit previous posts. 
•	Configure Segment Routing
•	Configure EVPN Layer-2 Service and IRB
•	Configure L3VPN
•	Configure DCI and Leafs to support EVPN and L3VPN interworking


### Task:1 Configuration of Segment Routing on DCI.
Segment routing configuration is covered in earlier post. For this we will show segment routing configuration of DCI as DCI is part of two segment routing domains. One segment routing domain is providing forwarding to EVPN fabric and the other to L3VPN domain.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-transport.png?raw=true)


    <table style="border-collapse: collapse; border: none;">
        <tr style="border: none;">
            <td style="border: none;">
DCI-1 
            </td>
            <td style="border: none;">
DCI-2
            </td>
        </tr>
        <tr style="border: none;">
            <td style="border: none;">
input1 
            </td>
            <td style="border: none;">
input2
            </td>
        </tr>
    </table> 


| **DCI-1** | **DCI-2** |
| router isis 1 | router isis 1 |
| is-type level-2-only | is-type level-2-only |
| net 49.0001.0000.0000.0008.00 | net 49.0001.0000.0000.0009.00 |
| nsr | nsr |
| log adjacency changes | log adjacency changes |
| address-family ipv4 unicast | address-family ipv4 unicast |
|  metric-style wide | metric-style wide |
|  segment-routing mpls | segment-routing mpls |
| ! | ! |
| interface Bundle-Ether68 | interface Bundle-Ether69 |
|  point-to-point | point-to-point |
|  address-family ipv4 unicast | address-family ipv4 unicast |
| ! | ! |
| interface Bundle-Ether78 | interface Bundle-Ether79 |
|  point-to-point | point-to-point |
|  address-family ipv4 unicast | address-family ipv4 unicast |
| ! | ! |
| interface Loopback0 | interface Loopback0 |
|  passive | passive |
|  address-family ipv4 unicast | address-family ipv4 unicast |
|   prefix-sid absolute 16008 | prefix-sid absolute 16009 |
| ! | ! |
| router isis 2 | router isis 2 |
| is-type level-2-only | is-type level-2-only |
| net 49.0002.0000.0000.0008.00 | net 49.0002.0000.0000.0009.00 |
| nsr | nsr |
| log adjacency changes | log adjacency changes |
| address-family ipv4 unicast | address-family ipv4 unicast |
|  metric-style wide | metric-style wide |
|  segment-routing mpls | segment-routing mpls |
| ! | ! |
| interface Bundle-Ether81 | interface Bundle-Ether91 |
|  point-to-point | point-to-point |
|  address-family ipv4 unicast | address-family ipv4 unicast |
| ! | ! |
| interface Loopback0 | interface Loopback0 |
|  passive | passive |
|  address-family ipv4 unicast | address-family ipv4 unicast |
|   prefix-sid absolute 16008 | prefix-sid absolute 16009 |
| ! | ! |  
 
 
 
 
  
| **DCI-1** | **DCI-2** |  
| DCI-1#show isis segment-routing label table 

IS-IS 1 IS Label Table
Label         Prefix/Interface
----------    ----------------
16001         1.1.1.1/32
16002         2.2.2.2/32
16006         6.6.6.6/32
16007         7.7.7.7/32
16008         Loopback0
16009         9.9.9.9/32

IS-IS 2 IS Label Table
Label         Prefix/Interface
----------    ----------------
16008         Loopback0
16009         9.9.9.9/32
16010         10.10.10.10/32 
 | DCI-2#show isis segment-routing label table 

IS-IS 1 IS Label Table
Label         Prefix/Interface
----------    ----------------
16001         1.1.1.1/32
16002         2.2.2.2/32
16006         6.6.6.6/32
16007         7.7.7.7/32
16008         8.8.8.8/32
16009         Loopback0

IS-IS 2 IS Label Table
Label         Prefix/Interface
----------    ----------------
16008         8.8.8.8/32
16009         Loopback0
16010         10.10.10.10/32
 |  






| **Loopback 0** | **Prefix-SID** | **ISIS Net** |
| Spine-1 6.6.6.6/32 | 16006 | 49.0001.0000.0000.0006.0 |
| Spine-2 7.7.7.7/32 | 16007 | 49.0001.0000.0000.0007.0 |
| Leaf-1  1.1.1.1/32 | 16001 | 49.0001.0000.0000.0001.0 |
| Leaf-2  2.2.2.2/32 | 16002 | 49.0001.0000.0000.0002.0 |
| Leaf-5  5.5.5.5/32 | 16005 | 49.0001.0000.0000.0005.0 |