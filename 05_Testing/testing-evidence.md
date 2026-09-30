# Testing Evidence

Per section 11 of the project brief. All screenshots referenced below
are in `screenshots/testing/` unless otherwise noted.

## 1. DHCP / Addressing

| Test | Device | Expected | Result | Evidence |
|---|---|---|---|---|
| `ipconfig` | PC-Manager | Address in 192.168.58.1–.30 (MGMT), gateway .1 | Got 192.168.58.6/27, gateway .1 | `pc-manager-ipconfig-mgmt.png` |
| `ipconfig` | LAP-Diagnostic | Address in 192.168.58.65–.94 (WORKSHOP), gateway .65 | Got 192.168.58.67/27, gateway .65 (after fault fix) | `../troubleshooting-cabling/` |
| `ipconfig` | PC-Jobcard | Address in 192.168.58.65–.94 (WORKSHOP), gateway .65 | Got 192.168.58.68/27, gateway .65 (after fault fix) | `../troubleshooting-cabling/` |
| `ipconfig` | GuestPhone1 | Address in 192.168.58.113–.126 (GUEST), gateway .113 | Got 192.168.58.114/28, gateway .113 (after fault fix) | `../troubleshooting-guest-wifi/` |

## 2. Internal Connectivity

| Test | From | To | Expected | Result | Evidence |
|---|---|---|---|---|---|
| Ping | PC-Sales2 (SALES) | 192.168.58.97 (SERVERS gateway) | Success | 4/4 replies, 0% loss | `pc-sales2-ping-server-success.png` |
| Ping | LAP-Diagnostic (WORKSHOP) | 192.168.58.65 (own gateway) | Success | 4/4 replies, 0% loss | `lap-diagnostic-ping-gateway-success.png` |
| Ping | PC-Jobcard (WORKSHOP) | 192.168.58.65 (own gateway) | Success | 4/4 replies, 0% loss | `pc-jobcard-ping-gateway-success.png` |

These confirm inter-VLAN routing is working correctly for internal
subnets (SALES → SERVERS, and WORKSHOP devices reaching their own
gateway after the cabling fault was resolved).

## 3. Guest Isolation (CR3)

| Test | From | To | Expected | Result | Evidence |
|---|---|---|---|---|---|
| Ping | GuestLaptop1 (GUEST) | 192.168.58.97 (SERVERS) | **Fail** (isolation) | 4/4 timed out | `guestlaptop1-ping-server-fail-isolation.png` |
| Ping | GuestPhone1 (GUEST) | 192.168.58.97 (SERVERS) | **Fail** (isolation) | 4/4 "Destination host unreachable" from 192.168.58.113 | `guestphone1-ping-server-fail-isolation.png` |
| `show access-lists` | Router1 | — | ACL `GUEST-ISOLATE` present and applied | Confirmed, all four deny statements + permit any any present | `router1-show-access-lists.png` |

The GuestPhone1 result is the stronger piece of evidence: the router
itself actively replies "Destination host unreachable," proving the
ACL is being evaluated and enforced, not just that the packet
disappeared.

**A failed ping from a guest device to an internal subnet is a PASS,
not a bug** — it is the intended behaviour required by CR3.

## 4. Fault-Isolation Evidence

See `docs/troubleshooting-log.md` for the full write-up. Screenshot
folders:

- `screenshots/troubleshooting-trunk/` — native VLAN mismatch (Fault 1)
- `screenshots/troubleshooting-cabling/` — missing physical link (Fault 2)
- `screenshots/troubleshooting-guest-wifi/` — SSID mismatch (Fault 3)

## 5. Configuration Persistence

`copy running-config startup-config` was run and confirmed with `[OK]`
on:

- Router1 — `screenshots/router/r1-saved-startup-config.png`
- SW-Core — `screenshots/switches/sw-core-final-mac-table-and-save.png`
- SW-Workshop — `screenshots/switches/sw-workshop-final-mac-table-and-save.png`

**Outstanding before final submission:** confirm SW-Office's config is
also saved (`copy running-config startup-config` on that device), and
do a final close/reopen test of the .pkt file to confirm it reproduces
the working solution without any manual intervention, per the brief's
testing requirement.
