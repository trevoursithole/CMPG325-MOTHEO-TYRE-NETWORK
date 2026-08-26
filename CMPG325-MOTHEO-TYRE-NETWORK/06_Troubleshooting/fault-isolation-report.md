# Assigned Technical Challenge — Network Troubleshooting (Fault Isolation)

## Scenario

The workshop technician reports they cannot reach shared resources on the rest of the
network. Fault deliberately introduced: VLAN 30 removed from SW-Core's uplink trunk to
Router1 (`interface fa0/24`).

## Fault-isolation method used

A layered (bottom-up) approach was used to isolate the cause before making any changes.

### Step 1 — Confirm the symptom (Layer 3)
Command: `ping 192.168.58.65` from PC-Jobcard
Result: [paste result / screenshot reference]

### Step 2 — Rule out a physical fault (Layer 1)
Command: `show interfaces fa0/24 status` on SW-Workshop
Result: [paste result / screenshot reference — link should be up]

### Step 3 — Check VLAN and trunk configuration (Layer 2)
Commands: `show vlan brief` (SW-Workshop), `show interfaces trunk` (SW-Core)
Result: [paste result / screenshot reference — VLAN 30 missing from SW-Core's allowed list]

### Step 4 — Root cause
[State the root cause in your own words once you've actually reproduced it]

### Step 5 — Correction
Command applied on SW-Core:
```
interface fa0/24
 switchport trunk allowed vlan add 30
```

### Step 6 — Re-verification
Command: `ping 192.168.58.65` from PC-Jobcard
Result: [paste result / screenshot reference — should now succeed]

## Screenshots

| Stage | Filename |
|---|---|
| Before — failed ping | |
| Before — show interfaces status | |
| Before — show vlan brief | |
| Before — show interfaces trunk (fault visible) | |
| After — corrected trunk config | |
| After — successful ping | |

## Reflection

[Once you've actually run this: what did you learn about isolating VLAN/trunk faults
specifically, and how would you approach a different kind of fault — e.g. an ACL
misconfiguration or wrong subnet mask — differently?]
