## Enterprise Network Design — 3-Site WAN with OSPF

> **Status:** Work in progress — built as a learning project by an undergraduate IT/Cybersecurity student (not yet graduated). Feedback and suggestions are welcome.

A Cisco Packet Tracer network design simulating a company with a headquarters and two branch offices, connected over WAN links with dynamic OSPF routing, DHCP, and a hosted web server.

## Topology

3 sites, 3 routers, 4 switches, 35 PCs, and 1 web server:

- **HQ**: 2 departments (IT, Sales) on separate switches, plus a web server
- **Branch 1**: Sales department
- **Branch 2**: Support department

See `screenshots/01-topology-full.png` for the full diagram.

## What it demonstrates

- **IP addressing & subnetting**: /27 subnets per department segment, /30 point-to-point subnets for WAN links
- **DHCP**: automatic IP assignment per site (router-based DHCP pools), with static IPs reserved for routers and the server
- **WAN connectivity**: Serial point-to-point links between HQ and each branch
- **Dynamic routing (OSPF)**: all three routers exchange routes automatically — no static routes

## IP Addressing Plan

| Site | Segment | Subnet | Gateway |
|---|---|---|---|
| HQ | IT + Server | `192.168.1.0/27` | `192.168.1.1` |
| HQ | Sales | `192.168.1.32/27` | `192.168.1.33` |
| Branch 1 | Sales | `192.168.2.0/27` | `192.168.2.1` |
| Branch 2 | Support | `192.168.3.0/27` | `192.168.3.1` |
| WAN | HQ ↔ Branch1 | `10.0.0.0/30` | - |
| WAN | HQ ↔ Branch2 | `10.0.0.4/30` | - |

## OSPF Configuration & Verification

- `screenshots/02-hq-ospf-config.png` — HQ: Serial interfaces brought up, OSPF configured with all 4 connected networks
- `screenshots/03-branch1-ospf-adjacency.png` — Branch1: OSPF adjacency reaching FULL state with HQ
- `screenshots/04-branch2-ospf-neighbor-full.png` — Branch2: `show ip ospf neighbor` confirming FULL adjacency with HQ

## Troubleshooting log

| Issue | Cause | Fix |
|---|---|---|
| `%Invalid interface type and number` on `interface fastEthernet 0/0` | Router model uses `GigabitEthernet`, not `FastEthernet` | Checked actual interface names with `show ip interface brief` first |
| `% Invalid input detected` on `do show ...` / `do write memory` | `do` only works inside `(config)#` mode, not at the base `Router#` prompt | Used the command directly (without `do`) when at the base prompt |
| DHCP pool configured on the wrong router | Relied on window titles instead of verifying the actual IP | Always confirmed the router's identity via `show ip interface brief` before any change |
| All router config (IP + DHCP) wiped after adding a WIC-2T serial card | Physical hardware change without saving config first | Re-applied config and ran `write memory` after every major step from then on |
| Wrong card installed initially (Ethernet module instead of `WIC-2T`) | Similar-looking hardware module names in Packet Tracer | Visually confirmed the Serial port icon before installing |

## Files

- `network-project.pkt` — the Packet Tracer project file
- `screenshots/` — topology and OSPF configuration/verification evidence

## Next steps (v2 ideas)

- Add VLANs within each site for finer segmentation
- Add a second OSPF area for better scalability
- Add basic ACLs to restrict inter-department traffic
- Add a DNS server alongside the web server
