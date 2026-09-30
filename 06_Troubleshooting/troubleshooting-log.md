# Troubleshooting Log — Network Troubleshooting (Fault Isolation)

This document satisfies the assigned networking challenge in section 9
of the project brief: **Network Troubleshooting (fault isolation
scenario), Intermediate difficulty.**

Two genuine faults were discovered and resolved during implementation
and testing — not staged in advance. Both are documented below using
a symptom → diagnosis → root cause → fix → re-verification structure,
suitable for direct use in the video demonstration.

---

## Fault 1: Native VLAN Mismatch (SW-Workshop ↔ SW-Core Trunk)

### Symptom

While reviewing switch configuration, the following log message
appeared on both SW-Core and SW-Workshop:

```
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on
FastEthernet0/1 (1), with SW-Workshop FastEthernet0/24 (30).
```

### Diagnosis

`show running-config` on SW-Workshop showed Fa0/24 configured as:

```
interface FastEthernet0/24
 switchport access vlan 30
 switchport trunk allowed vlan 30
 switchport mode access
```

`switchport mode access` overrides the trunk-related lines — the port
was sending all traffic untagged and reporting its access VLAN (30) as
native, while SW-Core's Fa0/1 (a proper trunk) treated untagged frames
as VLAN 1. The result: VLAN 30 traffic from the workshop never reached
VLAN 30 on the core side, since it was arriving untagged into the
wrong VLAN.

### Root Cause

The trunk and access configuration on Fa0/24 had effectively been
swapped/left in an inconsistent state: trunk-only commands were mixed
with `switchport mode access`.

### Fix

```
enable
configure terminal
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 30
 no switchport access vlan
end
copy running-config startup-config
```

### Verification

`show interfaces trunk` on both switches after the fix:

```
Port      Mode    Encapsulation  Status      Native vlan
Fa0/1     on      802.1q         trunking    1
Fa0/24    on      802.1q         trunking    1
```

Both ends now agree on native VLAN 1, VLAN 30 is correctly allowed on
the trunk, and the CDP mismatch message stopped appearing.

**Evidence:** `screenshots/troubleshooting-trunk/`

---

## Fault 2: No Connectivity — LAP-Diagnostic and PC-Jobcard (Workshop)

### Symptom

Both LAP-Diagnostic and PC-Jobcard (intended for VLAN 30, WORKSHOP)
failed to obtain a usable network configuration:

- PC-Jobcard: `ipconfig` showed IPv4 Address `0.0.0.0`, Subnet Mask
  `0.0.0.0`, Default Gateway `0.0.0.0` — no DHCP lease at all.
- LAP-Diagnostic: obtained `192.168.58.70 /27` with gateway
  `192.168.58.65` (a valid WORKSHOP address), but could **not** ping
  its own default gateway — 100% packet loss.

### Diagnosis

1. `show interfaces status` on SW-Workshop showed every port except
   Fa0/1 (trunk), Fa0/2 (VLAN 30) and Fa0/24 (trunk) as `notconnect`.
2. `show cdp neighbors detail` confirmed the only CDP-visible neighbour
   was SW-Core, on Fa0/24 — ruling out any other switch-to-switch link
   issue.
3. Since PCs do not run CDP, `ipconfig /all` was used on each PC to
   obtain its MAC address:
   - LAP-Diagnostic: `0060.7084.BC57`
   - PC-Jobcard: `0090.210D.8ADE`
4. `show mac-address-table` on SW-Workshop was checked against both
   MAC addresses — **neither appeared anywhere in the table.** A
   switch only learns a MAC address after receiving a frame on a port,
   so this confirmed both devices had never sent a single frame to
   this switch.

### Root Cause

LAP-Diagnostic and PC-Jobcard were not physically connected to
SW-Workshop with a working link — no cable, or a cable not properly
seated at both ends — despite appearing adjacent to the switch in the
topology diagram.

### Fix

1. Verified/redrew the copper straight-through cables from
   LAP-Diagnostic and PC-Jobcard to SW-Workshop Fa0/3 and Fa0/4
   respectively, confirming solid green link lights on both ends.
2. Configured the newly-connected ports:

   ```
   enable
   configure terminal
   interface range FastEthernet0/3 - 4
    switchport access vlan 30
    switchport mode access
   end
   copy running-config startup-config
   ```

### Verification

- `show mac-address-table` on SW-Workshop now listed both MAC
  addresses against Fa0/3 and Fa0/4.
- `ipconfig /release` then `/renew` on both PCs obtained valid
  WORKSHOP addresses (192.168.58.67 and .68, gateway .65).
- `ping 192.168.58.65` from both devices succeeded with 0% loss.

**Evidence:** `screenshots/troubleshooting-cabling/`

---

## Fault 3: Guest Wi-Fi SSID Mismatch (GuestPhone1 ↔ AP-Guest)

*(Bonus/related fault found while validating CR3 — included for
completeness since it is a genuine, separately-diagnosed issue.)*

### Symptom

GuestPhone1 could not obtain a valid IP address. `ipconfig` showed:

```
Autoconfiguration IPv4 Address..: 169.254.220.6
Default Gateway..................: 0.0.0.0
```

A 169.254.x.x address is an APIPA (self-assigned) address, indicating
the device never reached a DHCP server.

### Diagnosis

Compared the wireless configuration on both ends:

- **GuestPhone1** (Config → Wireless0): SSID set to `Default`.
- **AP-Guest** (Config → Port 1): SSID set to `Motheo-Guest`,
  WPA2-PSK, passphrase `Vryburg2026`.

The phone was attempting to associate with a network that did not
exist, so it never connected to the AP at all.

### Root Cause

SSID mismatch between the client device and the access point.

### Fix

On GuestPhone1 (Config → Wireless0): changed SSID from `Default` to
`Motheo-Guest` to match the AP, keeping the existing WPA2-PSK
passphrase.

### Verification

- `ipconfig` on GuestPhone1 now showed `192.168.58.114 /28`, gateway
  `192.168.58.113` — a valid GUEST address.
- `ping 192.168.58.97` (SERVERS VLAN) correctly failed, but this time
  with an active router response — `Reply from 192.168.58.113:
  Destination host unreachable` — confirming the router's
  `GUEST-ISOLATE` ACL is actively enforcing the deny rule (CR3), not
  simply that the device has no connectivity at all.

**Evidence:** `screenshots/troubleshooting-guest-wifi/`

---

## Summary

| # | Fault | Category | Method Used | Result |
|---|---|---|---|---|
| 1 | Native VLAN mismatch | Switch trunk config | CDP log, `show interfaces trunk`, `show running-config` | Fixed |
| 2 | No physical link (2 devices) | Cabling | `show interfaces status`, `show cdp neighbors`, `show mac-address-table`, MAC cross-reference | Fixed |
| 3 | SSID mismatch | Wireless config | `ipconfig` (APIPA detection), config comparison | Fixed |

All three faults were isolated using standard, methodical diagnostic
commands (log messages, interface status, CDP, MAC address tables, and
IP configuration checks) rather than guesswork — directly demonstrating
the assigned Network Troubleshooting (fault isolation) challenge.
