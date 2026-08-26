# Client Requirements Analysis

## Client
Motheo Tyre & Exhaust Centre, Vryburg — automotive tyre and exhaust workshop.

## Stated requirements (from project brief CMPG325-2026-137)

| # | Requirement | Source |
|---|---|---|
| 1 | Use assigned addressing block 192.168.58.0/24 | Brief §6 |
| 2 | Provide appropriate connectivity and network services for the scenario | Brief §6 |
| 3 | Accommodate design constraint: new app/file server planned within 6 months | Brief §8 |
| 4 | Configure, verify and demonstrate Network Troubleshooting (fault isolation) | Brief §9 |
| 5 | CR3 — Guest Wi-Fi must be added for visitors, isolated from internal resources | Brief §10 |
| 6 | Test relevant end-to-end connectivity, verify the assigned challenge, document evidence | Brief §11 |

## Inferred functional needs

Reading between the explicit requirements, the business itself implies three functional
areas that need network access with different trust levels:

- **Front office / management** — admin, invoicing, general business data
- **Sales / parts counter** — customer transactions, parts lookups
- **Workshop** — technician devices, diagnostics, separate from office data

These three areas plus the guest requirement (CR3) and the reserved server (constraint)
directly produced the five-VLAN design in `02-design/`.

## Out of scope

Per the brief's explicit instruction ("follow the specific wording of this project rather
than introducing additional scope"), the following were deliberately **not** added:
- No additional change requests beyond CR3
- No server actually built (only reserved) — constraint says "planned," not "required now"
- No additional networking challenge beyond the assigned fault-isolation scenario
