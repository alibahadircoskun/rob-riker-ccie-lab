# Data Center - Back-to-Back vPC Configuration

## Topology Overview

**Upper vPC Domain (34):**
- N9K3, N9K4 (Nexus 9000 switches with Layer 3 capability)
- Peer-link: Eth1/2-3 (Port-channel 100)
- Peer-keepalive: Eth1/1 (10.1.2.0/30) in VRF peer-keepalive

**Lower vPC Domain (56):**
- N9K5, N9K6 (Nexus 9000 switches)
- Peer-link: Eth1/2-3 (Port-channel 100)
- Peer-keepalive: Eth1/1 (10.1.2.0/30) in VRF peer-keepalive

**Back-to-Back vPC:**
- Port-channel 45: N9K3/N9K4 ↔ N9K5/N9K6

**Access Layer:**
- SW12 (Catalyst switch)
- Port-channel 12 to N9K5/N9K6 (vPC 6)

**VLANs:**
- VLAN 200

## Configuration Tasks

### 1. Enable N9K5 and N9K6 to Support Multi-Chassis EtherChannel

N9K5 and N9K6 only require basic vPC functionality without Layer 3 routing.

#### N9K5

```cisco
configure terminal
feature lacp
feature vpc
end
```

#### N9K6

```cisco
configure terminal
feature lacp
feature vpc
end
```

### 2. Enable N9K3 and N9K4 to Support Multi-Chassis EtherChannel, Gateway Redundancy with Industry Standard Protocol and SVI Support

N9K3 and N9K4 require Layer 3 capability with VRRP for gateway redundancy.

#### N9K3

```cisco
configure terminal
feature vrrp
feature eigrp
feature interface-vlan
feature lacp
feature vpc
end
```

#### N9K4

```cisco
configure terminal
feature vrrp
feature eigrp
feature interface-vlan
feature lacp
feature vpc
end
```

### 3. Eth1/1 on Both Switches Should Be in Its Own RIB and Be Pingable Over 10.1.2.0/30

The peer-keepalive link uses a separate VRF (peer-keepalive) to isolate management traffic.

#### Create VRF for Peer-Keepalive

**N9K3, N9K4, N9K5, N9K6**

```cisco
configure terminal
vrf context peer-keepalive
exit
end
```

#### N9K3 and N9K5 - Should Have 10.1.2.1/30

**N9K3**

```cisco
configure terminal
interface Ethernet1/1
  vrf member peer-keepalive
  ip address 10.1.2.1/30
  no shutdown
exit
end
```

**N9K5**

```cisco
configure terminal
interface Ethernet1/1
  vrf member peer-keepalive
  ip address 10.1.2.1/30
  no shutdown
exit
end
```

#### N9K4 and N9K6 - Should Have 10.1.2.2/30

**N9K4**

```cisco
configure terminal
interface Ethernet1/1
  vrf member peer-keepalive
  ip address 10.1.2.2/30
  no shutdown
exit
end
```

**N9K6**

```cisco
configure terminal
interface Ethernet1/1
  vrf member peer-keepalive
  ip address 10.1.2.2/30
  no shutdown
exit
end
```

**Verification:**

```cisco
ping 10.1.2.2 vrf peer-keepalive
```

### 4. Eth1/2-3 Should Be Configured to Exchange Information Between the Switches

The peer-link carries vPC control traffic and synchronizes MAC/ARP tables between peers.

#### N9K3

```cisco
configure terminal
interface Ethernet1/2
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit

interface Ethernet1/3
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit
end
```

#### N9K4

```cisco
configure terminal
interface Ethernet1/2
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit

interface Ethernet1/3
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit
end
```

#### N9K5

```cisco
configure terminal
interface Ethernet1/2
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit

interface Ethernet1/3
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit
end
```

#### N9K6

```cisco
configure terminal
interface Ethernet1/2
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit

interface Ethernet1/3
  switchport
  switchport mode trunk
  channel-group 100 mode active
  no shutdown
exit
end
```

### 5. Configure vPC Domain 34

#### N9K3 and N9K4

**N9K3**

```cisco
configure terminal
vpc domain 34
  peer-switch
  role priority 1
  peer-keepalive destination 10.1.2.2 source 10.1.2.1 vrf peer-keepalive
  peer-gateway
  layer3 peer-router
exit
end
```

**N9K4**

```cisco
configure terminal
vpc domain 34
  peer-switch
  role priority 2
  peer-keepalive destination 10.1.2.1 source 10.1.2.2 vrf peer-keepalive
  peer-gateway
  layer3 peer-router
exit
end
```

### 6. Configure vPC Domain 56

#### N9K5 and N9K6

**N9K5**

```cisco
configure terminal
vpc domain 56
  peer-switch
  role priority 1
  peer-keepalive destination 10.1.2.2 source 10.1.2.1 vrf peer-keepalive
  peer-gateway
  layer3 peer-router
exit
end
```

**N9K6**

```cisco
configure terminal
vpc domain 56
  peer-switch
  role priority 2
  peer-keepalive destination 10.1.2.1 source 10.1.2.2 vrf peer-keepalive
  peer-gateway
  layer3 peer-router
exit
end
```

### 7. Both Switches Should Appear as a Single Switch to SW12

This is accomplished through vPC configuration on the port-channel connecting to SW12.

### 8. N9K3 and N9K4 Should Be Able to Route Between Themselves

Enable Layer 3 routing with VRRP for gateway redundancy on VLAN 200.

#### Create VLAN 200

**N9K3 and N9K4**

```cisco
configure terminal
vlan 200
  name VLAN200
exit
end
```

#### Configure Port-Channel 100 as vPC Peer-Link

**N9K3**

```cisco
configure terminal
interface port-channel100
  switchport
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link
exit
end
```

**N9K4**

```cisco
configure terminal
interface port-channel100
  switchport
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link
exit
end
```

#### Configure SVIs for Layer 3 Routing

**N9K3**

```cisco
configure terminal
interface Vlan200
  no shutdown
  no ip redirects
  ip address 172.31.200.3/24
  no ipv6 redirects
  ip router eigrp 1
  ip passive-interface eigrp 1
  vrrp 200
    priority 150
    address 172.31.200.1 
    no shutdown
exit
end
```

**N9K4**

```cisco
configure terminal
interface Vlan200
  no shutdown
  no ip redirects
  ip address 172.31.200.4/24
  no ipv6 redirects
  ip router eigrp 1
  ip passive-interface eigrp 1
  vrrp 200
    address 172.31.200.1 
    no shutdown
exit
end
```

**Configure EIGRP**

**N9K3**

```cisco
configure terminal
router eigrp 1
  router-id 10.0.0.3
exit
end
```

**N9K4**

```cisco
configure terminal
router eigrp 1
  router-id 10.0.0.4
exit
end
```

### 9. N9K3 Should Be the Primary Device

This is accomplished by setting role priority 1 on N9K3 (already configured in step 5).

### 10. Eth1/4 and Eth1/5 on Both Switches Needs to Support Multiple VLANs and Form a Port-Channel

This creates the back-to-back vPC connection between domain 34 and domain 56.

#### N9K3 and N9K4 (Upper vPC Domain)

**N9K3**

```cisco
configure terminal
interface Ethernet1/4
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface Ethernet1/5
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface port-channel45
  switchport
  switchport mode trunk
exit
end
```

**N9K4**

```cisco
configure terminal
interface Ethernet1/4
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface Ethernet1/5
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface port-channel45
  switchport
  switchport mode trunk
exit
end
```

#### N9K5 and N9K6 (Lower vPC Domain)

**N9K5**

```cisco
configure terminal
interface Ethernet1/4
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface Ethernet1/5
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface port-channel45
  switchport
  switchport mode trunk
exit

! Configure peer-link
interface port-channel100
  switchport
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link
exit
end
```

**N9K6**

```cisco
configure terminal
interface Ethernet1/4
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface Ethernet1/5
  switchport
  switchport mode trunk
  channel-group 45 mode active
  no shutdown
exit

interface port-channel45
  switchport
  switchport mode trunk
exit

! Configure peer-link
interface port-channel100
  switchport
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link
exit
end
```

### 11. SW12 G0/0-1 Should Be Configured as Trunk Links and Be Treated as Single Logical Port

#### SW12

```cisco
configure terminal
interface GigabitEthernet0/0
  switchport trunk encapsulation dot1q
  switchport mode trunk
  channel-protocol lacp
  channel-group 12 mode active
exit

interface GigabitEthernet0/1
  switchport trunk encapsulation dot1q
  switchport mode trunk
  channel-protocol lacp
  channel-group 12 mode active
exit

interface Port-channel12
  switchport trunk encapsulation dot1q
  switchport mode trunk
exit
end
```

### 12. Create VLAN 20 and 21 on All Switches

**Note:** The actual lab configuration uses VLAN 200 instead of VLAN 20 and 21.

#### All Nexus Switches (N9K3, N9K4, N9K5, N9K6)

```cisco
configure terminal
vlan 200
  name VLAN200
exit
end
```

### 13. Downstream Interfaces on SW12 Should Be Configured to Support Endpoints and Begin Forwarding Immediately

#### SW12 (Endpoint Interfaces)

```cisco
configure terminal
interface GigabitEthernet0/2
  switchport access vlan 200
  switchport mode access
  spanning-tree portfast edge
exit

interface GigabitEthernet0/3
  switchport access vlan 200
  switchport mode access
  spanning-tree portfast edge
exit

! Repeat for all endpoint interfaces as needed
end
```

### Configure vPC on N9K5/N9K6 for Connection to SW12

The connection from N9K5/N9K6 to SW12 requires vPC configuration.

#### N9K5

```cisco
configure terminal
interface Ethernet1/6
  switchport
  switchport mode trunk
  channel-group 6 mode active
  no shutdown
exit

interface port-channel6
  switchport
  switchport mode trunk
  vpc 6
exit
end
```

#### N9K6

```cisco
configure terminal
interface Ethernet1/6
  switchport
  switchport mode trunk
  channel-group 6 mode active
  no shutdown
exit

interface port-channel6
  switchport
  switchport mode trunk
  vpc 6
exit
end
```

## Verification Commands

```cisco
! vPC Status
show vpc
show vpc brief
show vpc peer-keepalive
show vpc role
show vpc 6

! vPC Consistency
show vpc consistency-parameters global
show vpc consistency-parameters interface port-channel100

! Port-Channel Status
show port-channel summary
show lacp interface port-channel100
show lacp neighbor

! VLAN Information
show vlan brief

! Interface Status
show interface trunk
show interface status

! Spanning Tree
show spanning-tree summary
show spanning-tree vlan 200

! VRRP Status (N9K3 and N9K4)
show vrrp brief
show vrrp interface vlan 200

! EIGRP Status (N9K3 and N9K4)
show ip eigrp neighbors
show ip eigrp interfaces

! Connectivity Tests
ping 10.1.2.2 vrf peer-keepalive
ping 172.31.200.4 source 172.31.200.3
```

## Configuration Summary

### vPC Domain 34 (N9K3 and N9K4)

| Parameter | N9K3 | N9K4 |
|-----------|------|------|
| Role | Primary (priority 1) | Secondary (priority 2) |
| Peer-Keepalive | 10.1.2.1/30 (VRF peer-keepalive) | 10.1.2.2/30 (VRF peer-keepalive) |
| Peer-Link | Po100 (Eth1/2-3) | Po100 (Eth1/2-3) |
| Back-to-Back Link | Po45 (Eth1/4-5) | Po45 (Eth1/4-5) |
| VLAN 200 SVI | 172.31.200.3/24 | 172.31.200.4/24 |
| VRRP VLAN 200 | Active (priority 150) | Standby (default 100) |
| EIGRP Router ID | 10.0.0.3 | 10.0.0.4 |

### vPC Domain 56 (N9K5 and N9K6)

| Parameter | N9K5 | N9K6 |
|-----------|------|------|
| Role | Primary (priority 1) | Secondary (priority 2) |
| Peer-Keepalive | 10.1.2.1/30 (VRF peer-keepalive) | 10.1.2.2/30 (VRF peer-keepalive) |
| Peer-Link | Po100 (Eth1/2-3) | Po100 (Eth1/2-3) |
| Back-to-Back Link | Po45 (Eth1/4-5) | Po45 (Eth1/4-5) |
| vPC to SW12 | vPC 6 (Po6, Eth1/6) | vPC 6 (Po6, Eth1/6) |

### Access Layer (SW12)

| Parameter | Value |
|-----------|-------|
| Uplink | Po12 (G0/0-1) to N9K5/N9K6 |
| VLAN | 200 |
| Endpoint Ports | G0/2-3 (PortFast enabled) |

## Notes

### vPC Design Configuration

**Peer-Keepalive Link:**
- Uses separate VRF (peer-keepalive) for isolation
- Carries heartbeat messages between vPC peers
- Uses Ethernet1/1 on all switches
- Network: 10.1.2.0/30

**Peer-Link:**
- Port-channel 100 using Ethernet1/2-3
- Carries vPC control protocol messages
- Synchronizes MAC and ARP tables
- Uses `spanning-tree port type network` to prevent topology changes

**Back-to-Back vPC:**
- Port-channel 45 using Ethernet1/4-5
- Connects vPC domain 34 (N9K3/N9K4) to vPC domain 56 (N9K5/N9K6)
- Each domain sees the other as a single logical switch
- Does NOT require vPC configuration on Po45 (regular port-channel)

**vPC to Access Layer:**
- Port-channel 6 on N9K5/N9K6 with `vpc 6` configuration
- Connects to SW12 Port-channel 12
- SW12 sees N9K5 and N9K6 as a single logical switch

### VRRP Configuration

- **Industry standard protocol**: VRRP is used instead of HSRP
- **Gateway**: 172.31.200.1 for VLAN 200
- **N9K3**: Active gateway with priority 150
- **N9K4**: Standby gateway with default priority 100
- Both switches can route traffic for VLAN 200

### Role Priority

- **Priority 1**: Primary device (N9K3, N9K5)
- **Priority 2**: Secondary device (N9K4, N9K6)
- Lower value = higher priority
- Primary device handles vPC operations during failures

### PortFast Configuration

- Enabled on SW12 endpoint interfaces (G0/2, G0/3)
- Uses `spanning-tree portfast edge` command
- Transitions port to forwarding immediately
- Should NEVER be used on trunk or uplink ports

### Considerations

1. **Always configure peer-keepalive before peer-link**
2. **Verify peer-keepalive connectivity**: `ping 10.1.2.2 vrf peer-keepalive`
3. **Check vPC status after configuration**: `show vpc`
4. **Monitor consistency parameters**: `show vpc consistency-parameters global`
5. **Use `peer-switch`**: Allows both switches to appear as a single STP root
6. **Use `peer-gateway`**: Enables forwarding of packets destined to peer's MAC
7. **Use `layer3 peer-router`**: Optimizes OSPF/EIGRP routing over vPC
8. **LACP mode active**: Ensures dynamic port-channel negotiation

### Troubleshooting

**vPC not forming:**
- Verify peer-keepalive VRF configuration
- Check IP connectivity: `ping 10.1.2.2 vrf peer-keepalive`
- Verify peer-link interfaces are up
- Confirm vPC domain configuration matches

**Inconsistent vPC parameters:**
- Run `show vpc consistency-parameters global`
- Common issues: VLAN mismatch, STP configuration, MTU mismatch
- Fix inconsistencies on one or both peers

**Port-channel not forming:**
- Check LACP mode on both sides
- Verify VLANs are allowed on trunk
- Confirm physical interfaces are up
- Review `show port-channel summary`
