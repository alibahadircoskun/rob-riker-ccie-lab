# VXLAN at HQ Data Center Configuration

This section of the lab focuses on configuring VXLAN overlay networking using Nexus 9000 switches in a spine-leaf topology. It covers EVPN control plane with BGP, Layer 2 and Layer 3 VNI configuration, and VRF-aware VXLAN.

## Topology Overview
- Spine: N9K9 (BGP route reflector)
- Leaf: N9K8, N9K10 (VTEP switches)
- Underlay: OSPF
- Overlay: BGP EVPN
- VRF: enterprise
- VLANs: 100, 101 (L2 VNI), 33 (L3 VNI)

## Tasks

```
1. Enable features needed for VXLAN to operate
   a. NV overlay
   b. Interface VLAN
   c. Vn-segment-based-vlan
   d. Fabric forwarding
   e. BGP
   f. OSPF
2. Form OSPF adjacencies between the Spine and leaf switches
3. Form iBGP route reflector peerings from spine to leaf switches
4. Use vn-segment 1000 and the VLAN ID
   a. VLAN 100 – 1000100
   b. VLAN 101 – 1000101
5. Use the anycast MAC of 0001.0001.0001
6. Create VRF enterprise
7. Use VLAN 33 for L3 VNI
   a. Use vn-segment 10033
8. Apply the VRF to the SVIs
9. Enable EVPN
```

## Configuration

### 1. Enable Required Features

Leaf Switches (N9K8, N9K10)

```
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

nv overlay evpn
```

Spine Switch (N9K9)

```
feature ospf
feature bgp
feature nv overlay

nv overlay evpn
```

### 2. OSPF Underlay Configuration

OSPF provides IP reachability between all switches for the VXLAN underlay.

N9K8

```
interface Ethernet1/1
 ip address 10.8.9.8/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

interface loopback0
 ip address 10.0.0.8/32
 ip router ospf 1 area 0.0.0.0

router ospf 1
 router-id 10.0.0.8
 log-adjacency-changes
```

N9K9

```
interface Ethernet1/1
 ip address 10.8.9.9/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

interface Ethernet1/2
 ip address 10.9.10.9/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

interface loopback0
 ip address 10.0.0.9/32
 ip router ospf 1 area 0.0.0.0

router ospf 1
 router-id 10.0.0.9
 log-adjacency-changes
```

N9K10 (Similar to N9K8)

```
interface Ethernet1/1
 ip address 10.9.10.10/24
 ip router ospf 1 area 0.0.0.0
 no shutdown

interface loopback0
 ip address 10.0.0.10/32
 ip router ospf 1 area 0.0.0.0

router ospf 1
 router-id 10.0.0.10
 log-adjacency-changes
```

Verification:

```
show ip ospf neighbors
show ip route ospf
```

### 3. BGP EVPN Control Plane

N9K9 acts as a route reflector for the EVPN control plane.

N9K9 (Spine - Route Reflector)

```
router bgp 65002
 router-id 10.0.0.9
 log-neighbor-changes
 address-family ipv4 unicast
 address-family l2vpn evpn
 
 neighbor 10.0.0.8
  remote-as 65002
  update-source loopback0
  address-family ipv4 unicast
   route-reflector-client
   soft-reconfiguration inbound always
  address-family l2vpn evpn
   send-community
   send-community extended
   route-reflector-client
 
 neighbor 10.0.0.10
  remote-as 65002
  update-source loopback0
  address-family ipv4 unicast
   route-reflector-client
   soft-reconfiguration inbound always
  address-family l2vpn evpn
   send-community
   send-community extended
   route-reflector-client
```

N9K8 (Leaf)

```
router bgp 65002
 router-id 10.0.0.8
 log-neighbor-changes
 
 neighbor 10.0.0.9
  remote-as 65002
  update-source loopback0
  address-family ipv4 unicast
   soft-reconfiguration inbound always
  address-family l2vpn evpn
   send-community
   send-community extended
```

N9K10 (Leaf - Similar to N9K8)

```
router bgp 65002
 router-id 10.0.0.10
 log-neighbor-changes
 
 neighbor 10.0.0.9
  remote-as 65002
  update-source loopback0
  address-family ipv4 unicast
   soft-reconfiguration inbound always
  address-family l2vpn evpn
   send-community
   send-community extended
```

Verification:

```
show bgp ipv4 unicast summary
show bgp l2vpn evpn summary
```

### 4. VLAN and VNI Configuration

VLANs are mapped to VNIs using the vn-segment command.

Leaf Switches (N9K8, N9K10)

```
vlan 100
 name VLAN100
 vn-segment 1000100

vlan 101
 name VLAN101
 vn-segment 1000101
```

### 5. Anycast Gateway MAC

Configure the same anycast MAC on all leaf switches for distributed gateway functionality.

Leaf Switches (N9K8, N9K10)

```
fabric forwarding anycast-gateway-mac 0001.0001.0001
```

### 6. VRF Configuration

Create VRF enterprise for tenant isolation.

Leaf Switches (N9K8, N9K10)

```
vrf context enterprise
 vni 10033
 rd auto
 address-family ipv4 unicast
  route-target both auto
  route-target both auto evpn
```

### 7. L3 VNI Configuration

VLAN 33 is used as the Layer 3 VNI for VRF enterprise.

Leaf Switches (N9K8, N9K10)

```
vlan 33
 name L3_VNI
 vn-segment 10033
```

### 8. SVI Configuration with VRF

Apply VRF enterprise to the SVIs for VLAN 100 and 101.

N9K8

```
interface Vlan33
 no shutdown
 vrf member enterprise
 ip forward

interface Vlan100
 no shutdown
 vrf member enterprise
 ip address 172.16.100.1/24 tag 65002
 fabric forwarding mode anycast-gateway

interface Vlan101
 no shutdown
 vrf member enterprise
 ip address 172.16.101.1/24 tag 65002
 fabric forwarding mode anycast-gateway
```

N9K10 (Same configuration)

```
interface Vlan33
 no shutdown
 vrf member enterprise
 ip forward

interface Vlan100
 no shutdown
 vrf member enterprise
 ip address 172.16.100.1/24 tag 65002
 fabric forwarding mode anycast-gateway

interface Vlan101
 no shutdown
 vrf member enterprise
 ip address 172.16.101.1/24 tag 65002
 fabric forwarding mode anycast-gateway
```

### 9. EVPN and NVE Interface

Configure the NVE interface for VXLAN tunneling.

Leaf Switches (N9K8, N9K10)

```
interface nve1
 no shutdown
 host-reachability protocol bgp
 source-interface loopback0
 member vni 10033 associate-vrf
 member vni 1000100
  ingress-replication protocol bgp
 member vni 1000101
  ingress-replication protocol bgp
```

Configure EVPN for L2 VNIs.

Leaf Switches (N9K8, N9K10)

```
evpn
 vni 1000100 l2
  rd auto
  route-target import auto
  route-target export auto
 vni 1000101 l2
  rd auto
  route-target import auto
  route-target export auto
```

### 10. Access Port Configuration

Configure access ports for endpoint connectivity.

N9K8

```
interface Ethernet1/3
 switchport
 switchport access vlan 100
 spanning-tree port type edge
 no shutdown

interface Ethernet1/4
 switchport
 switchport access vlan 101
 spanning-tree port type edge
 no shutdown
```

N9K10 (Similar configuration)

```
interface Ethernet1/3
 switchport
 switchport access vlan 100
 spanning-tree port type edge
 no shutdown

interface Ethernet1/4
 switchport
 switchport access vlan 101
 spanning-tree port type edge
 no shutdown
```

### 11. BGP Redistribution for VRF

Configure route redistribution for VRF enterprise to advertise connected subnets.

Leaf Switches (N9K8, N9K10)

```
route-map BGP_DIRECT permit 10
 match tag 65002

router bgp 65002
 vrf enterprise
  address-family ipv4 unicast
   redistribute direct route-map BGP_DIRECT
```

Verification:

```
show nve peers
show nve vni
show bgp l2vpn evpn
show vxlan
show mac address-table
show ip route vrf enterprise
show bgp vrf enterprise ipv4 unicast
```

## Configuration Summary

### Underlay Network

| Device | Role | Loopback0 | OSPF Area | Router ID |
|--------|------|-----------|-----------|-----------|
| N9K8 | Leaf | 10.0.0.8/32 | 0.0.0.0 | 10.0.0.8 |
| N9K9 | Spine | 10.0.0.9/32 | 0.0.0.0 | 10.0.0.9 |
| N9K10 | Leaf | 10.0.0.10/32 | 0.0.0.0 | 10.0.0.10 |

### Overlay Network

| Parameter | Value |
|-----------|-------|
| BGP AS | 65002 |
| Route Reflector | N9K9 (10.0.0.9) |
| EVPN Address Family | l2vpn evpn |
| Anycast Gateway MAC | 0001.0001.0001 |

### VLAN and VNI Mapping

| VLAN | Name | VNI | Type | Subnet |
|------|------|-----|------|--------|
| 33 | L3_VNI | 10033 | L3 VNI | N/A |
| 100 | VLAN100 | 1000100 | L2 VNI | 172.16.100.0/24 |
| 101 | VLAN101 | 1000101 | L2 VNI | 172.16.101.0/24 |

### VRF Configuration

| VRF | L3 VNI | VLAN | Route Target |
|-----|--------|------|--------------|
| enterprise | 10033 | 33 | auto |

## Notes

* OSPF provides IP reachability in the underlay for loopback addresses.
* BGP EVPN is used for the control plane to exchange MAC and IP information.
* N9K9 acts as a route reflector to reduce BGP peering requirements.
* VNI follows the format 1000 + VLAN ID for L2 VNIs.
* L3 VNI 10033 enables inter-subnet routing within VRF enterprise.
* Anycast gateway allows all leaf switches to use the same gateway IP and MAC.
* The tag 65002 on SVI IP addresses is used for route-map matching in BGP redistribution.
* ingress-replication protocol bgp is used for BUM (Broadcast, Unknown unicast, Multicast) traffic.
* NVE interface uses loopback0 as the source for VXLAN tunnels.
* route-target both auto evpn enables automatic RT generation for EVPN routes.
* fabric forwarding mode anycast-gateway enables distributed gateway functionality.
* All leaf switches use identical SVI IP addresses for anycast gateway operation.

This configuration provides a scalable VXLAN overlay network with Layer 2 extension across the fabric and distributed Layer 3 gateway functionality.
