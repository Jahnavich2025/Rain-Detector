# Rain Detector Circuit — LTspice Current Analysis

An LTspice XVII schematic with a companion interactive visualizer for a **Rain Detector using LM393 Comparator**, focused on understanding **current flow** through each branch under dry and wet conditions.

---

## Files

```
/
├── rain_detector.asc        ← LTspice schematic (open directly in LTspice XVII)
├── rain_detector_viz.html   ← Interactive current visualizer (open in browser)
└── README.md                ← This file
```

---

## Circuit Topology

```
     VCC (5V)
      │
     R1 (10kΩ)  ← pull-up, sets quiescent current
      │
   NODE_A ─────────────────────────── (+) ─┐
      │                                   LM393    Open-collector OUT
   R_rain  (variable: 10MΩ dry → 50Ω wet)  │              │
      │                         V_THRESH── (−)       R_pull (10kΩ) ── VCC
     GND                                              R_LED  (470Ω)
                                                      D1 (LED)
                                                      GND
```

---

## Operating Principle

| Condition | R_rain | V_NODE_A | Comparator | LED |
|-----------|--------|----------|------------|-----|
| Dry       | 10 MΩ  | ≈ 5.00 V | V+ > V−, OUT HIGH | OFF |
| Drizzle   | 10 kΩ  | ≈ 2.50 V | Transitioning     | DIM |
| Rain      | 5 kΩ   | ≈ 1.67 V | V+ < V−, OUT LOW  | ON  |
| Flooded   | 500 Ω  | ≈ 0.24 V | V+ << V−, OUT LOW | ON  |

```
V_NODE_A = VCC × R_rain / (R1 + R_rain)
LED ON when V_NODE_A < V_THRESH (2.5V)  →  R_rain < 10kΩ
```

---

## Current Analysis (VCC = 5V)

| LTspice Probe | Branch        | Dry (10MΩ) | Rain (5kΩ) | Flooded (500Ω) |
|---------------|---------------|------------|------------|----------------|
| `I(R1)`       | Sensor branch | ~0 µA      | ~250 µA    | ~476 µA        |
| `I(R_pull)`   | Output pull-up| ~490 µA    | ~490 µA    | ~490 µA        |
| `I(R_LED)`    | LED current   | 0 mA       | ~6.3 mA    | ~6.3 mA        |
| `I(V1)`       | Total supply  | ~0.5 mA    | ~7.3 mA    | ~7.3 mA        |

### LED Current Formula

```
I_LED = (VCC − V_F − V_OL) / R_LED
      = (5 − 2.0 − 0.05) / 470
      ≈ 6.3 mA   (safe: 5–20mA range for standard LED)
```

---

## How to Run in LTspice XVII

### 1 — Open the Schematic
```
File → Open → rain_detector.asc
```

### 2 — DC Operating Point (single condition)
```spice
.param R_sensor 10k    ; try: 10MEG / 100k / 10k / 5k / 500
.op
```

### 3 — Parametric Sweep (all conditions)
```spice
.step param R_sensor list 10MEG 100k 10k 5k 1k 500
.op
```
After run: right-click any wire → **Add Current Probe** → plot `I(R1)`, `I(R_LED)`, `I(R_pull)`.

### 4 — Automatic Measurements
```spice
.meas DC I_SENSOR   find I(R1)       ; sensor branch current
.meas DC I_LED      find I(R_LED)    ; LED current
.meas DC I_PULL     find I(R_pull)   ; pull-up current
.meas DC I_TOTAL    find I(V1)       ; total supply current
.meas DC V_NODE_A   find V(NODE_A)   ; sensor node voltage
```

---

## Component Rationale

| Component        | Value    | Purpose |
|------------------|----------|---------|
| R1               | 10 kΩ    | Pull-up; trigger when R_rain ≈ R1 |
| R_thresh × 2     | 10 kΩ ea | 2.5V midpoint reference (VCC/2) |
| R_pull           | 10 kΩ    | Open-collector output pull-up |
| R_LED            | 470 Ω    | Limits LED to ~6.3 mA |
| D1               | LED      | Visual indicator (sim uses 1N4148) |
| U1               | LM393    | Open-collector voltage comparator |

---

## Interactive Visualizer

Open `rain_detector_viz.html` in any browser — no install needed.

- Drag **R_rain slider** from Dry → Submerged; all currents update in real time
- **Current bars** scale proportionally to computed branch currents
- **LED glows red** when V_NODE_A drops below the 2.5V threshold
- **Rain animation** plays when detection triggers
- Values match LTspice `.meas` probe outputs exactly

---

## Keywords

`LTspice XVII` · `Rain Detector` · `LM393 Comparator` · `Open Collector` · `Branch Current Analysis` · `DC Operating Point` · `Parametric Sweep` · `.meas` · `Analog Design` · `Sensor Interface Circuit`
