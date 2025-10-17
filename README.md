# Enterprise Practice Lab 1

A comprehensive enterprise network lab covering multiple networking technologies and topologies including campus networks, data centers, WAN connectivity, ISP routing, MPLS, DMVPN, and VXLAN overlay networks.

## Lab Overview

This lab simulates a complex enterprise network environment with the following major components:

- **Headquarters (HQ)** - Multi-layer campus network with Catalyst switches
- **Data Center** - Back-to-back vPC configuration with Nexus switches
- **WAN/Internet Edge** - Multi-ISP connectivity with BGP
- **DMVPN Hub-and-Spoke** - Secure site-to-site VPN overlay
- **MPLS Service Provider** - Label-switched WAN backbone
- **VXLAN Fabric** - Modern data center overlay at HQ and colocation sites
- **Multiple ISPs** - BGP routing with confederations and route reflection

## Network Topology

The lab consists of several interconnected network segments:

- Campus distribution and access layer switching
- Nexus-based data center fabric
- Routed core with EIGRP and OSPF
- Three ISP networks with different BGP topologies
- MPLS service provider network
- Remote spoke sites connected via DMVPN and BGP

## Technologies Implemented

### Layer 2 Technologies
- VTP (VLAN Trunking Protocol)
- Spanning Tree Protocol (STP) - Root bridge configuration
- Port Channels / EtherChannel
- 802.1Q Trunking
- vPC (Virtual Port Channel)

### Layer 3 Technologies
- EIGRP (Enhanced Interior Gateway Routing Protocol)
- OSPF (Open Shortest Path First)
- BGP (Border Gateway Protocol)
  - iBGP
  - eBGP
  - BGP Route Reflectors
  - BGP Confederations
- Route Redistribution

### Data Center Technologies
- Nexus vPC (Multi-Chassis EtherChannel)
- HSRP/VRRP Gateway Redundancy
- Nexus Fabric Forwarding
- VXLAN (Virtual Extensible LAN)
- EVPN (Ethernet VPN)
- L2 and L3 VNI (VXLAN Network Identifier)

### WAN Technologies
- DMVPN (Dynamic Multipoint VPN)
  - mGRE tunnels
  - IPsec encryption
  - EIGRP over DMVPN
- MPLS
  - LDP (Label Distribution Protocol)
  - MP-BGP for VPNv4

### Security
- IPsec VPN
- Firewall NAT configuration
- VRF (Virtual Routing and Forwarding)

## Lab Sections

### 1. HQ Campus Network
- Catalyst distribution switches (SW21, SW22)
- VTP domain configuration
- STP optimization for odd/even VLANs
- Rapid transition to forwarding for endpoints

### 2. Nexus Distribution / Catalyst Access
- Nexus 9K switches (N9K1, N9K2)
- vPC domain 12 configuration
- Multi-chassis EtherChannel to access layer
- VLAN 20 and 21 deployment

### 3. Data Center Back-to-Back vPC
- Dual vPC domains (34 and 56)
- Nexus switches (N9K3, N9K4, N9K5, N9K6)
- Inter-switch routing capability
- Redundant upstream connectivity

### 4. Routed Core
- EIGRP on Nexus switches
- OSPF on edge devices (CSR15, N9K7)
- Mutual route redistribution

### 5. Internet Edge
- Firewall NAT for RFC 1918 addresses
- Multiple ISP connectivity

### 6. ISP Networks

#### ISP 1
- OSPF Area 0 backbone
- BGP route reflector topology
- eBGP peerings to customers and other ISPs

#### ISP 2
- OSPF Area 0
- Full mesh iBGP
- Connected route advertisement

#### ISP 3
- BGP confederations (AS 3)
  - Sub-AS 1920 (XRv19, XRv20)
  - Sub-AS 2930 (CSR29, CSR30)
- Inter-confederation peering

### 7. MPLS Service Provider
- OSPF Area 0 backbone
- LDP label distribution
- Dual route reflectors (CSR46, XRv55)
- PE-CE BGP peering

### 8. Spoke Sites
- **CSR32** - Default route to ISP3, local loopbacks
- **CSR33** - Dual-homed to ISP2 and ISP3
- **CSR35** - Multiple BGP peerings, default to ISP3
- **CSR36** - Dual-homed to ISP2 and MPLS with route filtering

### 9. DMVPN
- Hub: CSR31
- Spokes: CSR32, CSR33, CSR35, CSR36
- Phase 1 configuration with weakest crypto options
- Pre-shared key: "enterprise"
- EIGRP as overlay routing protocol

### 10. VXLAN Overlay (HQ, Colo1, Colo2)
- Spine-leaf topology
- OSPF underlay
- BGP EVPN control plane
- L2 VNI: VLAN 100 (1000100), VLAN 101 (1000101)
- L3 VNI: VLAN 33 (10033) for VRF "enterprise"
- Anycast gateway: 0001.0001.0001
- Integration with core routers via eBGP

## Device Types

- **Catalyst Switches** - Access and distribution layer
- **Nexus 9000 Switches** - Data center and VXLAN fabric
- **CSR 1000v Routers** - WAN edge and spoke sites
- **IOS XRv Routers** - ISP and service provider backbone
- **Palo Alto Firewall** - Internet edge security (FW37)
