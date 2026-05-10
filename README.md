# syngas-efi
A writeup for a gasoline/syngas efi system

# Wood Gas / Syngas EFI Dual Fuel System
### Complete Design and Engineering Reference

---

## 1. Fuel Properties

### Wood Gas Composition (Typical)

| Component | % by Volume | Role |
|---|---|---|
| Nitrogen (N₂) | 45–55% | Inert diluent |
| Carbon monoxide (CO) | 18–25% | Primary fuel |
| Hydrogen (H₂) | 10–16% | Primary fuel, octane contributor |
| Carbon dioxide (CO₂) | 8–12% | Inert diluent |
| Methane (CH₄) | 2–4% | Secondary fuel |
| Water vapor | trace | — |

### Air/Fuel Ratio

| Parameter | Gasoline | Wood Gas |
|---|---|---|
| Stoichiometric AFR (by volume) | 14.7:1 | 1.1:1 – 1.5:1 |
| Stoichiometric AFR (by mass) | ~14.7:1 | ~1.0:1 – 1.35:1 |
| Lower Heating Value | ~34 MJ/m³ | ~4–6 MJ/m³ |

Wood gas requires roughly equal parts air to gas by volume due to the large inert fraction already present in the gas.

---

## 2. Octane Rating

### Component Octane Ratings (RON)

| Component | RON | Contribution |
|---|---|---|
| Hydrogen (H₂) | 130+ | Very high antiknock |
| Carbon monoxide (CO) | ~106 | High antiknock |
| Methane (CH₄) | 120–130 | High antiknock |

### Effective Octane of Wood Gas

```
H₂  (12%) × 130 = 15.6
CO  (20%) × 106 = 21.2
CH₄  (3%) × 125 =  3.75
Inerts     ×  0 =  0

Weighted RON estimate: ~100–110 RON
(up to 120 RON with high H₂ content)
```

### Fuel Octane Comparison

| Fuel | RON |
|---|---|
| Regular gasoline | 91–95 |
| Premium gasoline | 98–102 |
| Wood gas / syngas | 100–120 |
| Ethanol | 108–109 |
| LPG propane | 105–110 |
| Methane / CNG | 120–130 |
| Hydrogen | 130+ |

H₂ content is the primary variable — high H₂ output (15–18%) puts effective RON at ~115, while poor gasification (H₂ below 8–10%) drops it toward 100. Consistent feedstock (see Section 3) is key to stable octane.

---

## 3. Preferred Feedstock — Alnus incana (Grey Alder)

Grey alder is an excellent gasification feedstock, particularly for engine applications.

### Wood Properties

| Property | Alnus incana | Notes |
|---|---|---|
| Density | 450–520 kg/m³ | Moderate — good gasifier flow |
| Ash content | 0.3–0.5% | Very low — minimal clinker |
| Moisture (air-dry) | 15–20% | Typical well-dried alder |
| Tar yield | Very low | Critical for engine longevity |
| Resin content | Low | Clean gas output |

### Comparison to Other Woods

| Wood | Tar Yield | Ash | Density | Notes |
|---|---|---|---|---|
| Alnus incana | Very low | Very low | Medium | Excellent all-rounder |
| Oak | Low | Low | High | High energy, hard to dry |
| Birch | Low–medium | Low | Medium | Good alternative |
| Pine | High | Low | Medium | Problematic tar/resin |
| Spruce | High | Low | Low | Poor — high tar |
| Willow | Medium | Medium | Low | Fast-growing, lower quality |

Low ash content reduces clinker formation and agitation frequency. Low tar content is the most important engine protection factor — tar condenses in pipes, filters, and intake systems and is the primary cause of gasifier engine failures.

Ideal feedstock moisture: **below 20%**, critical threshold **25%**. Above 25% moisture, tar increases dramatically and CO/H₂ output drops.

---

## 4. EFI System — Megasquirt / Open Source ECU

### Recommended ECU Platforms

| Platform | Suitability | Notes |
|---|---|---|
| MS2/Extra | Good | Solid dual-fuel, wide community |
| MS3/MS3Pro | Best | More I/O, better table resolution |
| Speeduino | Good | Open source hardware, Arduino-based |
| rusEFI | Excellent | Lua scripting, very active development |

TunerStudio MS is the tuning and logging software for all platforms.

### Why MAP-Based Speed Density

Wood gas composition varies with feedstock and gasifier conditions. A MAF sensor would constantly mis-meter the diluted mixture. MAP + TPS + IAT speed-density fueling is fuel-agnostic by nature:

```
Fuel delivery = f(RPM, MAP, IAT correction, CLT correction)
```

Separate VE tables are built for each fuel mode. Megasquirt handles this natively.

---

## 5. Sensor Suite

### Primary Engine Sensors

| Sensor | Purpose | Notes |
|---|---|---|
| TPS | Load input, map switching trigger | Standard |
| MAP | Speed-density fueling, boost monitoring | 0–300 kPa range for turbo |
| IAT | Charge temp correction | Mount post-intercooler |
| CLT | Cold start lockout for syngas mode | Syngas only above 80°C |
| Wideband O₂ (λ) | Closed loop AFR, tuning | Innovate LC-2 or AEM 30-0300 |
| EGT per cylinder | Mixture distribution, lean detection | K-type, one per cylinder |
| Knock sensor | Timing safety, max advance | Closed loop retard |
| Crankcase pressure | Ring wear / blowby monitoring | 0–10 kPa sensor |
| Oil dilution sensor | Tar/fuel contamination in oil | Refractive index or conductivity |
| Barometric pressure | Altitude compensation | Often internal to MAP sensor |
| Ambient humidity | Charge density correction | BME280 combined sensor |

### Gasifier System Sensors

| Sensor | Location | Purpose |
|---|---|---|
| Thermocouple x4 | Drying / pyrolysis / oxidation / reduction zones | Zone temperature monitoring |
| Pressure transducer | Gasifier outlet | Draft balance |
| Pressure transducer | Post-filter | Filter blockage detection |
| Pressure transducer | Pre-mixer | Mixing ratio stability |
| Differential pressure | Across filter bed | Filter condition (rising ΔP = cleaning needed) |
| Thermal mass flow | Gas outlet | Volumetric flow, quality indicator |
| CO sensor | Gas stream | Direct fuel quality (MQ-7 or Figaro) |
| H₂ sensor | Gas stream | Secondary fuel quality (MQ-8) |
| O₂ sensor | Pre-engine gas stream | Safety — should read near zero |
| Tar dewpoint temp | After gas cooler | Condensation prevention |
| Flame sensor | Combustion zone | Gasifier active confirmation |
| Hopper level | Fuel hopper | Low fuel warning |

### Gasifier Zone Temperature Targets

| Zone | Target Temperature |
|---|---|
| Drying | 150–300°C |
| Pyrolysis | 300–600°C |
| Oxidation | 900–1200°C |
| Reduction | 700–900°C (critical) |

If the reduction zone drops below ~650°C, gas quality deteriorates rapidly — CO drops, tar increases, and effective octane falls.

### Forced Induction Additional Sensors

| Sensor | Purpose |
|---|---|
| Pre-turbo pressure | Inlet restriction monitor |
| Post-turbo pressure | Boost level |
| Post-intercooler temp | True charge temp (IAT location) |
| Post-intercooler pressure | Intercooler pressure drop |
| Pre-turbine EGT | Primary combustion reference |
| Post-turbine EGT | Turbo efficiency monitor |
| Turbo oil pressure | Bearing protection |
| Turbo oil temperature | Coking prevention |

### Safety Sensors

| Sensor | Threshold | Action |
|---|---|---|
| CO leak detector (cabin) | Alarm 50 ppm / shutoff 100 ppm | Operator safety — mandatory |
| O₂ pre-engine | Above 1% = air leak | Fire/explosion risk — shutoff |
| Flame sensor | Flame absent | Close syngas valve |
| EGT high limit | Above 950°C | Auto-revert to gasoline |

---

## 6. Ignition Timing

Wood gas has a slower flame front (~0.5 m/s vs ~0.8 m/s for gasoline) and high octane resistance to detonation. Both factors require significant ignition advance.

### Suggested Timing Maps

#### Gasoline Mode (BTDC degrees)

| RPM | 20 kPa | 40 kPa | 60 kPa | 80 kPa | 100 kPa |
|---|---|---|---|---|---|
| 1000 | 15 | 12 | 10 | 8 | 6 |
| 2000 | 25 | 22 | 18 | 14 | 10 |
| 3000 | 32 | 28 | 24 | 20 | 15 |
| 4000 | 35 | 30 | 26 | 22 | 17 |
| 5000 | 36 | 32 | 28 | 24 | 19 |

#### Wood Gas Mode (BTDC degrees — approximately 8–15° more advance)

| RPM | 20 kPa | 40 kPa | 60 kPa | 80 kPa | 100 kPa |
|---|---|---|---|---|---|
| 1000 | 23 | 20 | 18 | 16 | 14 |
| 2000 | 33 | 30 | 26 | 22 | 18 |
| 3000 | 40 | 36 | 32 | 28 | 23 |
| 4000 | 43 | 38 | 34 | 30 | 25 |
| 5000 | 44 | 40 | 36 | 32 | 27 |

These are starting point values requiring field tuning with wideband O₂ and knock sensor feedback. Exact values depend on engine compression ratio, combustion chamber geometry, and actual gas composition.

### Timing — H₂ Content Feedback

```
H₂ content high (15–18%)  → Allow maximum advance
H₂ content low  (8–10%)   → Retard 3–5° from table
H₂ very low               → Retard further + alarm
```

### Compression Ratio Consideration

The high octane rating of wood gas allows higher compression ratios than gasoline:

```
Gasoline engine:      8:1  – 11:1
Wood gas optimal:    11:1  – 13:1

Higher compression partially compensates for
lower energy density of wood gas mixture.
```

---

## 7. Dual Fuel Map Switching

### Fuel Mode Combinations

The system supports multiple fuel configurations:

| Mode | Description |
|---|---|
| Gasoline only | Standard operation, cold start |
| Wood gas only | Full syngas operation when gasifier is hot |
| Gasoline + syngas blend | Partial syngas substitution |
| Flexfuel (ethanol/gasoline) + syngas | Ethanol blend with syngas supplement |

Flexfuel/syngas is viable because ethanol's high octane (RON 108) complements wood gas well. The ethanol flex sensor (or ethanol content calculation from wideband feedback) provides the base fuel correction, with the syngas VE table layered on top.

### Switch Conditions

```
ALLOW SYNGAS MODE when:
  CLT  > 80°C          Engine fully warm
  IAT  < 50°C          Gas cooled sufficiently
  MAP  stable          Gasifier pressure stable
  EGT  < 900°C         Not running lean
  TPS  < 80%           Avoid full load initially
  Flame sensor active  Gasifier confirmed running
  O₂ pre-engine < 1%  No air leak in gas system

FORCE RETURN TO GASOLINE when:
  CLT  < 70°C
  EGT  > 950°C
  MAP  drops suddenly  Gasifier fault
  RPM  hunts > ±200    Unstable gas quality
  CO alarm triggered
  Flame sensor inactive
```

### Megasquirt Implementation

```
Digital GPIO input → Fuel Table A (Gasoline)
                   → Fuel Table B (Wood gas)

Map switch triggers:
  VE table          A → B
  Ignition table    A → B
  AFR target table  A → B
  Boost target      A → B (higher on wood gas)
  Injector cut/reduction (gas delivered via mixer)
```

### rusEFI Lua Script Logic (Example)

```lua
if (getCoolantTemp() > 80
    and getIAT() < 50
    and getInputPin(FLAME_SENSOR) == 1
    and getInputPin(O2_PRESENSOR) < 1.0
    and getInputPin(SYNGAS_SWITCH) == 1) then

    setFuelTable(SYNGAS_TABLE)
    setIgnTable(SYNGAS_IGN_TABLE)
    setBoostTarget(SYNGAS_BOOST)
    setInjectorOutput(OFF)
    setOutputPin(SYNGAS_VALVE, ON)

else
    setFuelTable(GASOLINE_TABLE)
    setIgnTable(GASOLINE_IGN_TABLE)
    setBoostTarget(GASOLINE_BOOST)
    setInjectorOutput(ON)
    setOutputPin(SYNGAS_VALVE, OFF)
end
```

---

## 8. Forced Induction

### Power Recovery Estimates

```
Naturally aspirated gasoline:          100%
Naturally aspirated wood gas:           55%
Wood gas + 0.7 bar boost:              78%
Wood gas + 1.0 bar boost:              88%
Wood gas + 1.2 bar + intercooler:      95%+
```

With consistent H₂ output from Alnus incana and proper intercooling, 95%+ power recovery is realistic.

### Turbocharger vs Supercharger

| Parameter | Turbocharger | Supercharger |
|---|---|---|
| Power source | Exhaust energy (free) | Engine driven (parasitic) |
| Throttle response | Lag at low RPM | Immediate |
| Complexity | Higher | Lower |
| Charge heating | Significant | Moderate |
| Intercooler | Required | Recommended |
| Stationary use | Excellent | Excellent |
| Electric supercharger option | No | Yes |

Wood gas produces higher exhaust temperatures due to its slower burn rate — this actually drives the turbo efficiently, making turbocharging self-compensating for the power loss.

### Boost Targets by Fuel Mode

```
GASOLINE MODE:
  Boost limit:  0.5 – 0.8 bar
  Timing:       Standard gasoline map

WOOD GAS MODE:
  Boost limit:  0.8 – 1.2 bar  (high octane allows more)
  Timing:       Advanced wood gas map
  At 1.5 bar+:  Requires careful tuning and monitoring
```

### Charge Path (Turbo Installation)

```
[Air intake]
     ↓
[Turbo compressor]
     ↓
[Intercooler]          ← IAT sensor here
     ↓
[Wood gas mixer]       ← Gas introduced POST intercooler
     ↓
[Throttle body]
     ↓
[MAP sensor]           ← Reads boost pressure
     ↓
[Engine]
     ↓
[EGT sensor]           ← Pre-turbine
     ↓
[Turbo turbine / Wastegate]
```

Gas is introduced after the intercooler to prevent tar condensation in the intercooler core and to avoid CO in pressurized leak points.

### EGT Targets with Turbo

| Location | Target Range | Notes |
|---|---|---|
| Pre-turbine | 650–850°C | Primary tuning reference |
| Post-turbine | 200–400°C lower | Large delta = turbo efficient |

### Electronic Boost Control

```
Megasquirt PWM output → boost solenoid → wastegate

Safety limits:
  MAP absolute limit   → fuel cut if overboost
  IAT high limit       → reduce boost
  EGT high limit       → reduce boost
  Knock sensor active  → retard timing + reduce boost
```

---

## 9. Gasifier Agitation System

### The Problem

Without agitation, wood char and ash in the reduction zone will:
- **Bridge** — arch formation blocking gas flow
- **Clinker** — fused solid masses (minimal with Alnus incana)
- **Channel** — preferential gas paths bypassing reduction zone
- **Pack** — compaction increasing pressure drop

Historical note: WW2 vehicle gasifiers required no active agitation because cobblestone and dirt roads provided continuous passive vibration. Modern smooth roads and stationary installations require active stirring.

### Agitation Intervals

| Operating Condition | Suggested Interval |
|---|---|
| Stationary generator | Every 20–30 min |
| Smooth road vehicle | Every 15–20 min |
| Rough terrain vehicle | Every 40–60 min |
| High ash wood | More frequent |
| Alnus incana (low ash) | Less frequent |

### Drive System — Chain Coupled 12V Motor

Chain drive is the correct choice for this environment:

```
Direct coupling  → Heat transfers to motor bearings
Belt drive       → Tar vapor destroys rubber belts
Gear drive       → Requires sealed housing against ash
Chain drive      → Tolerates heat, tar, ash, misalignment
                   Easy to replace, universally available
```

### Motor Selection

| Parameter | Target |
|---|---|
| Voltage | 12V DC |
| Type | Worm gear motor (self-locking) |
| Motor RPM | 30–150 RPM |
| Output RPM | 1–5 RPM at stirrer |
| Torque | 5–15 Nm at output shaft |

Suitable motors: 12V worm gear motor, wheelchair motor, windscreen wiper motor, winch motor with gearbox.

Worm gear motors are preferred — self-locking prevents back-driving when stopped, built-in reduction reduces chain ratio required.

### Chain Specification

| Chain | Pitch | Recommendation |
|---|---|---|
| #25 | 1/4" | Light duty — marginal |
| #35 | 3/8" | Good |
| **#40** | **1/2"** | **Recommended** |
| #50 | 5/8" | Overkill |

Standard bicycle/motorcycle tooling fits #40 chain.

### Drive Ratio Example

```
Motor output:      60 RPM (after worm gearbox)
Target stirrer:    2–3 RPM
Drive sprocket:    10 tooth
Driven sprocket:   40 tooth
Additional ratio:  4:1
Final output:      15 RPM → adjust gearbox ratio
```

### Motor Placement and Thermal Isolation

```
[Gasifier — HOT]
      ↓
[Stirrer shaft — graphite bushing at wall]
      ↓  300–500mm chain run minimum
[Driven sprocket]
      ↓
[Drive sprocket]
      ↓
[Worm gearbox — outside heat zone]
      ↓
[12V DC motor — cool zone]
```

### Bearing at Gasifier Wall

| Bearing Type | Suitability |
|---|---|
| Standard ball bearing | Poor — heat and ash |
| Graphite bushing | Excellent — self-lubricating, 400°C+ |
| Ceramic bearing | Excellent — heat immune |
| Bronze bushing (oilite) | Good to ~150°C |

Graphite bushing is the practical first choice.

### Chain Lubrication

| Lubricant | Max Temp | Notes |
|---|---|---|
| Standard chain oil | ~80°C | Too low |
| High temp grease | ~150°C | Marginal |
| Graphite dry lube | 500°C+ | Ideal |
| Molybdenum disulfide (MoS₂) | 400°C+ | Excellent |

Use MoS₂ paste or graphite dry spray — no wet oil that attracts ash and forms abrasive paste.

### Control Logic

```
TIMER CONTROL (simple):
  ON:  30 seconds
  OFF: 20 minutes
  Relay rated for motor stall current

SUPERVISOR OVERRIDE TRIGGERS:
  Differential pressure high  → immediate stir
  Gas flow drop > 15%         → immediate stir
  Reduction zone temp drop    → immediate stir
  Manual button               → immediate stir

JAM DETECTION (INA219 current sensor):
  Normal current:   2–3A
  Stall current:    8–15A → stop → reverse 2s → retry → alarm
```

---

## 10. System Architecture

### Two-Tier Control

```
TIER 1 — Megasquirt / rusEFI ECU
  Manages: TPS, MAP, IAT, CLT, O₂, EGT, knock, baro
  Real-time engine fueling and ignition
  Boost control via wastegate solenoid
  Fuel mode switching via GPIO

TIER 2 — Raspberry Pi / ESP32 Supervisor
  Manages: Gasifier temps, gas quality, pressures,
           safety sensors, hopper level, oil condition,
           agitation timer and override
  Sends mode-switch permission to ECU via GPIO
  Logs all data to SD card
  Drives dashboard display
  Triggers alarms and shutoffs independently of ECU
```

### Full System Wiring Summary

```
ECU INPUTS:
  TPS              0–5V analog
  MAP              0–5V analog
  IAT              Thermistor
  CLT              Thermistor
  Wideband O₂      0–5V from controller
  EGT x4           0–5V from MAX31855 amplifiers
  Knock sensor     AC signal
  Fuel mode switch Digital GPIO
  Boost solenoid   PWM output

SUPERVISOR INPUTS:
  Gasifier thermocouples x4
  Pressure transducers x3
  Differential pressure
  Gas flow sensor
  CO sensor
  H₂ sensor
  O₂ pre-engine
  Tar dewpoint temp
  Flame sensor
  Hopper level
  CO leak detector
  Motor current sense (INA219)
  Manual agitation button

SUPERVISOR OUTPUTS:
  Stirrer motor relay
  Syngas inlet valve relay
  Emergency shutoff relay
  Mode switch GPIO → ECU
  Alarm outputs
  Display / dashboard
```

### Recommended Hardware Stack

| Component | Choice |
|---|---|
| ECU firmware | rusEFI (Lua) or MS3/Extra |
| ECU hardware | Speeduino (budget) / MS3Pro (robust) |
| Wideband controller | Innovate LC-2 or AEM 30-0300 |
| EGT amplifiers | MAX31855 per cylinder |
| Supervisor | Raspberry Pi 4 or ESP32 |
| Gas sensors | MQ-7 (CO), MQ-8 (H₂) |
| Pressure sensors | MPX5010 (low range) |
| Combined ambient | BME280 (temp/humidity/baro) |
| Tuning software | TunerStudio MS |
| Data logging | MegaLogViewer or rusEFI logger |

---

## 11. EGT Operating Ranges

| EGT Range | Condition | Action |
|---|---|---|
| Below 650°C | Rich or misfiring | Lean mixture / check gasifier |
| 650–800°C | Ideal zone | Normal operation |
| 800–950°C | Approaching lean limit | Enrich mixture |
| Above 950°C | Dangerously lean | Auto-revert to gasoline |

---

## 12. Notes and Operational Summary

- Wood gas is a viable engine fuel with effective RON 100–120, making it genuinely turbo-compatible without knock risk at moderate boost pressures
- Alnus incana (grey alder) is an excellent feedstock choice — low ash reduces agitation frequency and clinker formation, low tar reduces engine fouling risk
- Gasoline/syngas blending and flexfuel (ethanol blend)/syngas combinations are viable — ethanol's high octane complements wood gas well
- The system requires active agitation on smooth roads and stationary installations — a timer-controlled 12V chain-driven worm gear motor is the practical solution
- Two-tier control separates engine-critical decisions (ECU) from system-level safety and gasifier management (supervisor)
- All timing values require field tuning with wideband O₂ and knock sensor feedback — tables provided are starting points
- Oil change intervals must be shortened significantly on wood gas operation due to accelerated contamination from combustion byproducts
- CO from wood gas is odorless and lethal — a dedicated electrochemical CO detector with automatic shutoff is mandatory for any vehicle installation
