# SW-Core — Running Configuration

Paste `show running-config` output here after Phase 3 of the build guide.

```
[paste show running-config output here]
```

## Notes
- Confirm VLANs 10, 20, 30, 40, 99 exist (`show vlan brief`).
- Confirm fa0/1, fa0/2 and fa0/24 are trunks carrying all 5 VLANs (`show interfaces trunk`).
- Confirm fa0/3 is an access port in VLAN 99.
