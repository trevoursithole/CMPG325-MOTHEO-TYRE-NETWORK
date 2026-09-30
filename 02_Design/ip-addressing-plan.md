# IP Addressing Plan (VLSM)

**Assigned block:** 192.168.58.0/24

The block is subnetted with VLSM to give each VLAN a size matched to its
expected number of devices, while leaving room free for future growth.

| VLAN | Name | Subnet | Mask | Usable range | Gateway | Broadcast | Hosts |
|---|---|---|---|---|---|---|---|
| 10 | MGMT | 192.168.58.0/27 | 255.255.255.224 | .1 – .30 | 192.168.58.1 | .31 | 30 |
| 20 | SALES | 192.168.58.32/27 | 255.255.255.224 | .33 – .62 | 192.168.58.33 | .63 | 30 |
| 30 | WORKSHOP | 192.168.58.64/27 | 255.255.255.224 | .65 – .94 | 192.168.58.65 | .95 | 30 |
| 40 | SERVERS | 192.168.58.96/28 | 255.255.255.240 | .97 – .110 | 192.168.58.97 | .111 | 14 |
| 99 | GUEST | 192.168.58.112/28 | 255.255.255.240 | .113 – .126 | 192.168.58.113 | .127 | 14 |
| — | Reserved / future expansion | 192.168.58.128/25 | 255.255.255.128 | .129 – .254 | — | .255 | 126 |

## DHCP

Dynamic pools are configured on Router1 for MGMT, SALES, WORKSHOP and
GUEST. Gateway addresses are excluded from each pool so they are never
handed out to a client.

| Pool | Network | Default Router | Excluded |
|---|---|---|---|
| MGMT | 192.168.58.0/27 | 192.168.58.1 | .1–.5 |
| SALES | 192.168.58.32/27 | 192.168.58.33 | .33–.35 |
| WORKSHOP | 192.168.58.64/27 | 192.168.58.65 | .65–.66 |
| GUEST | 192.168.58.112/28 | 192.168.58.113 | .113 |

**SERVERS (VLAN 40)** has no DHCP pool. Servers are expected to use
static addressing, consistent with normal practice and with the design
constraint that this VLAN is provisioned ahead of a server actually
being deployed.

## Design Rationale

- **VLSM sizing:** MGMT, SALES and WORKSHOP use /27s (30 usable hosts
  each) since they carry end-user devices. SERVERS and GUEST use /28s
  (14 usable hosts each) since they carry fewer, more predictable
  devices.
- **Large reserved block:** 192.168.58.128/25 is left completely unused,
  giving room for additional VLANs (e.g. a second office location, VoIP,
  CCTV) without redesigning the existing addressing.
- **Guest subnet isolation:** placing GUEST in its own subnet is what
  makes the CR3 access-list enforceable — see `docs/network-design.md`.
