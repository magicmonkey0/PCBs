# Schematic Review and Understanding Guide

## 1. The Idea

This schematic contains fusing, input filtering, a high-voltage DC-DC
converter, voltage trimming, a linear regulator, decoupling, an
indicator LED, and connectors.

Overall architecture:

**40--160 VDC input → protection/filtering → isolated 24 V DC-DC
converter → trim to \~21.6 V → 15 V linear regulator → 15 V output**

``` text
             HIGH VOLTAGE SIDE
                   │
                   ▼
VIN ── Fuse ── Inductor ──┬── RSDW40H-24 ──┬── 21.6 V
                          │                │
                         input            trim
                         caps              │
                           │               │
VIN_RETURN ────────────────┴───────────────┘
                                           │
                                           ▼
                                      MC78M15CDTG
                                   linear regulator
                                           │
                                           ▼
                                          15 V
                                           │
                                           ▼
                                      OUTPUT LOAD
```
Credits: T. Phantom
