# Motheo Tyre & Exhaust Centre — Network Design Project

**Module:** CMPG 325 — Computer Networks (NWU)
**Project ID:** CMPG325-2026-137 | **Client ID:** CLI-137
**Student:** Sithole, T (45485097)

## Project brief

Individual semester project: analyse, design, simulate, configure and demonstrate a
computer network for **Motheo Tyre & Exhaust Centre**, an automotive workshop in Vryburg.

- **Assigned addressing block:** `192.168.58.0/24`
- **Assigned technical challenge:** Network troubleshooting — fault isolation (Intermediate)
- **Change request (CR3):** Guest Wi-Fi, isolated from all internal resources
- **Design constraint:** New application/file server planned within 6 months

## Repository structure

| Folder | Contents |
|---|---|
| [`01-requirements/`](./01-requirements) | Client requirements analysis |
| [`02-design/`](./02-design) | Physical & logical topology, VLAN plan, IP addressing table |
| [`03-packet-tracer/`](./03-packet-tracer) | Final `.pkt` file and build screenshots |
| [`04-configuration/`](./04-configuration) | Exported running-configs (router, all switches) |
| [`05-testing/`](./05-testing) | Ping/DHCP verification evidence |
| [`06-troubleshooting/`](./06-troubleshooting) | Fault-isolation scenario: before/after evidence and explanation |
| [`07-video/`](./07-video) | Link to the 15–20 minute demonstration video |

## Network summary

| VLAN | Name | Subnet | Gateway |
|---|---|---|---|
| 10 | MGMT | 192.168.58.0/27 | .1 |
| 20 | SALES | 192.168.58.32/27 | .33 |
| 30 | WORKSHOP | 192.168.58.64/27 | .65 |
| 40 | SERVERS (reserved) | 192.168.58.96/28 | .97 |
| 99 | GUEST (isolated) | 192.168.58.112/28 | .113 |

Full design justification is in [`02-design/network-design-documentation.md`](./02-design/network-design-documentation.md).

## Key design decisions

- **Router-on-a-stick**, one physical uplink, five sub-interfaces — appropriate for the
  device count on this network, avoids the cost/complexity of a Layer-3 switch.
- **VLSM addressing** sized to real device counts rather than equal splits, keeping the
  upper half of the /24 free for future growth.
- **VLAN 40 reserved but not built** — satisfies the six-month server constraint without
  inventing scope that wasn't in the brief.
- **ACL on the guest sub-interface** blocks the four internal subnets while still
  permitting outbound traffic — satisfies CR3 with a single access-list.

## Assigned technical challenge

A fault was deliberately introduced (VLAN 30 removed from the core switch's uplink trunk),
then isolated using a layered method — physical link check, VLAN/trunk check, then
connectivity re-test — and corrected. Full evidence and command sequence in
[`06-troubleshooting/`](./06-troubleshooting).

## Status

- [x] Client requirements analysed
- [x] Network design complete (topology + IP addressing)
- [ ] Packet Tracer implementation
- [ ] Assigned technical challenge configured and verified
- [ ] Testing evidence captured
- [ ] Video demonstration recorded
- [ ] Final submission

## Academic integrity

This is individual work for CMPG325-2026-137. AI assistance was used for planning and
documentation support; all configuration, testing, verification and understanding is my
own, per the NWU AI Policy referenced in the project brief.
