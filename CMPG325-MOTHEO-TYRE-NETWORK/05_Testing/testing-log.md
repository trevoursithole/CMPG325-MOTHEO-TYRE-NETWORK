# Testing and Verification Evidence

Fill in each row as you complete the corresponding test from Phase 8-9 of the build guide.
Attach screenshot filenames alongside each result.

| # | Test | Expected result | Actual result | Screenshot | Pass/Fail |
|---|---|---|---|---|---|
| 1 | PC-Manager obtains DHCP address (VLAN 10) | 192.168.58.6-30 range, gateway .1 | | | |
| 2 | PC-Sales1 obtains DHCP address (VLAN 20) | 192.168.58.36-62 range, gateway .33 | | | |
| 3 | LAP-Diagnostic obtains DHCP address (VLAN 30) | 192.168.58.67-94 range, gateway .65 | | | |
| 4 | GuestLaptop1 obtains DHCP address (VLAN 99) | 192.168.58.114-126 range, gateway .113 | | | |
| 5 | PRN-Office reachable at static 192.168.58.2 | Ping succeeds from VLAN 10 | | | |
| 6 | Ping PC-Manager (VLAN10) to SALES gateway (.33) | Success | | | |
| 7 | Ping PC-Manager (VLAN10) to WORKSHOP gateway (.65) | Success | | | |
| 8 | Ping GuestLaptop1 to MGMT gateway (.1) | **Fail** — blocked by ACL | | | |
| 9 | Ping GuestLaptop1 to SALES/WORKSHOP/SERVERS gateways | **Fail** — blocked by ACL | | | |
| 10 | Ping GuestLaptop1 to external/simulated host | Success — outbound still works | | | |
| 11 | `show ip route` on Router1 | All 5 connected subnets listed | | | |

## How to capture evidence

For each test, use Packet Tracer's Desktop > Command Prompt on the source device, run the
command, and screenshot the full terminal window showing both the command and its output —
not just the result. Save screenshots into this folder with descriptive names, e.g.
`test-08-guest-to-mgmt-blocked.png`.
