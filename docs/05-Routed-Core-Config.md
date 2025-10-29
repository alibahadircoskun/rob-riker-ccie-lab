# Nexus Distribution and Routed Core Configuration

This section of the lab focuses on configuring EIGRP on Nexus distribution switches for internal enterprise routing.

## Tasks

```
1. Enable EIGRP on Nexus switches
2. Configure all IP addressed interfaces for EIGRP
3. The loopback0 interface should be the router ID
4. Redistribute EIGRP into BGP and BGP into EIGRP on core routers
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

Verification:

```
show ip eigrp neighbors
show ip eigrp interfaces
show ip route eigrp
show ip eigrp topology
```

### 4. Redistribute EIGRP into BGP and BGP into EIGRP

Core routers (CSR13, CSR14) connect the EIGRP domain to the BGP domain (VXLAN fabric AS 65002).

CSR13

```
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

CSR14

```
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

Verification:

```
show ip bgp summary
show ip bgp neighbors
show ip route bgp
show ip route eigrp
```

## Complete Configuration Examples

### N9K7 (Nexus Distribution Switch)

```
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

interface loopback0
 ip address 10.0.0.7/32

router eigrp 1
 router-id 10.0.0.7
```

### CSR13 (Core Router with BGP)

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

### CSR14 (Core Router with BGP)

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

| Device | Router ID | EIGRP | BGP AS | BGP Neighbor | Neighbor AS |
|--------|-----------|-------|--------|--------------|-------------|
| N9K7 | 10.0.0.7 | Process 1 | N/A | N/A | N/A |
| CSR13 | 10.0.0.13 | Process 1 | 65001 | 10.8.13.8 | 65002 |
| CSR14 | 10.0.0.14 | Process 1 | 65001 | 10.10.14.10 | 65002 |

### Redistribution

| Device | EIGRP to BGP | BGP to EIGRP |
|--------|--------------|--------------|
| CSR13 | Yes (route-map) | Yes (route-map with metric) |
| CSR14 | Yes (route-map) | Yes (route-map with metric) |

## Notes

* EIGRP is used for internal enterprise routing on Nexus distribution switches.
* Nexus switches use `ip router eigrp` command on interfaces instead of network statements.
* The `feature eigrp` command must be enabled before configuring EIGRP.
* Loopback0 is configured as the router ID for stability.
* All IP addressed interfaces that should participate in EIGRP routing are configured with `ip router eigrp 1`.
* CSR13 and CSR14 connect the EIGRP domain to the BGP domain (VXLAN fabric AS 65002).
* BGP AS 65001 is used for the enterprise core routers.
* BGP AS 65002 is used for the VXLAN leaf switches.
* Mutual redistribution occurs between EIGRP and BGP on CSR13 and CSR14.
* EIGRP metric parameters (bandwidth 100000, delay 1, reliability 255, load 1, MTU 1500) are required when redistributing BGP into EIGRP.
* Route-maps are used to control redistribution and can be configured with additional filtering if needed.
* CSR routers use `network 10.0.0.0` which includes all interfaces in the 10.0.0.0/8 range for EIGRP.

This configuration provides internal routing for the enterprise network using EIGRP and connects to the VXLAN fabric using BGP.
