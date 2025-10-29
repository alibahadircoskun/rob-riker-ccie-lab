# WAN/Internet Edge Configuration
This section of the lab focuses on configuring OSPF at the WAN/Internet edge and mutual redistribution between EIGRP and OSPF. It also covers configuring the Palo Alto firewall to allow internet access for RFC 1918 private addresses.

## Tasks
### WAN/Internet Edge
1. Enable OSPF on Nexus switch
2. Configure CSR15 and N9K7 interfaces facing SW26 and SW27 for OSPF
3. Configure OSPF to WAN and Internet edge devices

### Redistribution
1. Redistribute OSPF and EIGRP mutually on CSR15 and N9K7

### Internet Access
1. Configure FW37 (Palo Alto) to allow internet access for RFC 1918 traffic

## Configuration
### 1. Enable OSPF on Nexus Switch
OSPF is used at the WAN edge to connect to service provider networks.

**N9K7**

```
feature ospf
```

### 2. Configure Interfaces Facing SW26 and SW27 for OSPF
**N9K7**

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

**CSR15**

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

**Verification:**

```
show ip ospf neighbors
show ip ospf interface brief
show ip route ospf
```

### 3. Redistribute EIGRP into OSPF
**N9K7**

```
route-map RM_EIGRP_TO_OSPF permit 10

router ospf 1
 router-id 10.0.0.7
 redistribute eigrp 1 route-map RM_EIGRP_TO_OSPF
```

**CSR15**

```
router ospf 1
 router-id 10.0.0.15
 redistribute eigrp 1 subnets
```

### 4. Redistribute OSPF into EIGRP
**N9K7**

```
route-map RM_OSPF_TO_EIGRP permit 10

router eigrp 1
 router-id 10.0.0.7
 redistribute ospf 1 route-map RM_OSPF_TO_EIGRP
```

**CSR15**

```
router eigrp 1
 redistribute ospf 1 metric 1000000 1 255 1 1500
```

**Verification:**

```
show ip route eigrp
show ip route ospf
show ip eigrp topology
show ip ospf database
```

### 5. Configure Palo Alto Firewall for Internet Access
FW37 (Palo Alto) is configured to allow RFC 1918 private addresses to access the internet using NAT. The firewall connects to the internal network and to the ISP router (CSR16).

**FW37 Configuration (via GUI):**

- **Inside Interface (ethernet1/1)**: 10.26.100.100/24, Zone: in
- **Outside Interface (ethernet1/3)**: 1.16.0.111/24, Zone: out
- **OSPF**: Enabled on ethernet1/1, Router ID 10.0.0.111, Area 0.0.0.0
- **NAT Rule**: Dynamic IP and Port (PAT) from zone "in" to zone "out", translating to ethernet1/3 interface address
- **Security Policy**: Allow all traffic from zone "in" to zone "out"
- **Static Route**: Default route to 1.16.0.16 via ethernet1/3 (ISP router CSR16)

### 6. Configure ISP Router (CSR16)
CSR16 acts as the Internet Service Provider router.

**CSR16**

```
interface GigabitEthernet3
 ip address 1.16.0.16 255.255.255.0
 negotiation auto
```

**Verification:**

```
show ip interface brief
show ip interface GigabitEthernet3
ping 1.16.0.111
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

## Notes
- N9K7 and CSR15 perform mutual redistribution between EIGRP and OSPF.
- Nexus switches use `ip router ospf` command with area specification on interfaces.
- CSR routers use `network` statements under the OSPF process.
- The `subnets` keyword in OSPF redistribution is required to redistribute subnets.
- EIGRP metric parameters (bandwidth, delay, reliability, load, MTU) are required when redistributing into EIGRP on IOS routers.
- Route-maps are used to control redistribution on N9K7.
- FW37 is a Palo Alto firewall configured via GUI that provides internet access using NAT for RFC 1918 private addresses.
- Dynamic PAT translates internal addresses to the outside interface IP address.
- Zones are used to define security boundaries (in=inside, out=outside).
- The security policy explicitly permits all traffic from inside to outside zone.
- OSPF is enabled on the inside interface to learn internal enterprise routes.
- Default route points to ISP router (CSR16) at 1.16.0.16.
- CSR16 acts as the Internet Service Provider router with interface GigabitEthernet3 connecting to FW37.
- This configuration connects the enterprise EIGRP domain with the WAN OSPF domain and provides internet access through the Palo Alto firewall and ISP router.
