# HQ Catalyst Distribution/Access Configuration

## Topology Overview
- **Distribution Layer**: SW21, SW22 (Catalyst switches)
- **Access Layer**: Multiple access switches
- **VTP Domain**: enterprise
- **VLANs**: Odd and even VLANs

## Configuration Tasks

### 1. Configure VTP Domain

**SW21 and SW22 (Distribution Switches)**
```
configure terminal
vtp mode server
vtp domain enterprise
vtp version 2
end
```

**Access Layer Switches**
```
configure terminal
vtp mode client
vtp domain enterprise
vtp version 2
end
```

### 2. Configure Dynamic Trunking on Access Switches

Access layer switches use DTP (Dynamic Trunking Protocol) to negotiate IEEE 802.1Q trunking.

**Access Layer Switches (Uplink Interfaces)**
```
configure terminal
interface range g0/0-1
switchport mode dynamic desirable
switchport trunk encapsulation dot1q
end
```

### 3. Access Layer VLAN Configuration

Access layer switches should NOT be configured with VLANs directly - they learn VLANs via VTP from the distribution switches.

**Verification Only**
```
show vtp status
show vlan brief
```

### 4. Configure PortFast for Endpoint Interfaces

Any endpoints connected to access switches should transition to forwarding immediately.

**Access Layer Switches (Endpoint Interfaces)**
```
configure terminal
interface GigabitEthernet0/2
switchport access vlan 10
switchport mode access
spanning-tree portfast edge
no shutdown
exit

interface GigabitEthernet0/3
switchport access vlan 11
switchport mode access
spanning-tree portfast edge
no shutdown
exit

! Repeat for all endpoint interfaces
end
```

**Trunk Interfaces to Distribution Layer**
```
configure terminal
interface range GigabitEthernet1/0-3
switchport trunk encapsulation dot1q
switchport mode trunk
no shutdown
end
```

### 5. Configure SW21 as Root for Odd VLANs

SW21 should have the lowest priority for all odd-numbered VLANs.

**SW21 (Using Root Primary Command)**
```
configure terminal
spanning-tree vlan 11 root primary
end
```

This command automatically sets the priority to a value lower than the current root bridge (typically 24576 or 4096).

### 6. Configure SW22 as Root for Even VLANs

SW22 should have the lowest priority for all even-numbered VLANs.

**SW22 (Using Root Primary Command)**
```
configure terminal
spanning-tree vlan 10 root primary
end
```

This command automatically sets the priority to a value lower than the current root bridge.

### 7. Configure Secondary Root Bridges

Each switch should be the secondary (backup) root for the other switch's VLANs.

**SW21 (Secondary for Even VLANs)**
```
configure terminal
spanning-tree vlan 10 root secondary
end
```

This command automatically sets the priority to 28672.

**SW22 (Secondary for Odd VLANs)**
```
configure terminal
spanning-tree vlan 11 root secondary
end
```

### 8. Configure SW21 as Primary Egress for Odd VLANs

Use HSRP (Hot Standby Router Protocol) for gateway redundancy. SW21 should be active for odd VLANs.

**SW21**
```
configure terminal
interface vlan 11
ip address 10.1.11.21 255.255.255.0
standby 11 ip 10.1.11.1
standby 11 priority 150
standby 11 preempt
exit

! Repeat for all odd VLANs
end
```

### 9. Configure SW22 as Primary Egress for Even VLANs

SW22 should be active HSRP router for even VLANs.

**SW22**
```
configure terminal
interface vlan 10
ip address 10.1.10.22 255.255.255.0
standby 10 ip 10.1.10.1
standby 10 priority 150
standby 10 preempt
exit

! Repeat for all even VLANs
end
```

### 10. Configure Backup HSRP on Both Switches

Each switch serves as backup for the other's VLANs with lower priority (default 100).

**SW22 (Backup for Odd VLANs)**
```
configure terminal
interface vlan 11
ip address 10.1.11.22 255.255.255.0
standby 11 ip 10.1.11.1
standby 11 preempt
exit

! Repeat for all odd VLANs
end
```

**SW21 (Backup for Even VLANs)**
```
configure terminal
interface vlan 10
ip address 10.1.10.21 255.255.255.0
standby 10 ip 10.1.10.1
standby 10 preempt
exit

! Repeat for all even VLANs
end
```

## Verification Commands

```
! VTP Status
show vtp status
show vtp counters

! VLAN Information
show vlan brief

! Spanning Tree
show spanning-tree summary
show spanning-tree root
show spanning-tree vlan 1
show spanning-tree vlan 2

! HSRP Status
show standby brief
show standby vlan 1
show standby vlan 2

! Interface Status
show interface trunk
show interface status

! PortFast Configuration
show spanning-tree interface g0/2 detail
```

## Configuration Summary

| Switch | Role | VLAN 10 (Even) STP | VLAN 11 (Odd) STP | VLAN 10 HSRP | VLAN 11 HSRP |
|--------|------|-------------------|-------------------|--------------|--------------|
| SW21   | Distribution | Secondary (28672) | Primary (24576) | Standby (100) | Active (150) |
| SW22   | Distribution | Primary (24576) | Secondary (28672) | Active (150) | Standby (100) |

**STP Configuration Method:**
- Used `spanning-tree vlan X root primary` command (sets priority to 24576)
- Used `spanning-tree vlan X root secondary` command (sets priority to 28672)

**HSRP Configuration:**
- VLAN 10 Gateway: 10.1.10.1 (SW22 active with priority 150)
- VLAN 11 Gateway: 10.1.11.1 (SW21 active with priority 150)

## Notes

- VTP version 2 is recommended for better VLAN database management
- `spanning-tree vlan X root primary` automatically sets priority to 24576 (or lower if another switch has priority below 24576)
- `spanning-tree vlan X root secondary` automatically sets priority to 28672
- Manual priority configuration uses values in increments of 4096 (0, 4096, 8192, etc.)
- HSRP priority 150 makes the switch active, default 100 makes it standby
- PortFast edge should only be enabled on access ports connected to endpoints, never on trunk ports
- DTP (Dynamic Trunking Protocol) uses "dynamic desirable" mode for automatic trunk negotiation
- Trunk interfaces use `switchport trunk encapsulation dot1q` before setting mode to trunk on older Catalyst switches
- The `root primary` and `root secondary` commands are easier to use and automatically adjust priorities based on the current topology
