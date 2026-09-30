# Reflection

*points below are a starting outline based on what actually happened
during the project, not a finished reflection.*

## What went well

- The VLSM addressing plan fit the client's requirements cleanly, with
  room left for future growth.
- The guest isolation requirement (CR3) was implemented with a single,
  clearly-scoped ACL and verified with real evidence, including a
  router-generated "Destination host unreachable" response proving the
  ACL is actively enforced.
- Three real faults were found and independently diagnosed during
  testing, rather than one artificial fault being planted — this gave
  genuine practice with CDP, MAC address tables, interface status, and
  systematic elimination as diagnostic tools.

## Challenges

- The trunk/native VLAN mismatch was subtle — the interfaces looked
  "configured" but had trunk and access commands mixed together on the
  same port, which took careful reading of `show running-config` to
  spot.
- Tracing the missing physical links for LAP-Diagnostic and PC-Jobcard
  required ruling out configuration as a cause first (VLAN assignment,
  DHCP pool) before confirming, via the MAC address table, that the
  devices had never actually sent a frame to the switch — a good
  lesson in not assuming a config problem when the real issue is
  physical.

## What I would do differently

- build and verify physical,cabling before configuring VLANs, to catch this kind of fault
  earlier.

## What I learned

- Networking is important for better communication and sharing of resources.