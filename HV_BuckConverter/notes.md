# Schematic Review and Understanding Guide

## 1. The big picture

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
                           │                 │
                         input             trim
                         caps                │
                           │                 │
VIN_RETURN ────────────────┴─────────────────┘
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

### Key roles

**U1** is doing the major voltage conversion. The RSDW40H-24 is a 40 W
isolated DC-DC converter designed for the high input voltage range.

**U2** is doing the final cleanup/regulation. The MC78M15CDTG is a 15 V
linear regulator.

So you are not asking the linear regulator to turn 160 V into 15 V.
Instead:

**40--160 V → \~21.6 V → 15 V**

------------------------------------------------------------------------

## 2. F1 --- fuse

You have:

**F1 = 0.5 A, 250 VDC**

``` text
VIN
 │
[FUSE]
 │
```

The purpose is to protect the circuit if something goes catastrophically
wrong.

For example, if U1 fails short-circuit, the fuse can interrupt the input
current.

### Fuse sizing

Do not select the fuse simply because the converter says 0.5 A. Estimate
worst-case input current:

\[ I\_{IN} pprox rac{P\_{OUT}}{`\eta `{=tex}V\_{IN}} \]

For 40 W and 89% efficiency:

At 160 V:

\[ I\_{IN}pproxrac{40}{0.89(160)}pprox0.28A \]

At 40 V:

\[ I\_{IN}pproxrac{40}{0.89(40)}pprox1.12A \]

So a 40 W converter can require substantially more input current at low
input voltage.

A 0.5 A fuse could therefore potentially prevent full rated power at the
low end of the input range. This should be investigated using the
converter's actual maximum-input-current specifications and
startup/inrush requirements.

------------------------------------------------------------------------

## 3. L1 --- input inductor

After the fuse:

``` text
VIN
 │
Fuse
 │
L1
 │
 U1 VIN+
```

L1 is part of the input EMI/filtering network.

Switching converters rapidly change current internally. Those current
changes contain high-frequency components that can travel back toward
the source.

An inductor makes high-frequency current changes more difficult:

\[ X_L=2`\pi `{=tex}fL \]

As frequency increases, the inductor's impedance increases.

So L1 can pass the DC input while helping suppress some high-frequency
noise.

### Improvement to investigate

Check L1's:

-   inductance
-   saturation current
-   RMS current rating
-   DC resistance
-   voltage rating/insulation
-   temperature rise

At minimum input voltage, input current is highest, so L1 needs to
tolerate the worst-case current without problematic saturation.

------------------------------------------------------------------------

## 4. C1 --- 100 µF input capacitor

C1 = **100 µF** across VIN+ and VIN_RETURN.

This is your bulk input capacitance.

It provides local energy storage and helps reduce voltage fluctuations
at the converter input.

``` text
        L1
VIN ───LLLL─────┬──── U1
                │
              100µF
                │
VIN_RETURN ─────┴────
```

If U1 suddenly demands a pulse of current, the capacitor can supply some
of that current locally rather than forcing the source and wiring to
provide an instantaneous current change.

------------------------------------------------------------------------

## 5. C9--C12 --- high-frequency filtering

You have 4700 pF capacitors around the converter input/output
connections.

These behave very differently from the 100 µF bulk capacitor.

### 100 µF

Useful for relatively low-frequency/bulk energy storage.

### 4700 pF

More relevant to high-frequency noise.

Capacitive reactance is:

\[ X_C=rac{1}{2`\pi `{=tex}fC} \]

For 4700 pF:

-   At 1 kHz: approximately 33.9 kΩ
-   At 1 MHz: approximately 33.9 Ω
-   At 10 MHz: approximately 3.39 Ω

So the capacitors become much more effective at providing a
low-impedance path for high-frequency noise as frequency rises.

------------------------------------------------------------------------

## 6. Why two 4700 pF capacitors in series?

Two equal capacitors in series have:

\[ C\_{eq}=rac{C}{2} \]

Therefore:

\[ C\_{eq}=rac{4700}{2}pprox2350pF \]

A reason for using series capacitors can be voltage stress distribution.
With a high-voltage input, two capacitors can share the voltage.

However:

> You cannot automatically assume the voltage divides perfectly equally
> under every condition.

For a serious high-voltage design, examine:

-   capacitor type
-   leakage current
-   voltage rating
-   transient conditions
-   whether balancing resistors are required

This is a good example of the difference between copying a reference
circuit and understanding why the circuit works.

------------------------------------------------------------------------

# 7. U1 --- RSDW40H-24

This is the heart of the circuit.

The **RSDW40H-24** is a 40 W isolated DC-DC converter.

Conceptually:

``` text
40–160 VDC
     │
     ▼
 ┌──────────────┐
 │ RSDW40H-24   │
 │              │
 │   DC → DC    │
 │              │
 └──────────────┘
     │
     ▼
   24 V
```

The important property is **isolation**.

The input and output returns are electrically isolated inside the
converter.

Conceptually:

``` text
VIN_RETURN
    │
    │     isolation barrier
    │           ║
    │           ║
    │           ║
                │
          VOUT_RETURN
```

That is an important safety and system-level feature.

------------------------------------------------------------------------

# 8. The trim resistor

You have:

**Rd = 76.8 kΩ**

connected between:

**TRIM → +VOUT**

This is the trim-down configuration.

The basic idea is:

> "Instead of giving me 24 V, lower your output."

Your target is approximately:

\[ 24V imes0.9=21.6V \]

So you are using approximately the full 10% trim-down range.

This is a good design decision to derive directly from the datasheet
rather than treating 76.8 kΩ as a magic value.

------------------------------------------------------------------------

# 9. Why 21.6 V?

This is one of the most important design decisions.

U2 is:

**MC78M15CDTG**

a fixed 15 V linear regulator.

A linear regulator maintains approximately:

\[ V\_{OUT}=15V \]

while dissipating the excess voltage as heat.

Voltage dropped:

\[ V\_{dropped}=V\_{IN}-V\_{OUT} \]

With 21.6 V:

\[ 21.6-15=6.6V \]

So U2 has 6.6 V of nominal headroom.

The key engineering question is whether 21.6 V remains high enough under
all relevant conditions after accounting for:

-   converter output tolerance
-   trim accuracy
-   load regulation
-   temperature
-   transients
-   regulator dropout

------------------------------------------------------------------------

# 10. U2 --- MC78M15CDTG

This is essentially a 7815-family linear regulator.

``` text
21.6V ────────┐
              │
             U2
              │
              └────── 15V
```

It regulates the output around 15 V.

It also provides protection features such as current limiting and
thermal shutdown.

But:

> Thermal shutdown is not a substitute for thermal design.

------------------------------------------------------------------------

# 11. The important thermal calculation

The linear regulator dissipates:

\[ P\_{loss}=(V\_{IN}-V\_{OUT})I \]

With 21.6 V input:

\[ P\_{loss}=6.6I \]

Examples:

### 100 mA

\[ P=6.6(0.1)=0.66W \]

### 250 mA

\[ P=6.6(0.25)=1.65W \]

### 500 mA

\[ P=6.6(0.5)=3.3W \]

3.3 W is significant for a small regulator package.

This is one of the biggest things to analyze in the design.

------------------------------------------------------------------------

# 12. Why not make U1 output 15 V directly?

The RSDW40H family has different output variants, including 15 V
versions.

So ask:

``` text
40–160 V
   ↓
24 V converter
   ↓
21.6 V
   ↓
15 V linear
```

versus:

``` text
40–160 V
   ↓
15 V converter
   ↓
15 V
```

There could be valid reasons for the first architecture:

-   availability
-   output noise
-   regulation
-   filtering
-   transient performance
-   system requirements

But if the only objective is "make 15 V," ask:

> **What benefit does the second regulator provide that justifies its
> efficiency and thermal cost?**

That is an important engineering-design question.

------------------------------------------------------------------------

# 13. C13 and C14

You have:

-   C13 = **0.33 µF** at the regulator input
-   C14 = **0.1 µF** at the regulator output

These are bypass/decoupling capacitors.

The input capacitor is especially important for regulator stability and
local high-frequency bypassing.

Physical placement matters:

> A capacitor that is electrically "near" the regulator on the schematic
> but physically far away on the PCB is not equivalent to one placed
> directly beside the regulator pins.

Keep these connections short and low-inductance.

------------------------------------------------------------------------

# 14. R1 + D1 --- power indicator

You have:

``` text
15V
 │
R1 = 750Ω
 │
LED
 │
VOUT_RETURN
```

This is a simple power indicator.

The resistor limits LED current:

\[ I=rac{15-V_F}{750} \]

If:

\[ V_Fpprox2V \]

then:

\[ Ipproxrac{13}{750}pprox17mA \]

That's reasonable for a traditional indicator LED.

However, you could likely use a lower LED current with a modern LED and
save power.

For example, if 2--5 mA is visually sufficient, the resistor could be
increased accordingly.

------------------------------------------------------------------------

# 15. J1 and SW1

This section implements a latching power switch.

Conceptually:

``` text
J1
 │
 ▼
SW1
 │
 ├──── VIN
 │
 └──── GND
```

The annotation says:

> "Latched switch, NO wired"

The goal is essentially:

``` text
SW1 OFF → VIN disconnected
SW1 ON  → VIN connected
```

For a polished schematic, make the switch's function visually obvious so
another engineer does not have to trace every connection to understand
it.

------------------------------------------------------------------------

# 16. J2 --- output connector

J2 is your 8-pin output header.

If the intent is to distribute the 15 V and return connections across
multiple pins, make the connections explicit.

For example:

``` text
15V ─────┬── 1
         ├── 2
         ├── 3
         └── 4

GND ─────┬── 5
         ├── 6
         ├── 7
         └── 8
```

Multiple parallel power/return pins can help with connector current
capacity and contact resistance, but the actual pinout should be
documented clearly.

------------------------------------------------------------------------

# 17. Grounding and isolation

Your important nets are:

-   `VIN`
-   `VIN_RETURN`
-   `VOUT_RETURN`
-   `GND`

Keeping input return and output return separate is important because U1
provides isolation.

Do not accidentally create:

``` text
VIN_RETURN ───────── GND
```

unless the system requirements specifically call for grounding/bonding
the isolated side.

A clearer schematic could show an explicit isolation boundary:

``` text
          NON-ISOLATED INPUT
                 │
       ┌─────────┴─────────┐
       │                   │
      VIN             VIN_RETURN
       │                   │
       └─────── U1 ────────┘
                  ║
                  ║ ISOLATION
                  ║
       ┌──────────┴──────────┐
       │                     │
      15V              VOUT_RETURN
```

This makes the safety architecture easier to understand.

------------------------------------------------------------------------

# 18. Functional blocks

Think of the whole schematic as five blocks.

## Block 1 --- Input protection

``` text
VIN → F1
```

Protects against catastrophic overcurrent.

## Block 2 --- EMI/input filtering

``` text
F1 → L1 → C1 + C9–C12
```

Reduces conducted switching noise and stabilizes the converter input.

## Block 3 --- Main conversion

``` text
RSDW40H-24
```

Converts:

**40--160 VDC → \~24 VDC**

while providing isolation.

## Block 4 --- Post-regulation

``` text
Trim
 ↓
~21.6 V
 ↓
MC78M15
 ↓
15 V
```

Provides the final regulated output.

## Block 5 --- User interface/output

``` text
15V → LED
15V → J2
```

------------------------------------------------------------------------

# 19. What I would improve

Do a design review before immediately redrawing the schematic.

## ① Fuse sizing

Calculate worst-case input current at minimum VIN and maximum intended
output power.

Do not simply choose the fuse based on the nominal current at one
operating point.

Also consider startup/inrush current and the fuse's time-current
characteristic.

------------------------------------------------------------------------

## ② Thermal analysis of U2

This is probably the biggest engineering question.

Calculate:

\[ P\_{U2}=(21.6-15)I\_{OUT} \]

Then use the regulator's thermal resistance and PCB copper area to
estimate junction temperature.

At 500 mA, the nominal dissipation is:

**3.3 W**

That deserves serious attention.

------------------------------------------------------------------------

## ③ Why use a linear post-regulator?

Ask:

> What problem is U2 solving?

Potential reasons:

-   reducing output ripple
-   improving regulation
-   filtering switching noise
-   providing a clean 15 V rail

If that's the purpose, quantify it.

For example:

> "The DC-DC converter has X mV ripple and the linear regulator reduces
> it to approximately Y mV."

Then the second regulator has a defensible engineering purpose.

------------------------------------------------------------------------

## ④ Reconsider the 21.6 V trim point

You're operating near the maximum trim-down range.

Verify:

-   minimum/maximum trim accuracy
-   converter output tolerance
-   temperature variation
-   load variation
-   worst-case U2 dropout
-   transient conditions

You need to ensure:

\[ V\_{U1,min}\>V\_{U2,required} \]

under all relevant operating conditions.

At the same time, you don't want excessive voltage across U2 because
that increases heat.

This becomes a tradeoff:

**enough headroom for regulation vs. minimizing linear-regulator power
loss.**

------------------------------------------------------------------------

## ⑤ Check capacitor voltage ratings

This is especially important because the circuit can see **up to 160
VDC**.

Every capacitor on the high-voltage side needs an appropriate voltage
rating and transient margin.

Don't only check:

> "160 V \< capacitor rating"

Consider:

-   maximum steady-state voltage
-   input transients
-   tolerances
-   temperature
-   aging
-   capacitor DC-bias behavior where applicable

------------------------------------------------------------------------

## ⑥ Check L1 saturation current

The inductor needs to tolerate the worst-case input current without
excessive saturation.

At minimum input voltage, input current is highest.

So approximately:

\[ I\_{L1,max}pprox I\_{IN,max} \]

Then compare this against the inductor's:

-   saturation current
-   RMS current rating
-   temperature rise
-   DCR

------------------------------------------------------------------------

## ⑦ Check creepage and clearance

This is especially important because you're working with **160 VDC** and
an isolated converter.

Don't only think:

> "I made all the traces nice and straight."

Instead ask:

> **"What voltage exists between these two conductors, and how much
> physical separation is required?"**

You need to think about both:

-   creepage: distance along the PCB surface
-   clearance: shortest distance through air

Also consider the isolation requirements around U1 and between the
high-voltage input and isolated output domains.

------------------------------------------------------------------------

# 20. The biggest conceptual shift

When you first made this PCB, it would have been natural to focus on:

> "Make the traces straight, neat, and thicker."

That's normal while learning.

But the next stage is to start thinking differently.

Instead of:

> "Does my PCB look clean?"

start asking:

## Electrical

-   What current flows here?
-   What voltage is here?
-   What's the ripple?
-   What's the switching frequency?
-   What's the impedance?
-   Where does the return current flow?

## Thermal

-   How much power is dissipated?
-   Where does that heat go?
-   Does the component need copper area or heatsinking?

## Safety

-   What's the maximum voltage?
-   Where is the isolation barrier?
-   What's the creepage/clearance?
-   What happens during an input transient?

## EMI

-   Where are the high-di/dt loops?
-   Where are the high-dv/dt nodes?
-   Where should the capacitors physically sit?
-   Where does the high-frequency return current flow?

## Reliability

-   What happens if the load shorts?
-   What happens if VIN is 160 V?
-   What happens if the input spikes?
-   What happens if U1 fails?
-   What happens if U2 overheats?
-   What happens during startup?
-   What happens if a connector is unplugged under load?

This is the transition from:

> **"I can make a PCB."**

to:

> **"I can design a power supply."**

This particular project is a very good one to revisit because you
already have the schematic and PCB. You can reverse-engineer your own
previous decisions and understand what you would do differently now.

------------------------------------------------------------------------

# 21. Recommended next step: redesign/review from first principles

A useful next stage is to **redesign/review the circuit from first
principles**.

Work through it in this order:

1.  **Define requirements**

    -   40--160 V input
    -   15 V output
    -   Required output current
    -   Required output power

2.  **Calculate worst-case currents**

3.  **Understand why RSDW40H-24 was selected**

4.  **Derive the 76.8 kΩ trim resistor yourself**

5.  **Calculate the input/output capacitor requirements**

6.  **Analyze the 0.5 A fuse**

7.  **Analyze L1**

8.  **Calculate U2's heat dissipation**

9.  **Determine the maximum safe output current**

10. **Redesign the PCB layout around current, thermal performance, EMI,
    and isolation rather than aesthetics**

The goal is to turn this from:

> **"a schematic you made a while ago"**

into something you can genuinely explain in an interview when someone
asks:

> **"Walk me through a power electronics project you've worked on and
> explain why you selected the components."**

That is much more valuable than simply making the old schematic look
prettier.
