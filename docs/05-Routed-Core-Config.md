# Routed Core and WAN Edge Configuration

This section of the lab focuses on configuring the routed core with EIGRP on Nexus switches and OSPF at the WAN/Internet edge. It covers route redistribution between EIGRP and OSPF on edge devices to connect the enterprise network to the WAN.

## Tasks

```
Nexus Distribution/Catalyst Distribution and Routed Core
1. Enable EIGRP on Nexus switches
2. Configure all IP addressed interfaces for EIGRP
3. The loopback0 interface should be the router ID

WAN/Internet Edge
1. Enable OSPF on Nexus switch
2. Configure CSR15 and N9K7 interfaces facing SW26 and SW27 for OSPF

Redistribution
1. Redistribute OSPF and EIGRP mutually on CSR15 and N9K7
```

## Configuration

### 1. Enable EIGRP on Nexus Switches

EIGRP is used for internal routing within the enterprise network on Nexus distribution switches.

N9K7

```
feature eigrp
```

### 2. Configure EIGRP on All IP Interfaces

All routed interfaces participating in the enterprise network should be configured for EIGRP.

N9K7

```
interface Ethernet1/1
 ip address 10.7.15.7/24
 ip router eigrp 1
 no shutdown

interface Ethernet1/2
 ip address 10.7.14.7/24
 ip router eigrp 1
 no shutdown

interface Ethernet1/3
 ip address 10.2.7.7/24
 ip router eigrp 1
 no shutdown

interface Ethernet1/4
 ip address 10.7.22.7/24
 ip router eigrp 1
 no shutdown
```

### 3. Configure EIGRP Router ID

The loopback0 interface IP address should be used as the router ID.

N9K7

```
interface loopback0
 ip address 10.0.0.7/32

router eigrp 1
 router-id 10.0.0.7
```

CSR13

```
interface Loopback0
 ip address 10.0.0.13 255.255.255.255

router eigrp 1
 network 10.0.0.0
```

CSR14

```
interface Loopback0
 ip address 10.0.0.14 255.255.255.255

router eigrp 1
 network 10.0.0.0
 eigrp router-id 10.0.0.14
```

CSR15

```
interface Loopback0
 ip address 10.0.0.15 255.255.255.255

router eigrp 1
 network 10.4.15.0 0.0.0.255
 network 10.7.15.0 0.0.0.255
 network 10.13.15.0 0.0.0.255
 eigrp router-id 10.0.0.15
```

Verification:

```
show ip eigrp neighbors
show ip eigrp interfaces
show ip route eigrp
```

### 4. Enable OSPF on WAN Edge

OSPF is used at the WAN edge to connect to service provider networks.

N9K7

```
feature ospf
```

### 5. Configure OSPF on Interfaces Facing SW26 and SW27

N9K7

```
interface Ethernet1/5
 ip address 10.7.26.7/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

interface Ethernet1/6
 ip address 10.7.27.7/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

router ospf 1
 router-id 10.0.0.7
```

CSR15

```
interface GigabitEthernet4
 ip address 10.15.26.15 255.255.255.0
 negotiation auto

interface GigabitEthernet5
 ip address 10.15.27.15 255.255.255.0
 negotiation auto

router ospf 1
 router-id 10.0.0.15
 network 10.15.26.0 0.0.0.255 area 0
 network 10.15.27.0 0.0.0.255 area 0
```

Verification:

```
show ip ospf neighbors
show ip ospf interface brief
show ip route ospf
```

### 6. Redistribute EIGRP into OSPF

N9K7

```
route-map RM_EIGRP_TO_OSPF permit 10

router ospf 1
 router-id 10.0.0.7
 redistribute eigrp 1 route-map RM_EIGRP_TO_OSPF
```

CSR15

```
router ospf 1
 router-id 10.0.0.15
 redistribute eigrp 1 subnets
```

### 7. Redistribute OSPF into EIGRP

N9K7

```
route-map RM_OSPF_TO_EIGRP permit 10

router eigrp 1
 router-id 10.0.0.7
 redistribute ospf 1 route-map RM_OSPF_TO_EIGRP
```

CSR15

```
router eigrp 1
 redistribute ospf 1 metric 1000000 1 255 1 1500
```

Verification:

```
show ip route eigrp
show ip route ospf
show ip eigrp topology
show ip ospf database
```

## Complete Configuration Examples

### N9K7 (WAN Edge Nexus Switch)

```
feature ospf
feature eigrp

interface Ethernet1/1
 ip address 10.7.15.7/24
 ip router eigrp 1
 no shutdown

interface Ethernet1/2
 ip address 10.7.14.7/24
 ip router eigrp 1
 no shutdown

interface Ethernet1/3
 ip address 10.2.7.7/24
 ip router eigrp 1
 no shutdown

interface Ethernet1/4
 ip address 10.7.22.7/24
 ip router eigrp 1
 no shutdown

interface Ethernet1/5
 ip address 10.7.26.7/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

interface Ethernet1/6
 ip address 10.7.27.7/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

interface loopback0
 ip address 10.0.0.7/32

route-map RM_EIGRP_TO_OSPF permit 10

route-map RM_OSPF_TO_EIGRP permit 10

router eigrp 1
 router-id 10.0.0.7
 redistribute ospf 1 route-map RM_OSPF_TO_EIGRP

router ospf 1
 router-id 10.0.0.7
 redistribute eigrp 1 route-map RM_EIGRP_TO_OSPF
```

### CSR15 (WAN Edge Router)

```
interface Loopback0
 ip address 10.0.0.15 255.255.255.255

interface GigabitEthernet1
 ip address 10.13.15.15 255.255.255.0
 negotiation auto

interface GigabitEthernet2
 ip address 10.7.15.15 255.255.255.0
 negotiation auto

interface GigabitEthernet3
 ip address 10.4.15.15 255.255.255.0
 negotiation auto

interface GigabitEthernet4
 ip address 10.15.26.15 255.255.255.0
 negotiation auto

interface GigabitEthernet5
 ip address 10.15.27.15 255.255.255.0
 negotiation auto

router eigrp 1
 network 10.4.15.0 0.0.0.255
 network 10.7.15.0 0.0.0.255
 network 10.13.15.0 0.0.0.255
 redistribute ospf 1 metric 1000000 1 255 1 1500
 eigrp router-id 10.0.0.15

router ospf 1
 router-id 10.0.0.15
 redistribute eigrp 1 subnets
 network 10.15.26.0 0.0.0.255 area 0
 network 10.15.27.0 0.0.0.255 area 0
```

### CSR13 (Core Router)

```
interface Loopback0
 ip address 10.0.0.13 255.255.255.255

interface GigabitEthernet1
 ip address 10.13.14.13 255.255.255.0
 negotiation auto

interface GigabitEthernet2
 ip address 10.13.15.13 255.255.255.0
 negotiation auto

interface GigabitEthernet3
 ip address 10.8.13.13 255.255.255.0
 negotiation auto

interface GigabitEthernet4
 ip address 10.3.13.13 255.255.255.0
 negotiation auto

router eigrp 1
 network 10.0.0.0
 redistribute bgp 65001 metric 100000 1 255 1 1500 route-map RM_BGP_TO_EIGRP

router bgp 65001
 bgp router-id 10.0.0.13
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 10.8.13.8 remote-as 65002
 
 address-family ipv4
  redistribute eigrp 1 route-map RM_EIGRP_TO_BGP
  neighbor 10.8.13.8 activate
 exit-address-family

route-map RM_BGP_TO_EIGRP permit 10
route-map RM_EIGRP_TO_BGP permit 10
```

### CSR14 (Core Router)

```
interface Loopback0
 ip address 10.0.0.14 255.255.255.255

interface GigabitEthernet1
 ip address 10.13.14.14 255.255.255.0
 negotiation auto

interface GigabitEthernet2
 ip address 10.7.14.14 255.255.255.0
 negotiation auto

interface GigabitEthernet3
 ip address 10.1.14.14 255.255.255.0
 negotiation auto

interface GigabitEthernet4
 ip address 10.10.14.14 255.255.255.0
 negotiation auto

interface GigabitEthernet5
 ip address 10.14.21.14 255.255.255.0
 negotiation auto

router eigrp 1
 network 10.0.0.0
 redistribute bgp 65001 metric 100000 1 255 1 1500 route-map RM_BGP_TO_EIGRP
 eigrp router-id 10.0.0.14

router bgp 65001
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 10.10.14.10 remote-as 65002
 
 address-family ipv4
  redistribute eigrp 1 route-map RM_EIGRP_TO_BGP
  neighbor 10.10.14.10 activate
 exit-address-family

route-map RM_BGP_TO_EIGRP permit 10
route-map RM_EIGRP_TO_BGP permit 10
```

## Configuration Summary

### EIGRP Configuration

| Device | Router ID | Process ID | Redistribution |
|--------|-----------|------------|----------------|
| N9K7 | 10.0.0.7 | 1 | OSPF into EIGRP |
| CSR13 | 10.0.0.13 | 1 | BGP into EIGRP |
| CSR14 | 10.0.0.14 | 1 | BGP into EIGRP |
| CSR15 | 10.0.0.15 | 1 | OSPF into EIGRP |

### OSPF Configuration

| Device | Router ID | Process ID | Area | Redistribution |
|--------|-----------|------------|------|----------------|
| N9K7 | 10.0.0.7 | 1 | 0.0.0.0 | EIGRP into OSPF |
| CSR15 | 10.0.0.15 | 1 | 0 | EIGRP into OSPF |

### BGP Configuration

| Device | AS | Neighbor | Neighbor AS | Redistribution |
|--------|----|---------|-----------|----|
| CSR13 | 65001 | 10.8.13.8 | 65002 | EIGRP into BGP |
| CSR14 | 65001 | 10.10.14.10 | 65002 | EIGRP into BGP |

### Redistribution Points

| Device | EIGRP to OSPF | OSPF to EIGRP | EIGRP to BGP | BGP to EIGRP |
|--------|---------------|---------------|--------------|--------------|
| N9K7 | Yes | Yes | No | No |
| CSR15 | Yes | Yes | No | No |
| CSR13 | No | No | Yes | Yes |
| CSR14 | No | No | Yes | Yes |

## Notes

* EIGRP is used for internal enterprise routing.
* OSPF is used at the WAN edge to connect to service provider networks (SW26, SW27).
* BGP is used to connect to VXLAN leaf switches (AS 65002).
* N9K7 and CSR15 perform mutual redistribution between EIGRP and OSPF.
* CSR13 and CSR14 perform mutual redistribution between EIGRP and BGP.
* Nexus switches use `ip router eigrp` command on interfaces instead of network statements.
* Nexus switches use `ip router ospf` command with area specification on interfaces.
* CSR routers use traditional `network` statements under the EIGRP process.
* EIGRP metric parameters (bandwidth, delay, reliability, load, MTU) are required when redistributing into EIGRP on IOS routers.
* Route-maps are used to control redistribution on all devices.
* CSR13 and CSR14 use `network 10.0.0.0` which includes all interfaces in the 10.0.0.0/8 range.
* BGP neighbors use `no bgp default ipv4-unicast` to require explicit address-family activation.

This configuration connects the enterprise EIGRP domain with the WAN OSPF domain and VXLAN BGP domain, enabling end-to-end connectivity while maintaining separate routing domains.
