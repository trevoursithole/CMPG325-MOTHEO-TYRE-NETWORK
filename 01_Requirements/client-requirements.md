# Client Requirements

## Client

- **Organisation:** Motheo Tyre & Exhaust Centre (Vryburg)
- **Industry:** Automotive
- **Client ID:** CLI-137

## Requirements (from project brief, section 6)

- Assigned addressing block: **192.168.58.0/24**
- Provide appropriate connectivity and network services for the assigned
  scenario.
- Accommodate the stated design constraint and change request.
- Produce a working, testable Packet Tracer implementation.

## Design Constraint (section 8)

> New server (application or file) is planned within six months.

**How this is addressed:** a dedicated VLAN (40 — SERVERS) and subnet
(192.168.58.96/28) are reserved and routed, but left without a live
server device. When the server is procured it can be plugged into
SW-Workshop or a new access switch on this VLAN and given a static
address in the range, with no re-addressing of the rest of the network
required.

## Change Request — CR3 (section 10)

> Guest Wi-Fi must be added for visitors, isolated from internal
> resources.

**How this is addressed:** a separate VLAN (99 — GUEST) and subnet
(192.168.58.112/28) serves an access point (AP-Guest). An extended
access list (`GUEST-ISOLATE`) is applied inbound on the router's guest
sub-interface, denying traffic from the guest subnet to every internal
subnet (MGMT, SALES, WORKSHOP, SERVERS) while permitting everything
else. See `docs/network-design.md` and `docs/testing-evidence.md` for
the design and verification.

## Assigned Networking Challenge (section 9)

> Network Troubleshooting (fault isolation scenario) — Intermediate
> difficulty.

**How this is addressed:** during implementation, two genuine faults
were encountered and resolved using CLI-based fault-isolation
techniques (CDP, `show interfaces trunk`, `show mac-address-table`,
`show interfaces status`, ping testing). Full write-up in
`docs/troubleshooting-log.md`.
