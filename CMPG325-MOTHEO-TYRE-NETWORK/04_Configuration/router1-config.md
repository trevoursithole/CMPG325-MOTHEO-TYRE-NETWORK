# Router1 — Running Configuration

Paste the output of `show running-config` here once configured in Packet Tracer.
Capture this AFTER Phase 6 of the build guide is complete and verified.

```
[paste show running-config output here]
```

## Notes
- Confirm all 5 sub-interfaces (g0/0.10, .20, .30, .40, .99) are present with correct
  encapsulation and IP addresses matching the addressing plan.
- Confirm all 4 DHCP pools are present (MGMT, SALES, WORKSHOP, GUEST).
- Confirm the GUEST-ISOLATE ACL is present and applied on g0/0.99.
