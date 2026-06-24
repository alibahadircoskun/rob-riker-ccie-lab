# Rob Riker CCIE Enterprise Lab Notes

Structured notes and configuration walkthroughs from working through Rob Riker's large CCIE-style enterprise lab. This repo focuses on the parts I documented in enough detail to be reusable: campus switching, Nexus/vPC design, VXLAN, routed core services, and WAN edge behavior.

## What this proves

- Work through large multi-domain networking labs without losing structure
- Document campus, data center, WAN, and overlay technologies in a way someone else can follow
- Move between Catalyst, Nexus, CSR, IOS XRv, firewall, and service-provider style concepts in one topology
- Break a very large CCIE lab into smaller study sections instead of keeping it as one unreadable blob

## Topics covered

- Campus Catalyst distribution and access design
- Nexus distribution with Catalyst access uplinks
- Back-to-back vPC data center design
- VXLAN/EVPN fabric concepts at HQ and colo sites
- Routed core with EIGRP, OSPF, BGP, and redistribution
- WAN edge and upstream connectivity patterns
- MPLS, DMVPN, firewall, and multi-ISP concepts in the wider topology

## Repository layout

- `docs/01-HQ-Catalyst-Distribution-Access.md`
- `docs/02-Nexus-Distribution-Catalyst-Access.md`
- `docs/03-Data-Center-Back-to-Back-vPC.md`
- `docs/04-VXLAN-HQ-Data-Center.md`
- `docs/05-Routed-Core-Config.md`
- `docs/06-Wan-Edge-Config.md`

## Lab design summary

The overall topology includes:

- HQ campus switching
- Nexus-based data center segments
- A routed core joining campus and WAN functions
- Multiple ISP-style upstream sections
- MPLS and DMVPN components
- VXLAN overlays for data center extension
