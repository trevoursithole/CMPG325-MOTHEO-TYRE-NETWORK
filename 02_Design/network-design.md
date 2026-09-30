# Network Design

## Topology Choice: Router-on-a-Stick

A single router (Router1, Cisco 2911) is used with one physical trunk
link to SW-Core, carrying all VLANs via 802.1Q sub-interfaces
(`GigabitEthernet0/0.10`, `.20`, `.30`, `.40`, `.99`). This was chosen
over a multi-router design because:

- The client is a single small site with modest traffic volumes —
  a single router provides more than enough throughput.
- It keeps the topology simple to build, configure, and explain,
  which matters for both the video demonstration and long-term
  maintainability by the client's future IT support.
- It is the standard, expected way to demonstrate inter-VLAN routing
  in a Packet Tracer project of this scope.

## Devices

| Device | Role |
|---|---|
| Router1 (Cisco 2911) | Router-on-a-stick; inter-VLAN routing, DHCP, guest ACL |
| SW-Core (2960-24TT) | Core/distribution switch; trunks to Router1, SW-Workshop, SW-Office, AP-Guest |
| SW-Workshop (2960-24TT) | Access switch for workshop devices (VLAN 30) |
| SW-Office (2960-24TT) | Access switch for office/management/sales devices (VLANs 10, 20) |
| AP-Guest | Wireless access point for Guest Wi-Fi (VLAN 99), SSID `Motheo-Guest` |
| PC-Manager | VLAN 10 (MGMT) |
| PC-Reception | VLAN 10 (MGMT) |
| PC-Sales1, PC-Sales2 | VLAN 20 (SALES) |
| PRN-Office | VLAN 20 (SALES) |
| LAP-Diagnostic, PC-Jobcard | VLAN 30 (WORKSHOP) |
| GuestLaptop1, GuestPhone1 | VLAN 99 (GUEST), via AP-Guest |

## VLAN and Trunking

All inter-switch links (Router1↔SW-Core, SW-Core↔SW-Workshop,
SW-Core↔SW-Office) are configured as 802.1Q trunks carrying VLANs
10, 20, 30, 40 and 99, with native VLAN 1 consistent across both ends
of every trunk. End-device ports are configured as access ports in
their respective VLAN.

## Guest Wi-Fi and CR3 Isolation

CR3 requires guest Wi-Fi to be isolated from internal resources. This
is implemented with:

1. **A separate VLAN and subnet** for guest traffic (VLAN 99,
   192.168.58.112/28), so guest and internal traffic are never in the
   same broadcast domain.
2. **An extended ACL (`GUEST-ISOLATE`)** applied inbound on the
   router's guest sub-interface (`Gi0/0.99`):

   ```
   ip access-list extended GUEST-ISOLATE
    deny ip 192.168.58.112 0.0.0.15 192.168.58.0   0.0.0.31   ! deny -> MGMT
    deny ip 192.168.58.112 0.0.0.15 192.168.58.32  0.0.0.31   ! deny -> SALES
    deny ip 192.168.58.112 0.0.0.15 192.168.58.64  0.0.0.31   ! deny -> WORKSHOP
    deny ip 192.168.58.112 0.0.0.15 192.168.58.96  0.0.0.15   ! deny -> SERVERS
    permit ip any any                                          ! allow everything else
   ```

   Applying the ACL **inbound on the guest sub-interface** means it is
   evaluated the moment guest traffic enters the router, before any
   routing decision is made toward an internal subnet — the most
   efficient and correct placement for this kind of isolation.

3. **A note on scope:** the ACL is stateless. It blocks guest-sourced
   traffic to internal subnets, but by the same logic, internal hosts
   cannot receive return traffic from guests either — full isolation,
   in both directions, which is the intended behaviour for CR3.

This design was verified by testing (see `docs/testing-evidence.md`):
guest devices can reach the router itself and (in a fuller topology)
the internet, but pings from guest devices to MGMT, SALES, WORKSHOP or
SERVERS subnets are correctly blocked, with the router replying
"Destination host unreachable" — proof the ACL is actively enforcing
the deny rule rather than the traffic simply failing to route.

## Server VLAN (Design Constraint)

VLAN 40 (SERVERS, 192.168.58.96/28) is fully configured and routed but
intentionally has no DHCP pool and no live device. This satisfies the
brief's design constraint that a new server is planned within six
months, without requiring any redesign of the addressing scheme when
that server is deployed — it will simply be given a static IP inside
192.168.58.97–.110 and connected to an access port set to VLAN 40.

## Out of Scope

No WAN/internet-facing interface or ISP router is included in this
topology. Router1's unused physical interfaces (`Gi0/1`, `Gi0/2`) are
shut down. Internet access for guest and staff devices is outside the
scope of this assignment brief and addressing block.
