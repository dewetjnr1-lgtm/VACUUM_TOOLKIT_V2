# Formula Verification Report — Vacuum & Compressor Toolkit

**Scope:** every calculation in `index.html` (the entire app), checked against published
industry references: Busch Vacuum Academy / technical manuals, Leybold *Fundamentals of
Vacuum Technology*, Peacock (in Lafferty, ed.) *Foundations of Vacuum Science and
Technology*, Giampaolo *Compressor Handbook*, Crane TP-410, Mills *Pneumatic Conveying
Design Guide*, ASME B36.10, DIN 1343 / ISO 2533 (reference conditions), WMO/Sonntag
(Magnus equation), and standard steam tables.

**Method:** each formula was re-derived by hand, the code path traced for unit
conversions, and every calculator's built-in example re-computed independently to
confirm the code reproduces the correct value.

**Verdict summary:** 11 of 12 calculators are correctly implemented. One formula error
was found (the "Which machine" motor estimate), plus four minor items listed below.
Note: the app contains **no orifice plate testing calculator** — see item 5.

---

## 1. Verified correct

### 1.1 Gas-law demos (Learn tab)
| Demo | Code | Reference | Verdict |
|---|---|---|---|
| Boyle | `p = 1000/V` (P₁V₁ = 100×10) | P₁V₁ = P₂V₂ | ✓ correct |
| Charles | `V = 5·T_K/273.15` | V₁/T₁ = V₂/T₂, Kelvin | ✓ correct |
| Gay-Lussac | `p = 100·T_K/273.15` | P₁/T₁ = P₂/T₂, Kelvin | ✓ correct |
| Combined | `p = (100·10/273.15)·T_K/V` | P₁V₁/T₁ = P₂V₂/T₂ | ✓ correct |
| Dalton | `P_dry = 101.3 − P_sat(T)` | P_total = ΣP_i | ✓ correct |

All demos use 273.15 for the Celsius→Kelvin offset. The displayed "plug-in" strings
(1,000 / 0.018 / 0.37 / 3.66) all match hand calculation.

### 1.2 Normal ↔ Actual flow converter (`fc`)
Code: `Q_act = Q_N · (101.325 / P_abs) · (T_K / T_ref,K)` with
`P_abs = P_atm(local) + gauge − line loss` (or `P_atm − vacuum`).

This is the combined gas law rearranged exactly as published in the Busch technical
manual ("conversion of standard volume flow to volume flow") and Giampaolo Ch. 1.
Normal reference is fixed at 101.325 kPa / 273.15 K (DIN 1343), with 15 °C (ISO 2533)
and 20 °C options — all handled correctly. Reverse direction is the exact algebraic
inverse. Gauge/absolute/vacuum reading conversions (`readPress`) are correct:
abs = reading; abs = atm + gauge; abs = atm − vacuum. Guard for P_abs ≤ 0 present.
Example verified: 1000 Nm³/h at −20 kPa g, 40 °C → 1428.8 m³/h. **✓ correct**

### 1.3 Vacuum pump sizing (`vac`)
Same combined-gas-law formula with P_abs = P_atm − vacuum − line loss. The optional
readout back to Nm³/h applies the exact inverse factor `(P_abs/101.325)·(T_ref/T)` —
round-trips to the input, confirming algebraic consistency. **✓ correct**

### 1.4 Compressor sizing (`cmp`)
Compression ratio = P_discharge,abs / (P_inlet,abs − inlet loss), both absolute —
matches Giampaolo's definition. Actual inlet flow uses the same verified gas-law
conversion. CO₂ multiplier applied to the curve-reading flow only (see minor item 4.2).
**✓ correct**

### 1.5 Vapour & wet gas (`vap`)
Saturation pressure: Magnus form `e_s = 6.112·exp(17.62·T/(243.12+T))` hPa — these are
the WMO/Sonntag-1990 coefficients; ÷10 to kPa is correct. Checked against steam tables:
20 °C → 2.333 (table 2.339), 40 °C → 7.367 (table 7.384), 50 °C → 12.345 (table
12.349) — within 0.3 %, the Magnus formula's stated accuracy. Water mole fraction
= P_sat/P_total (Dalton), volumetric split of a saturated flow by mole fraction, and
mixture molecular weight `y_dry·MW_dry + y_w·18.02` are all textbook-correct.
Boiling guard (P_sat ≥ P_total) present. **✓ correct**

### 1.6 Pump-down time (`pd`)
`t = F · (V/S) · ln(P₁/P₂)` with absolute pressures — this is the standard evacuation
equation in the Busch technical manual, Leybold *Fundamentals* (§ "pump-down time") and
Peacock. Units traced: V [m³] / S [m³/h] → hours, ×60 → minutes. ✓
The empirical slowdown factor F (1.5–3) for real systems matches Busch/Leybold practice
of applying a correction because S falls near ultimate pressure. Guard for P₂ ≥ P₁
present. Example verified: 10 m³, 500 m³/h, 1013→50 mbar, F=1.5 → 5.42 min.
The liquid-ring seal-water note (floor ≈ 4/7/12 kPa at 30/40/50 °C) matches water
vapour pressure tables. **✓ correct**

### 1.7 Leak check (`lk`)
`Q_L = V·Δp/Δt` in mbar·L/s — the standard pressure-rise (rate-of-rise) leak formula
per Leybold *Fundamentals* and Busch. Unit conversions traced: m³→L ×1000 ✓,
kPa→mbar ×10 ✓, min→s ×60 ✓. Example: 1 m³, 5 mbar in 10 min → 8.33 mbar·L/s ✓.
Leak-unit table (1 Pa·m³/s = 10 mbar·L/s) correct. **✓ correct**

### 1.8 Pipe pressure loss (`pl`)
Darcy–Weisbach `Δp = f·(L/D)·ρv²/2 + ΣK·ρv²/2` with Swamee–Jain friction factor
`f = 0.25/[log₁₀(ε/3.7D + 5.74/Re^0.9)]²` and laminar `f = 64/Re` below Re 2300 —
all exactly as published (Swamee & Jain 1976; Crane TP-410). Every unit conversion
traced (mm→m, m³/h→m³/s, Pa→kPa, head = Δp/ρg). Example verified: 15 m³/h through
50 mm ID → 2.12 m/s, Re ≈ 106 000, f = 0.0219, ≈ 19.7 kPa. The 1–3 m/s guidance is
standard practice. **✓ correct**

### 1.9 Power & motor current (`pw`)
Isothermal compression power `P_iso = P₁·Q̇₁·ln(P₂/P₁)` — the textbook formula
(Giampaolo; Busch manual). Units traced: kPa × m³/h ÷ 3600 = kW exactly. Divided by
machine efficiency, times motor margin — standard. Example: 1000 m³/h, 80→200 kPa abs,
η 0.45, margin 1.15 → 52 kW → "55 kW motor": realistic.
Three-phase current `I = P/(√3·U·cosφ·η)` with √3 = 1.7320508 — textbook-correct.
Cross-check: 55 kW at 415 V, pf 0.85, η 0.93 → 96.8 A, matching published motor FLC
tables (~98 A). **✓ correct**

### 1.10 Conveying sizing (`pc`)
Dilute-phase model verified against Mills, *Pneumatic Conveying Design Guide*:
- Pickup velocities (15–18 m/s fine powders, ~20 granules, 25 heavy) ✓ standard.
- Solids-loading ratios (2–8 vacuum, up to ~10 pressure, decreasing with length) ✓.
- Bend = 8 m equivalent straight pipe ✓ within the published 6–12 m rule of thumb.
- Air density scaled by pressure `ρ = 1.2·P/101.325` ✓ (isothermal ideal gas at 20 °C).
- Minimum velocity enforced at the *highest-pressure* end of the line (pickup), where
  the air is densest and slowest — correct physics for both vacuum and pressure systems.
- Friction with solids: `f=0.02` air friction times `(1 + 0.7·μ)` solids multiplier —
  a recognised simplified method (Mills' air-only-plus-solids-factor approach).
- Riser static head `μ·ρ·g·h·2` — the ×2 is a slip/holdup allowance, defensible.
- Machine inlet swell for vacuum `Q_pump = ṁ_air/ρ(P_inlet)·1.1` ✓ Boyle-consistent.
- Isothermal motor estimate at 50 % overall efficiency: uses
  `P_atm·Q̇_free·ln(ratio)` — algebraically identical to `P_in·Q̇_in·ln(ratio)`
  because P·Q̇ is invariant (Boyle), so this is **correct** here.
- Iterative solve with convergence check, +20 % pressure margin, IEC motor frame
  round-up ✓.
The tab correctly labels itself budget/dilute-phase-only. **✓ correct as a
rule-of-thumb estimator** (see note 4.3).

### 1.11 Unit converter and reference tables
- Pressure factors (kPa 1, mbar 0.1, bar 100, psi 6.894757, mmHg 0.1333224,
  inHg 3.386389, atm 101.325) — all match NIST values. ✓
- Flow (cfm 1.69901 m³/h, L/s 3.6, L/min 0.06) ✓; length, power (hp 0.7457 kW),
  time, volume, velocity (ft/s 0.3048) ✓.
- Temperature: °C↔°F↔K conversions exact in both directions. ✓
- Saturated vapour table (0.611…12.349 kPa) matches steam tables. ✓
- Water density/viscosity table matches standard data. ✓
- Altitude table matches ISA (ISO 2533): 500 m → 95.5, 1000 m → 89.9 kPa. ✓
- Gas molecular weights ✓. Roughness values ✓ (Moody chart standards).
- Schedule 40 IDs match ASME B36.10 (½" 15.8, 1" 26.6, 2" 52.5, 3" 77.9, 4" 102.3,
  6" 154.1 mm). ✓
- K-factors match Crane TP-410 ballpark values (std elbow 0.75, long 0.45, gate 0.15,
  globe 10). ✓

### 1.12 Which machine (`mp`) — selection logic
Operating windows (side channel to ~600 mbar abs; Roots to ~450; claw to ~140; oiled
vane to 0.05; liquid ring to ~40; dry screw to 0.01 mbar; boost limits) are consistent
with Busch product ranges (R5, Mink, COBRA, Dolphin, Samos, Tyr) and general industry
guidance. The liquid-ring 40 mbar floor warning and sub-1-mbar "booster/turbomolecular"
referral match Peacock/Leybold. Gas-compatibility rankings are sound. **Selection logic
✓ — but see the motor estimate error in 2.1.**

---

## 2. Needs fixing

### 2.1 "Which machine" tab — motor power estimate is wrong for vacuum duties
**Location:** `index.html:559-561`

```js
var ratio=job==="v"?101.325/p:(101.325+p)/101.325;
var work=Math.min(101325*Math.log(ratio),65000);
var kw=(q/3600)*(job==="v"?work:101325*Math.log(ratio))/0.47/1000;
```

**Problem:** isothermal power is `P₁·Q̇₁·ln(P₂/P₁)` where **P₁ is the inlet absolute
pressure** and Q̇₁ the flow *at that pressure*. The input `q` is explicitly "flow at
the working pressure", but the code multiplies it by **atmospheric** pressure
(101 325 Pa) instead of the working pressure `p`. For vacuum jobs this overstates power
by a factor of P_atm/P_inlet:

- At 50 kPa abs, 300 m³/h: code gives **11.5 kW**; correct isothermal is **6.3 kW**
  (~1.8× high).
- The deeper the vacuum, the worse it gets, until the 65 kJ/m³ cap kicks in — and the
  cap itself is wrong: the true peak of `P·ln(P_atm/P)` is **37.3 kJ/m³** (at
  P = P_atm/e ≈ 37.3 kPa), so even the "capped" value is ~74 % too high.
- The in-app "Show the math" text ("flow × 101.3 kPa × ln(pressure ratio)") documents
  the bug: that expression is only valid when the flow is measured as *free air*, which
  contradicts the input's definition.

**Internal inconsistency:** the Power tab (`pw`) and the Conveying tab (`pc`) both
compute isothermal power correctly (P₁·Q̇₁·ln r, and P_atm·Q̇_free·ln r respectively).
Only this tab is wrong.

**Fix:** for vacuum, `work = p*1000*Math.log(101.325/p)` (Pa·ln, per inlet m³) — no cap
needed, the function is naturally bounded at 37.3 kJ/m³. For compression, decide whether
`q` is at inlet (then `101325·ln r` is right) or at discharge (then use
`(101.325+p)*1000·ln r`), and align the help text.

*(Severity: moderate — it is labelled "a conversation number only", and the error is
conservative for vacuum, but it can push the suggested motor 1–2 IEC frames too big.)*

---

## 3. Missing calculators the review brief expected

### 3.1 No orifice plate testing calculator exists
The review request asked to verify "orifice plate testing" formulas. A full-text search
confirms the app contains **no orifice module** (no ISO 5167 flow equation, no
discharge coefficient, no β-ratio, no capacity-test orifice per ISO 21360 / ASME PTC 9).
Nothing to verify — flagging so nobody assumes it's covered. If a pump capacity check
via calibrated orifice is wanted, it would be a new feature (ISO 5167-2 is the
authoritative reference).

---

## 4. Minor items (judgement calls, not errors)

### 4.1 Flow-converter output label is direction-blind (`fc`)
When converting "real flow into reference flow", the answer box still reads **"Real
flow at the inlet"** and the flowN back-conversion is unavailable. The number is
correct; only the label misleads. Cosmetic fix: swap the label per direction.

### 4.2 Compressor "Total for all machines" excludes the CO₂ allowance (`cmp`)
`total = act × N` uses the pre-allowance actual flow, while the per-machine
curve-reading flow includes ×1.15. If the total is meant for header/knock-out sizing on
a CO₂ duty, it should arguably be `curve × N`. Decide the intent and document it.

### 4.3 Conveying acceleration term uses a low solids coefficient (`pc`)
`acc = (1+0.8μ)·ρ·v²/2` puts the solids acceleration at `0.4·μ·ρv²`; the common
textbook form (G_s·v_p with v_p ≈ 0.8 v_gas) gives `0.8·μ·ρv²`, i.e. double. Buried
under the 20 % margin and the tool's stated ±rough status, so acceptable — but worth a
comment in the code.

### 4.4 Magnus formula range (`vap`)
The WMO coefficients are specified for roughly −45…+60 °C over water. The tool accepts
any temperature; above ~60 °C error grows (still <1 % to 80 °C, ~2 % at 100 °C). Fine
for wet-gas sizing; a hint in the help text wouldn't hurt.

### 4.5 No in-app source citations
The "Show the math" panels describe every formula in plain words (good) but cite no
standards or texts. Recommend one line each, e.g.: pump-down & leak — Busch technical
manual / Leybold Fundamentals; pipe loss — Darcy-Weisbach + Swamee-Jain (Crane TP-410);
power — isothermal compression (Giampaolo); vapour — Magnus/WMO; conveying — Mills;
reference conditions — DIN 1343 / ISO 2533.

---

## 5. Fixes applied (2026-07-13)

All flagged items have been fixed in `index.html`, except the orifice plate
calculator (3.1), which the owner decided to drop.

- **2.1 fixed** — the "Which machine" motor estimate now uses the true isothermal
  formula `P_work · Q_work · ln(ratio) / 0.47` for both vacuum and compression. The
  arbitrary 65 kJ/m³ cap is gone (the correct expression is naturally bounded at
  37.3 kJ/m³). Verified in-browser: 300 m³/h at 200 mbar abs now gives 5.8 kW
  (was 11.5 kW); the math panel text was corrected to match.
- **4.1 fixed** — the flow-converter answer label now switches between "Real flow at
  the inlet" and "Reference flow (the datasheet figure)" with the direction.
- **4.2 fixed** — "Total for all machines" now includes the CO₂ allowance
  (`total = curve × N`); verified total = 2 × curve for the 2-machine example.
- **4.3 fixed** — conveying acceleration term now uses the textbook solids
  coefficient: `(1 + 1.6·μ)·ρv²/2` (solids accelerated to ≈0.8× gas velocity).
  Example still converges (DN50, ~217 mbar, 2.2 kW motor).
- **4.4 fixed** — Magnus validity range noted in the vapour tab's math panel.
- **4.5 fixed** — every "Show the math" panel now carries a one-line source citation
  (Busch technical manual, Leybold Fundamentals, Giampaolo, Crane TP-410, Mills,
  DIN 1343, WMO/Sonntag).

All calculators were re-run end-to-end in Chromium after the changes: correct values,
no JavaScript errors.

## 6. Bottom line

| Calculator | Verdict |
|---|---|
| Gas-law demos (Boyle, Charles, Gay-Lussac, Combined, Ideal, Dalton) | ✓ verified |
| Normal/Actual flow converter | ✓ verified |
| Vacuum pump sizing | ✓ verified |
| Compressor sizing | ✓ verified (minor 4.2) |
| Conveying sizing | ✓ verified as rule-of-thumb (note 4.3) |
| Which machine — selection logic | ✓ verified |
| **Which machine — motor estimate** | **✓ fixed (2.1, 5)** |
| Vapour & wet gas | ✓ verified (note 4.4) |
| Pump-down time | ✓ verified |
| Leak check | ✓ verified |
| Pipe loss | ✓ verified |
| Power & motor current | ✓ verified |
| Unit converter & reference tables | ✓ verified |
| Production test orifice flow (`pt`) | ✓ physics verified — test-standard caveat (7) |

---

## 7. Addendum (2026-07-20) — Production test orifice flow calculator

Section 3.1 above noted that a "capacity-test orifice" calculator, if added, should be
checked against ASME PTC 9 or ISO 21360. That calculator has since been built (the
Production test vs factory curve page, `pt`): it works out actual flow at each field
test point from orifice qty, orifice diameter, vacuum, air pressure and air temp, using
the standard compressible-flow equation for a gas orifice (subsonic and choked forms).

**Verified this pass:**
- Re-derived the subsonic and choked-flow equations from isentropic flow theory by
  hand. Confirmed the two forms agree at the choked/subsonic transition (r = P₂/P₁ =
  critical ratio) to within floating-point rounding — a wrong exponent anywhere in
  either branch would have broken this continuity.
- The choked-flow constant the code implies for air (k = 1.4), C\* = √k·(2/(k+1))^((k+1)/(2(k-1))),
  computes to 0.6847, matching the widely published value for air's critical flow
  function (Crane TP-410 and standard gas-dynamics references).
- Two worked examples — one choked case (700 kPa abs → atmosphere through a 10 mm
  orifice) and one subsonic case (200 → 180 kPa abs) — hand-calculated, then checked
  against both a freshly-written independent implementation and the actual deployed
  `orificeFlow()` function. All three agree to the precision shown (236.7 m³/h and
  23.49 m³/h respectively).
- Edge cases checked: zero orifices open → 0 flow (a valid blank-off reading, not a
  missing-data gap); vacuum reading exceeding local air pressure, or discharge
  pressure ≥ upstream pressure → `NaN` with a warning, not a wrong number, per the
  app's house style.

**Not verified — open question:** the physics (compressible flow through a sharp,
square, or rounded-edge orifice) is correct gas dynamics, but this pass did not check
it against ASME PTC 9 or ISO 21360, the standards named in 3.1 for a pump capacity
*test* specifically. Those standards may specify a different or tabulated discharge
coefficient for a calibrated test orifice, a humidity or Reynolds-number correction, or
other test-acceptance conventions not present here — this implementation borrows its
discharge-coefficient options (0.61 / 0.82 / 0.98) from the app's existing liquid
orifice sizing calculator, not from a compressible-flow-specific table. I do not have
access to either standard to check further. Anyone comparing this tool's output against
a factory acceptance certificate that cites PTC 9 or ISO 21360 should expect the
discharge coefficient, in particular, to be the first thing worth reconciling.
