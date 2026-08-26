# Motheo Tyre & Exhaust Centre (Vryburg) — Network Design Documentation
**Project ID:** CMPG325-2026-137 | **Client ID:** CLI-137 | **Industry:** Automotive

---

## 1. Client & Project Overview

Motheo Tyre & Exhaust Centre is a small automotive workshop in Vryburg offering tyre and
exhaust services. The business needs a segmented, secure internal network covering the
front office, sales/parts counter and workshop, with guest Wi-Fi for waiting customers and
headroom to add an application/file server within six months.

**Assigned addressing block:** `192.168.58.0/24`
**Assigned technical challenge:** Network troubleshooting — fault isolation scenario (Intermediate)
**Change request CR3:** Guest Wi-Fi, isolated from internal resources
**Design constraint:** New application/file server planned within 6 months

---

## 2. Requirements Analysis

| Area | Need | Driver |
|---|---|---|
| Front office / management | Admin PC, invoicing, printer | Day-to-day business admin |
| Sales / parts counter | 2 PCs, receipt/label printer | Customer transactions, parts lookup |
| Workshop | Diagnostic laptop, job-card PC | Technicians need network access without touching office data |
| Guest | Wi-Fi for waiting customers | CR3 — must not reach internal resources |
| Future | Application/file server | Constraint — capacity must exist without redesign |
| Security | Segmentation between departments | Prevent workshop/guest traffic from reaching office data |
| Support | Documented fault-isolation method | Assigned technical challenge |

**Departmental separation is the core design driver** — office, sales and workshop are
functionally different, and the client explicitly requires guest isolation, so VLANs are the
right tool rather than a flat network.

---

## 3. Device Inventory

| Qty | Device | VLAN | Location |
|---|---|---|---|
| 1 | Router1 (router-on-a-stick) | — | Edge |
| 1 | Core switch (L2, trunking) | — | Comms cabinet |
| 1 | Office switch | 10, 20 | Front office |
| 1 | Workshop switch | 30 | Workshop |
| 1 | Wireless access point | 99 | Waiting area |
| 1 | Manager PC | 10 | Office |
| 1 | Reception/Admin PC | 10 | Office |
| 1 | Shared network printer | 10 | Office |
| 2 | Sales/parts counter PCs | 20 | Front counter |
| 1 | Workshop diagnostic laptop | 30 | Workshop bay |
| 1 | Job-card lookup PC | 30 | Workshop |
| 2 | Guest test devices (laptop/phone) | 99 | Waiting area |
| — | *Reserved: future server* | 40 | *Not built — see §7* |

This gives a realistic ~10-endpoint small-business network — enough to demonstrate
inter-VLAN routing, DHCP, guest isolation and a genuine fault-isolation scenario without
being unmanageably large in Packet Tracer.

---

## 4. VLAN Plan

| VLAN | Name | Purpose |
|---|---|---|
| 10 | MGMT | Office/admin |
| 20 | SALES | Front desk & parts counter |
| 30 | WORKSHOP | Technician devices |
| 40 | SERVERS | Reserved for future server |
| 99 | GUEST | Isolated guest Wi-Fi |

VLAN 1 (default) carries no data traffic — standard hardening practice, worth noting
explicitly in your video defence.

---

## 5. IP Addressing Plan (VLSM on 192.168.58.0/24)

| VLAN | Subnet | Mask | Usable range | Gateway | Static-reserved | DHCP pool |
|---|---|---|---|---|---|---|
| 10 MGMT | 192.168.58.0/27 | 255.255.255.224 | .1–.30 | .1 | .2–.5 (printer) | .6–.30 |
| 20 SALES | 192.168.58.32/27 | 255.255.255.224 | .33–.62 | .33 | .34–.35 | .36–.62 |
| 30 WORKSHOP | 192.168.58.64/27 | 255.255.255.224 | .65–.94 | .65 | .66 | .67–.94 |
| 40 SERVERS | 192.168.58.96/28 | 255.255.255.240 | .97–.110 | .97 | .98 (reserved for server) | n/a |
| 99 GUEST | 192.168.58.112/28 | 255.255.255.240 | .113–.126 | .113 | — | .114–.126 |
| Unallocated | 192.168.58.128/25 | — | — | — | — | growth reserve |

**Justification:** VLSM avoids wasting the /24 on five equal /27s when GUEST and SERVERS
need far fewer hosts than MGMT/SALES/WORKSHOP. Sizes are based on actual device counts
plus realistic headroom, not arbitrary splits — this is the kind of justification the
rubric rewards under "IP addressing is efficient, correct and fully documented."

---

## 6. Configuration Plan

### 6.1 Router1 — sub-interfaces (router-on-a-stick)
```
interface g0/0
 no shutdown
!
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.58.1 255.255.255.224
!
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.58.33 255.255.255.224
!
interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.58.65 255.255.255.224
!
interface g0/0.40
 encapsulation dot1Q 40
 ip address 192.168.58.97 255.255.255.240
!
interface g0/0.99
 encapsulation dot1Q 99
 ip address 192.168.58.113 255.255.255.240
```

### 6.2 DHCP pools (on Router1)
```
ip dhcp excluded-address 192.168.58.1 192.168.58.5
ip dhcp excluded-address 192.168.58.33 192.168.58.35
ip dhcp excluded-address 192.168.58.65 192.168.58.66
ip dhcp excluded-address 192.168.58.113 192.168.58.113

ip dhcp pool MGMT
 network 192.168.58.0 255.255.255.224
 default-router 192.168.58.1
!
ip dhcp pool SALES
 network 192.168.58.32 255.255.255.224
 default-router 192.168.58.33
!
ip dhcp pool WORKSHOP
 network 192.168.58.64 255.255.255.224
 default-router 192.168.58.65
!
ip dhcp pool GUEST
 network 192.168.58.112 255.255.255.240
 default-router 192.168.58.113
```

### 6.3 Guest isolation ACL — satisfies CR3
```
ip access-list extended GUEST-ISOLATE
 deny ip 192.168.58.112 0.0.0.15 192.168.58.0 0.0.0.31
 deny ip 192.168.58.112 0.0.0.15 192.168.58.32 0.0.0.31
 deny ip 192.168.58.112 0.0.0.15 192.168.58.64 0.0.0.31
 deny ip 192.168.58.112 0.0.0.15 192.168.58.96 0.0.0.15
 permit ip any any
!
interface g0/0.99
 ip access-group GUEST-ISOLATE in
```
This permits guest devices out to the internet (if simulated) but blocks any traffic aimed
back at MGMT, SALES, WORKSHOP or the reserved SERVERS block — directly demonstrating CR3.

### 6.4 Core switch
```
vlan 10
vlan 20
vlan 30
vlan 40
vlan 99
!
interface range fa0/1-2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
!
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
```

### 6.5 Office & Workshop switches
Office switch: access ports for MGMT devices/printer on VLAN 10, SALES PCs on VLAN 20;
uplink port set to trunk (VLANs 10,20 allowed at minimum). Workshop switch: access ports on
VLAN 30 for the diagnostic laptop and job-card PC; uplink trunk carrying VLAN 30.

### 6.6 Access point
SSID `Motheo-Guest`, WPA2-PSK, mapped to an access port on VLAN 99.

---

## 7. Design Constraint — Future Server

The client's server is **not built now** — the brief says it's planned "within six
months," so inventing one would exceed scope. Instead: VLAN 40 and subnet
`192.168.58.96/28` are fully reserved and documented, with `.97` as the gateway already
configured on the router and `.98` earmarked for the server's static IP. When the server
arrives, it plugs into a trunk port already carrying VLAN 40 — zero redesign required. Make
this reasoning explicit in your report/video; it's exactly what "design decisions are
clearly justified" is graded on.

---

## 8. Assigned Technical Challenge — Network Troubleshooting (Fault Isolation)

**Recommended scenario:** After the network is built and working, the workshop
technician reports they can't reach the shared printer/sales system. Introduce **one**
deliberate fault — pick one and document it clearly:
- Workshop switch access port assigned to the wrong VLAN, **or**
- Core switch trunk's allowed-VLAN list missing VLAN 30, **or**
- Router sub-interface `g0/0.30` has the wrong encapsulation VLAN ID

**Demonstrate a systematic (layered) isolation method**, not guesswork:
1. **Physical layer:** `show interfaces status` — confirm link/line protocol up
2. **Data link layer:** `show vlan brief`, `show interfaces trunk` — confirm correct VLAN and trunk allowed-list
3. **Network layer:** `show ip interface brief` on the router, `ping` from the workshop PC to its gateway, then to another VLAN
4. **Isolate → correct → re-verify:** fix the single misconfigured line, then re-run the same ping/`show` sequence to prove resolution

Capture **before** (failure + diagnostic commands) and **after** (fix + successful ping)
screenshots for GitHub, and narrate this exact sequence in the video — this maps directly
to "verification and testing are complete and accurate" in the rubric.

---

## 9. Testing Plan

| Test | Expected result |
|---|---|
| PC in each VLAN gets DHCP address | Correct subnet, correct gateway |
| Ping between VLAN 10 and VLAN 20 | Success (business-to-business allowed) |
| Ping from GUEST (VLAN 99) to VLAN 10/20/30/40 | **Fails** — ACL blocks it (proves CR3) |
| Ping from GUEST to internet-simulated host | Success (guest still gets connectivity) |
| Workshop PC to shared printer (post-fault-fix) | Success, after fault isolation demo |
| `show ip route` on Router1 | All five connected subnets present |

---

## 10. GitHub Portfolio Structure

```
/README.md                  — project overview, client, how to navigate
/01-requirements/            — this brief's requirements, summarised
/02-design/                  — topology diagrams, VLAN plan, addressing table
/03-packet-tracer/           — final .pkt file + build screenshots
/04-configuration/           — exported running-configs per device (.txt)
/05-testing/                 — ping/tracert evidence, DHCP verification
/06-troubleshooting/         — fault scenario: before/after screenshots + explanation
/07-video/                   — link to the 15–20 min demonstration
```
Commit incrementally against the real milestones (28 Aug design, 2 Oct implementation,
16 Oct final) rather than one bulk commit — the rubric explicitly rewards "frequent
meaningful commits" showing development over time.

---

## 11. Video Demonstration — Script Outline (15–20 min)

1. **Intro** (0:30) — yourself, client, project ID
2. **Requirements & design** (2–3 min) — walk through §2–§5, explain VLSM choices
3. **Packet Tracer build** (4–5 min) — show topology, VLANs, trunks, DHCP working
4. **Assigned challenge demo** (4–5 min) — live fault isolation per §8
5. **CR3 guest Wi-Fi demo** (2 min) — show guest gets internet but is blocked internally
6. **Testing** (2 min) — run through §9's test table live
7. **Reflection** (1 min) — what you'd do differently, what you learned

---
*Document prepared as part of Milestone 1 evidence (client requirements, physical topology,
logical topology, IP addressing plan) for CMPG325-2026-137.*
