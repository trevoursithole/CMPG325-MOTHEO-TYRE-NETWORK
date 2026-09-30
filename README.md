# CMPG 325 — Individual Semester Project
## Motheo Tyre & Exhaust Centre (Vryburg)

| Field | Value |
|---|---|
| Student | Sithole, T |
| Student Number | 45485097 |
| Project ID | CMPG325-2026-137 |
| Client ID | CLI-137 |
| Assigned Organisation | Motheo Tyre & Exhaust Centre (Vryburg) |
| Industry | Automotive |
| Assigned Networking Challenge | Network Troubleshooting (fault isolation scenario) — Intermediate |
| Addressing Block | 192.168.58.0/24 |
| Design Constraint | New server (application or file) planned within six months |
| Change Request | CR3: Guest Wi-Fi must be added for visitors, isolated from internal resources |

## Project Overview

This project designs and simulates a small automotive-shop network for
Motheo Tyre & Exhaust Centre in Cisco Packet Tracer. The network uses a
**router-on-a-stick** topology: a single Cisco 2911 router (Router1) with
VLAN sub-interfaces, trunked down through a core switch (SW-Core) to two
access switches (SW-Workshop, SW-Office) and a wireless access point
(AP-Guest).

Five VLANs are implemented over the assigned 192.168.58.0/24 block using
VLSM, giving the shop separate broadcast domains for management, sales,
workshop devices, a future server, and isolated guest Wi-Fi (satisfying
CR3). See `docs/ip-addressing-plan.md` for the full breakdown.

During implementation and testing, **two real faults** were found and
resolved using systematic fault-isolation methodology (not staged faults):
a trunk/native-VLAN mismatch between two switches, and a set of
connectivity failures traced to a missing physical link and a wireless
SSID mismatch. Both are documented in full in
`docs/troubleshooting-log.md`, satisfying the assigned Network
Troubleshooting challenge (section 9 of the project brief).

## Topology

See `screenshots/topology/` for the full logical topology. In summary:

```
                    Router1 (2911)
                        |
                    SW-Core (2960)
              /         |          \
      SW-Workshop    SW-Office    AP-Guest
      /        \     /  |  |  \       |
LAP-Diagnostic  |  Recep Mgr Sales1 Sales2  GuestLaptop1
           PC-Jobcard        Printer         GuestPhone1
```

## VLAN Summary

| VLAN | Name | Subnet | Gateway | Purpose |
|---|---|---|---|---|
| 10 | MGMT | 192.168.58.0/27 | .1 | Management |
| 20 | SALES | 192.168.58.32/27 | .33 | Sales / office |
| 30 | WORKSHOP | 192.168.58.64/27 | .65 | Workshop / diagnostic devices |
| 40 | SERVERS | 192.168.58.96/28 | .97 | Reserved for planned server (design constraint) |
| 99 | GUEST | 192.168.58.112/28 | .113 | Guest Wi-Fi, isolated (CR3) |

Full VLSM working is in `docs/ip-addressing-plan.md`.

## Repository Structure

```
├── README.md
├── docs/
│   ├── client-requirements.md
│   ├── network-design.md
│   ├── ip-addressing-plan.md
│   ├── troubleshooting-log.md
│   └── testing-evidence.md
├── configs/
│   ├── R1-running-config.txt
│   ├── SW-Core-running-config.txt
│   └── SW-Workshop-running-config.txt
├── screenshots/
│   ├── topology/
│   ├── router/
│   ├── switches/
│   ├── testing/
│   ├── troubleshooting-trunk/
│   ├── troubleshooting-cabling/
│   └── troubleshooting-guest-wifi/
├── packet-tracer/
│   └── PROJECT.pkt
└── reflection.md
```

## How to Open

1. Open `packet-tracer/PROJECT.pkt` in Cisco Packet Tracer.
2. All device configurations are saved to startup-config.
3. Suggested verification commands are listed in `docs/testing-evidence.md`.

## Video Demonstration

[Link to be added]

## Academic Integrity

AI assistance  was used during this project for design guidance,
configuration review, and documentation drafting, in line with the CMPG
325 brief and the NWU AI Policy. All configuration was entered, tested,
and verified on the actual devices by the student, including diagnosis
of two real faults that emerged during implementation. The student
remains responsible for the correctness, understanding, and academic
integrity of everything submitted.
