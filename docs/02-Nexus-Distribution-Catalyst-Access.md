# Nexus Distribution/Catalyst Access Configuration

## Topology Overview
- Distribution Layer: N9K1, N9K2 (Nexus 9000 switches)
- Access Layer: SW11 (Catalyst switch)
- vPC Domain: 12
- VLANs: 20, 21
- Peer-link: Eth1/2-3 on both N9K switches
- vPC Member Port: Eth1/4 on both N9K switches to SW11

## Configuration Tasks

### 1. Enable Required Features on N9K1 and N9K2

Enable support for Multi-Chassis EtherChannel (vPC), gateway redundancy (HSRP), and SVI routing.

N9K1 and N9K2
```
configure terminal
feature vpc
feature lacp
feature hsrp
feature interface-vlan
end
```

### 2. Configure Keepalive Link (Eth1/1)

The keepalive link should be in its own VRF and routing table for management traffic.

N9K1
```
configure terminal
vrf context peer-keepalive

interface Ethernet1/1
description vPC Keepalive Link
no switchport
vrf member keepalive
ip address 10.1.2.1/30
no shutdown
end
```

N9K2
```
configure terminal
vrf context peer-keepalive

interface Ethernet1/1
description vPC Keepalive Link
no switchport
vrf member peer-keepalive
ip address 10.1.2.2/30
no shutdown
end
```

Verify Keepalive Connectivity
```
ping 10.1.2.2 vrf peer-keepalive   ! From N9K1
ping 10.1.2.1 vrf peer-keepalive   ! From N9K2
```

### 3. Configure Peer-Link (Eth1/2-3)

The peer-link exchanges vPC control information and carries data traffic.

N9K1 and N9K2
```
configure terminal
interface Ethernet1/2-3
description vPC Peer-Link
switchport mode trunk
switchport trunk allowed vlan all
channel-group 1 mode active
no shutdown
end
```

### 4. Configure vPC Domain 12

N9K1
```
configure terminal
vpc domain 12
peer-keepalive destination 10.1.2.2 source 10.1.2.1 vrf keepalive
end
```

N9K2
```
configure terminal
vpc domain 12
peer-keepalive destination 10.1.2.1 source 10.1.2.2 vrf keepalive
end
```

### 5. Configure Peer-Link Port-Channel

Both switches should appear as a single logical switch to downstream devices.

N9K1 and N9K2
```
configure terminal
interface port-channel1
description vPC Peer-Link
switchport mode trunk
switchport trunk allowed vlan all
vpc peer-link
no shutdown
end
```

### 6. Enable Layer 3 Routing Between Switches

Ensure both switches can route between themselves using the peer-link.

N9K1 and N9K2
```
configure terminal
feature ospf
ip routing
end
```

The peer-link automatically carries Layer 3 traffic when SVIs are configured.

### 7. Configure N9K1 as Primary Device

Set role priority to make N9K1 the primary vPC switch.

N9K1
```
configure terminal
vpc domain 12
role priority 1
end
```

N9K2
```
configure terminal
vpc domain 12
role priority 2
end
```

Lower priority value = primary device

### 8. Configure vPC Member Port (Eth1/4)

Create port-channel for downstream connection to SW11.

N9K1 and N9K2
```
configure terminal
interface Ethernet1/4
description vPC to SW11
switchport mode trunk
switchport trunk allowed vlan all
channel-group 10 mode active
no shutdown
end
```

Configure vPC on Port-Channel

N9K1 and N9K2
```
configure terminal
interface port-channel10
description vPC to SW11
switchport mode trunk
switchport trunk allowed vlan all
vpc 10
no shutdown
end
```

### 9. Configure SW11 Port-Channel

SW11 should see both N9K switches as a single logical switch.

SW11
```
configure terminal
interface range GigabitEthernet0/0-1
description Port-Channel to N9K1 and N9K2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
channel-group 10 mode active
no shutdown
end

interface port-channel10
description Uplink to Nexus Distribution
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
end
```

### 10. Create VLANs 20 and 21

N9K1 and N9K2
```
configure terminal
vlan 20
name VLAN20
exit

vlan 21
name VLAN21
exit
end
```

SW11
```
configure terminal
vlan 20
name VLAN20
exit

vlan 21
name VLAN21
exit
end
```

### 11. Configure Endpoint Ports on SW11

Downstream interfaces should support endpoints and begin forwarding immediately.

SW11
```
configure terminal
interface range GigabitEthernet0/2-24
switchport mode access
spanning-tree portfast edge
no shutdown
end
```

Optional: Configure specific VLANs
```
configure terminal
interface range GigabitEthernet0/2-12
switchport access vlan 20
exit

interface range GigabitEthernet0/13-24
switchport access vlan 21
end
```

## Configuration Examples

### N9K1 Configuration
```
configure terminal

feature vpc
feature lacp
feature hsrp
feature interface-vlan
ip routing

vrf context keepalive

interface Ethernet1/1
description vPC Keepalive Link
no switchport
vrf member peer-keepalive
ip address 10.1.2.1/30
no shutdown
exit

interface Ethernet1/2-3
description vPC Peer-Link
switchport mode trunk
switchport trunk allowed vlan all
channel-group 1 mode active
no shutdown
exit

interface port-channel1
description vPC Peer-Link
switchport mode trunk
switchport trunk allowed vlan all
vpc peer-link
no shutdown
exit

vpc domain 12
peer-keepalive destination 10.1.2.2 source 10.1.2.1 vrf peer-keepalive
role priority 10
exit

interface Ethernet1/4
description vPC to SW11
switchport mode trunk
switchport trunk allowed vlan all
channel-group 10 mode active
no shutdown
exit

interface port-channel10
description vPC to SW11
switchport mode trunk
switchport trunk allowed vlan all
vpc 10
no shutdown
exit

vlan 20
name VLAN20
exit

vlan 21
name VLAN21
exit

end
```

### N9K2 Configuration
```
configure terminal

feature vpc
feature lacp
feature hsrp
feature interface-vlan
ip routing

vrf context keepalive

interface Ethernet1/1
description vPC Keepalive Link
no switchport
vrf member keepalive
ip address 10.1.2.2/30
no shutdown
exit

interface Ethernet1/2-3
description vPC Peer-Link
switchport mode trunk
switchport trunk allowed vlan all
channel-group 1 mode active
no shutdown
exit

interface port-channel1
description vPC Peer-Link
switchport mode trunk
switchport trunk allowed vlan all
vpc peer-link
no shutdown
exit

vpc domain 12
peer-keepalive destination 10.1.2.1 source 10.1.2.2 vrf keepalive
role priority 20
exit

interface Ethernet1/4
description vPC to SW11
switchport mode trunk
switchport trunk allowed vlan all
channel-group 10 mode active
no shutdown
exit

interface port-channel10
description vPC to SW11
switchport mode trunk
switchport trunk allowed vlan all
vpc 10
no shutdown
exit

vlan 20
name VLAN20
exit

vlan 21
name VLAN21
exit

end
```

### SW11 Configuration
```
configure terminal

interface range GigabitEthernet0/0-1
description Port-Channel to N9K1 and N9K2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
channel-group 10 mode active
no shutdown
exit

interface port-channel10
description Uplink to Nexus Distribution
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit

vlan 20
name VLAN20
exit

vlan 21
name VLAN21
exit

interface range GigabitEthernet0/2-24
switchport mode access
spanning-tree portfast edge
no shutdown
exit

interface range GigabitEthernet0/2-12
switchport access vlan 20
exit

interface range GigabitEthernet0/13-24
switchport access vlan 21
exit

end
```

## Verification Commands

### vPC Status
```
show vpc
show vpc brief
show vpc consistency-parameters global
show vpc role
show vpc peer-keepalive
```

### Port-Channel Status
```
show port-channel summary
show interface port-channel 1
show interface port-channel 10
show lacp neighbor
```

### VLAN Information
```
show vlan brief
show vlan id 20
show vlan id 21
```

### Interface Status
```
show interface status
show interface trunk
show interface Ethernet1/1
show interface Ethernet1/4
```

### Keepalive Connectivity
```
ping 10.1.2.2 vrf peer-keepalive   ! From N9K1
ping 10.1.2.1 vrf peer-keepalive   ! From N9K2
show ip route vrf peer-keepalive
```

### HSRP Status (if configured)
```
show hsrp brief
show hsrp
```

## Configuration Summary

| Device | Role | Keepalive IP | vPC Role | vPC Priority |
|--------|------|--------------|----------|--------------|
| N9K1   | Distribution | 10.1.2.1/30 | Primary | 10 |
| N9K2   | Distribution | 10.1.2.2/30 | Secondary | 20 |
| SW11   | Access | N/A | Member | N/A |

| Port-Channel | Purpose | Members | vPC ID |
|--------------|---------|---------|--------|
| Po1 | Peer-Link | Eth1/2-3 | peer-link |
| Po10 | vPC to SW11 | Eth1/4 | 10 |

## Notes

- Keepalive Link: Must be in its own VRF to isolate management traffic
- Peer-Link: Carries both control plane (vPC protocol) and data plane traffic
- Role Priority: Lower value = primary; primary switch handles vPC operations
- LACP: Used for dynamic port-channel negotiation (mode active)
- PortFast edge: Only enable on access ports connected to endpoints, never on trunks
- vPC ID: Must match on both vPC peer switches for the same downstream device
- The peer-link automatically becomes a Layer 3 trunk when SVIs are configured
