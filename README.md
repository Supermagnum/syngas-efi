# Wood Gas / Syngas EFI Dual Fuel System
### Complete Design and Engineering Reference

## Disclaimer

This repository and its documentation are provided **for informational purposes only**. The **author assumes no responsibility or liability** for **underperforming systems**, **fire or other property damage**, **engine or equipment damage**, **personal injury**, or **any other loss** arising from the use, misuse, or reliance on this material.

**Use is entirely at your own risk.** You are **solely responsible** for safe **design**, **construction**, **testing**, **operation**, and **maintenance**, and for obtaining **qualified professional advice** where appropriate. **Biomass gasification, producer gas, and dual-fuel engine work involve intrinsic hazards** (toxic and flammable gases, high temperatures, moving machinery, and pressure systems).

**Applicable laws, regulations, emissions rules, registration requirements, and workplace or environmental obligations** vary by jurisdiction. **You must verify and comply with all local and national requirements** and **consult the relevant authorities** before building, modifying, or operating any system described or inspired here.

## Table of contents

Anchors `sec-community-forums`, `sec-theory`, `sec-theory-vehicle`, and `sec-01` … `sec-18` are explicit HTML IDs placed immediately before those chapter headings so intra-document links stay reliable in GitHub rendering.

| Ch. | Section |
|:---:|:---|
| [D](#sec-community-forums) | Community forums and recent examples |
| [T](#sec-theory) | Theory of operation |
| [Tv](#sec-theory-vehicle) | Theory — vehicle platform, induction, and conversion strategy |
| [1](#sec-01) | Fuel Properties |
| [2](#sec-02) | Octane Rating |
| [3](#sec-03) | Preferred Feedstock — Alnus incana (Grey Alder) — incl. feedstock cube size standard |
| [4](#sec-04) | EFI System — Megasquirt / Open Source ECU |
| [4a](#sec-04-gasifiers) | Vehicle producer-gas (syngas) sources (verified links) |
| [4b](#sec-04-mixers) | Gas mixers and actuators (EFI-oriented, 12 V DC) |
| [5](#sec-05) | Sensor Suite |
| [6](#sec-06) | Ignition Timing |
| [7](#sec-07) | Dual Fuel Map Switching |
| [8](#sec-08) | Forced Induction |
| [9](#sec-09) | Gasifier Agitation System |
| [10](#sec-10) | System Architecture |
| [11](#sec-11) | Gasifier Warmup and Fuel Switch Procedure |
| [12](#sec-12) | Engine Breather and Oil Catch Tank |
| [13](#sec-13) | Final Filtration System |
| [14](#sec-14) | Alternative Feedstocks — Locally Available Materials |
| [14b](#sec-14b) | Regional Feedstock Alternatives — International |
| [14c](#sec-14c) | Invasive and Weed Species — Free Fuel From Problem Plants |
| [15](#sec-15) | Fuel Briquette Production — Converted Hydraulic Log Splitter |
| [16](#sec-16) | EGT Operating Ranges |
| [17](#sec-17) | Notes and Operational Summary |
| [18](#sec-18) | Commercial Systems, DIY Plans and Salvage Materials |

---

<a id="sec-community-forums"></a>
## Community forums and recent examples

It is **strongly advisable** to **post your plans** and **ask for advice** on **active forums** where **experienced builders** can comment on **sizing**, **safety**, **controls**, and **filtering**. This README is **not** a substitute for **peer review** of a concrete design.

**Primary English-language hub**

- [Drive On Wood — forum](https://forum.driveonwood.com/) — vehicle **gasoline / wood gas** projects, operating experience, and troubleshooting.  
- [Drive On Wood — main site and library](https://www.driveonwood.com/) — articles, historic **GENGAS** material, and links from **[§18](#sec-18)**.

**Megasquirt EFI tuning**

- [Megasquirt forum — introduction / how to post](https://www.msextra.com/forum-info/) — MS1/MS2/MS3 support areas, **MSQ** and **datalog** attachments.  
- [Megasquirt support forums](https://www.msextra.com/forums) — primary user support (dual-fuel and gaseous-fuel threads vary by section; read the sticky guidance first).

**Other communities** (search for **current** traffic; treat **old** mailing-list archives as **historical** context only.)

**Recent public write-ups and threads (about the last five years)** — **illustrative only**, not vetted by this repository; **gasoline** engines and **producer gas** appear in different ways (**dual-fuel** vs **wood-only** after warm-up); read each source in full.

| Approx. date | Description | Link |
|---|---|---|
| 2026-04 | Syndicated **press** piece on a **Chevrolet** small-block truck running **long distances** on **wood gas** (claims and mileage in the article; trace to primary video/channel if needed). | [Yahoo Autos / syndicated story](https://autos.yahoo.com/classic-and-collector/articles/1972-chevy-truck-traveled-over-170034061.html) |
| 2024-12 | Forum: **Converting engines**, possibly to **wood gas** — planning and community input. | [Drive On Wood topic](https://forum.driveonwood.com/t/converting-engines-maybe-to-wood-gas/8064) |
| 2023-10 | Forum: **International Harvester 392** V8 **truck** gasifier project thread. | [Drive On Wood topic](https://forum.driveonwood.com/t/ih-392-truck-gasifier/7539) |
| 2022-06 | Forum: **Gasification planning** (includes **MPFI** trucks and highway goals). | [Drive On Wood topic](https://forum.driveonwood.com/t/gasification-planning/6725) |
| 2022-04 | Forum: **2009 Ford F-150** **4.6 L V8** on **wood gas** — build and operation discussion. | [Drive On Wood topic](https://forum.driveonwood.com/t/2009-f150-4-6l-v8-on-wood-gas/6619) |

---



<a id="sec-theory"></a>
## Theory of operation

This section summarizes **how the engine ECU meters fuel in each mode** and **how the gasifier produces clean-enough gas**, so the detailed chapters later fit into one mental model.

### EFI in gasoline and flex-fuel modes

In **gasoline-only** operation the ECU runs like any modern **port-fuel-injected** spark engine. For **speed-density** (the model emphasized in this document), the ECU estimates air mass entering the cylinders from **RPM**, **manifold absolute pressure (MAP)**, **throttle position (TPS)**, **intake air temperature (IAT)**, and often **barometric pressure** (key-on, internal MAP, or a dedicated baro sensor such as **MapDaddy** on Megasquirt). It looks up or calculates a **volumetric efficiency (VE)** or equivalent fuel map, commands **injector pulse width**, and trims using **closed-loop lambda** when a wideband or narrowband **O₂** sensor is enabled. **Ignition timing** comes from a separate map with knock and temperature protections.

**Flex-fuel** adds an **ethanol content** input (flex sensor or inferred ethanol percentage). The ECU scales **fueling**, **cold-start**, **acceleration enrichment**, and sometimes **spark** with ethanol fraction because stoichiometry and latent heat differ between gasoline and ethanol. The **air side** is still inferred from MAP/TPS/IAT (and baro), not from fuel composition alone.

Throughout, **liquid fuel is injected** in the intake port; the **throttle** controls air flow; **MAP** reflects load.

### EFI in producer-gas (syngas / wood gas) mode

**Producer gas** is already a **low heating value, gaseous** mixture drawn from the gasifier train. It is **not** metered with gasoline injectors in the usual way. Typical dual-fuel setups **introduce gas through a mixer / venturi / proportional valve** ahead of the throttle (for example **post-intercooler** on a turbo build; see **Section 8**). **Gasoline injectors** are **turned off** or used only for **pilot**, **cold start**, or **fallback**.

The ECU still uses **MAP, TPS, IAT**, and **RPM** for **speed-density** because the **air path** is the same: the throttle sets **charged air mass**, and the **mixer** sets how much **producer gas** accompanies that air. Separate **VE (or fuel) tables** and **AFR targets** are calibrated for **wood gas mode**. A **wideband** remains essential for tuning and closed-loop trim. **Ignition** maps are usually **advanced** relative to gasoline because flame speed and knock behavior differ.

Because producer gas **composition and humidity** drift with gasifier state, **MAP-based** control is preferred over a **MAF** (mass airflow) calibrated for a fixed gas type. The **supervisor** layer may use extra sensors (CO, H₂, pressures, filter ΔP) for safety and quality; the **ECU** still closes the loop on **lambda** where possible.

Mode switching between **gasoline/flex** and **syngas** is gated by **temperature, quality, and safety** checks (see **Section 7**).

### How the downdraft gasifier works (component roles)

A **fixed-bed downdraft** gasifier for engine fuel processes **solid biomass** in stacked **zones** (see **Section 5** for typical temperature bands):

1. **Drying** — Free and bound moisture is driven off as steam. Moisture wastes heat and lowers gas quality; excessive moisture increases **tar** in the raw gas.
2. **Pyrolysis** — Heat without enough oxygen breaks biomass into **volatiles** (tars, light hydrocarbons) and **char**. Tars are unavoidable; the job of the design is to **crack** or remove them before the engine.
3. **Oxidation (combustion)** — A controlled **air or oxygen** supply burns a fraction of the char and volatiles, releasing heat that sustains the process at **high temperature**.
4. **Reduction** — Hot **CO₂** and **steam** react with **carbon** in the char (**Boudouard** and **water–gas** reactions), producing **CO** and **H₂**, the main **combustible** components. **Nitrogen** from air passes through largely **inert**, diluting the product—expected for air-blown systems.

**Hopper, grate, nozzles, throat, and insulation** define **flow paths**, **temperature profile**, and **residence time**. If the **reduction** zone **cools** or channels, **CO/H₂** drop and **tar** rises. **Agitation** (see **Section 9**) fights bridging, clinker, and channeling so reduction stays active.

### Why cooling and filters are required

Raw **producer gas** carries **fine char**, **ash**, **acid and water aerosols**, and **tar** as **vapor** at temperature. **Tar** that stays vapor passes through coarse stages; when the gas **cools**, tar **condenses** into **droplets** that can be **separated** mechanically.

Filtration and cleaning exist to protect the **engine and mixer**:

- **Solids** (dust, char) **abrasive** to valves, seats, and **turbo** wheels.
- **Tar** **plugs** small passages, **sticks** valves, and **fouls** spark plugs and rings.
- **Acids and water** from condensate **corrode** plumbing and raise **maintenance**.

Hence the usual chain (**Section 13**): **coarse separation** while hot, **cooling** to condense tar, **cyclone/coalescer** stages, then a **final fine filter** as the **last** element before the **mixer**, after all intended cooling—so condensate is managed upstream of the engine.

<a id="sec-theory-vehicle"></a>
### Vehicle platform, induction, and conversion strategy

**Pickup trucks** are a strong match for **gasoline / syngas** conversions: **cargo bed** space for the **gasifier, filters, and coolers**; **payload** margin; relatively straightforward **routing** of gas lines; and **cooling airflow** that is easier to manage than in many tightly packaged car engine bays. Many historical and current **woodgas vehicle** builds use **pickups** or similar **utility chassis** for these reasons.

**Turbocharged** engines are **preferred** when the objective is to **recover power** lost to producer gas’s **low volumetric energy** (see **Section 8** effective octane and boost discussion). **Naturally aspirated** builds remain common and valid, but expect **significantly lower output** on wood gas than on gasoline for the same engine—plan **gearing**, **usable grade load**, and driver expectations accordingly.

**Registration, emissions, fuel handling, and road-use rules** depend on **jurisdiction**. **Check applicable law** and **ask the relevant authorities** (motor vehicle agency, environmental agency, workplace safety where relevant) **before** assuming **on-road** or **commercial** use is permitted.

**Converting a carbureted engine to EFI** then to dual-fuel is **possible**, but **starting from a vehicle that already has port EFI** usually **saves time and effort**: **MAP**, **TPS**, **injectors**, and **return-style fuel plumbing** are already present; you extend with **mode switching**, **gas train**, and **mixer** hardware rather than replacing the whole **air and fuel metering** architecture. **Less plumbing** is typically required than for a **carb → EFI → syngas** sequence.

**Recommended sequence:** bring the vehicle to a **stable, verified baseline on gasoline** (cold start, idle, part-load, WOT, **lambda** on target, no unresolved misfire or knock) **before** commissioning **producer gas**. Diagnose **one fuel at a time**.

**Mechanical baseline:** treat dual-fuel as a **reliability** project. Start with **good** engine health: **fresh oil**, **new filters** (engine oil, air, fuel), **belts**, **coolant hoses**, a sound **water pump**, **alternator**, and **battery**. Latent **vacuum leaks**, compression loss, cooling faults, or weak ignition that are **masked** on gasoline will dominate troubleshooting once syngas is added.

### External references — gasifier theory, design, and gas cleanup

| Document | Link | Notes |
|---|---|---|
| FAO — *Wood gas as engine fuel* (Forestry Paper 72, full HTML) | [Contents / start](https://www.fao.org/4/t0512e/t0512e00.htm) | Theory, gasifier types, fuel specs, engine fueling, hazards, cleaning. |
| FAO — Chapter 2 (small wood and charcoal gasifiers for engines) | [Chapter 2](https://www.fao.org/4/T0512E/T0512e07.htm) | Suited to engine-scale hardware. |
| FAO — §2.5 (design of downdraught gasifiers) | [§2.5](https://www.fao.org/4/t0512e/T0512e0c.htm) | Downdraft zones and layout in depth. |
| Reed & Das — *Handbook of Biomass Downdraft Gasifier Engine Systems* (NREL/SP-271-3022) | [NREL Research Hub](https://research-hub.nrel.gov/en/publications/handbook-of-biomass-downdraft-gasifier-engine-systems/) | Practical handbook: design, testing, measurement, cleanup, engine systems. |
| Same handbook — digitized copy | [UNT Digital Library](https://digital.library.unt.edu/ark:/67531/metadc1061385/) | PDF and viewer options. |
| Same handbook — OSTI catalog | [OSTI biblio 5206099](https://www.osti.gov/biblio/5206099) | Metadata and document access path. |
| Drive On Wood — GENGAS / historical collection | [GENGAS library](https://www.driveonwood.com/library/gengas/) | Swedish program documentation and related scans. |
| GEK Gasifier Wiki | [wiki.gekgasifier.com](http://wiki.gekgasifier.com/) | Community build and theory notes; if **HTTPS** fails in the browser, use this **HTTP** link (TLS hostname issues on some setups). |

---

<a id="sec-01"></a>
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

<a id="sec-02"></a>
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

<a id="sec-03"></a>
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

### Feedstock Cube Size — 100 × 100 mm Standard

Grey alder is cut and split to **100 × 100 mm cubes** (approximately 4 × 4 inches) as the standard feedstock form for the engine displacement range this document targets. This size is not arbitrary — it is determined by four interacting requirements of the downdraft gasifier.

**1 — Throat geometry and bridging**

The cube size must be matched to the gasifier throat diameter. Pieces that are too large bridge above the throat, blocking gas flow. The 100 mm cube suits the throat dimensions appropriate for 1.2–4 litre engine gasifiers. Smaller pieces increase bed packing density, raise pressure drop across the reduction zone, and promote channelling — preferential gas paths that bypass the reduction zone and allow tar to reach the engine uncracked.

**2 — Reduction zone residence time**

A 100 mm cube takes long enough to gasify completely that the reduction zone has adequate time to perform the Boudouard and water-gas reactions — converting CO₂ and steam to CO and H₂, and cracking tar vapour. Pieces significantly smaller than 80 mm fall through the reduction zone too quickly, lowering gas quality and increasing tar output.

**3 — Hopper flow**

Uniform cube geometry feeds predictably under gravity. Irregular shapes, mixed sizes, and pieces with large flat faces cause hopper bridging — a suspended arch that blocks material flow without any visible warning. Consistent 100 mm cubes eliminate this failure mode. The log splitter forming box (Section 15) produces this geometry directly from loose material.

**4 — Moisture uniformity**

A 100 mm cube dried to below 15% surface moisture has a core moisture close enough to the surface moisture that performance is consistent. Larger pieces — 150 mm and above — can have a dry exterior masking a wet core. That hidden moisture enters the gasifier as steam, consumes heat in the drying zone, and increases tar yield unpredictably. The 100 mm size is the practical upper limit for reliable through-drying in open air storage.

### Size Tolerance and Acceptable Variation

```
Target:          100 × 100 mm (4 × 4 inch) cross-section
                 Length: 80–150 mm acceptable

Minimum:         No dimension below 60 mm
                 Fines and small pieces increase bed resistance
                 and can be carried into the gas stream

Maximum:         No dimension above 150 mm
                 Bridging and incomplete gasification risk
                 above this size

Irregular pieces: Acceptable if they fit within the
                  60–150 mm envelope
                  Avoid large flat slabs — bridging risk

Mixed sizes:     Acceptable within the 60–150 mm window
                 Consistent sizing always preferred
```

### Applying the 100 mm Standard to Other Feedstocks

The 100 mm cube standard applies to all solid feedstocks fed directly to the hopper. Briquetted materials from the log splitter press (Section 15) should target the same 100 × 100 mm cross-section using the forming box dimensions specified there.

| Feedstock | Preparation to 100 mm standard |
|---|---|
| Grey alder logs | Split and cut to 100 × 100 mm cubes — chainsaw cross-cut, then log splitter or axe |
| Hardwood logs (oak, hickory, black locust) | Same — split to 100 mm cross-section, cut to length |
| Wood pellets | Already small — no sizing needed; feed directly |
| Briquettes (newspaper, cardboard, straw) | Target 100 × 100 mm forming box — see Section 15 |
| Wood chips | Acceptable if chips are 40–100 mm — screen out fines below 20 mm |
| Corn cobs | Natural size typically 80–150 mm — feed whole or halved |
| Coconut shells | Halved or quartered to 80–120 mm pieces |
| Charcoal | Lump charcoal 40–80 mm preferred — screen out dust and fines |

---

<a id="sec-04"></a>
## 4. EFI System — Megasquirt / Open Source ECU

### Recommended ECU Platforms

| Platform | Suitability | Notes |
|---|---|---|
| MS2/Extra | Good | Solid dual-fuel, wide community |
| MS3/MS3Pro | Best | More I/O, better table resolution |
| Speeduino | Good | Open source hardware, Arduino-based |
| rusEFI | Excellent | Lua scripting, very active development |

TunerStudio MS is the tuning and logging software for all platforms.

### Megasquirt documentation, support forum, and propane (LPG)

- **Manual contents (legacy HTML mirror):** [Megasquirt manual — table of contents / `mtabcon.htm`](http://megasquirt.free.fr/sources/MS/manual/mtabcon.htm) — entry point into the classic **Megasquirt** documentation set on this mirror; newer boards may also need vendor-specific wiring and the current **M** or **Extra** manual for your processor.
- **Official support forum — how to post:** [Megasquirt forum introduction](https://www.msextra.com/forum-info/) — sections for **MS1/MS2/MS3**, **TunerStudio**, attaching **MSQ** and **datalogs**; live forum: **[msextra.com/forums](https://www.msextra.com/forums)**.

**Propane (LPG):** Megasquirt firmware can control **spark-ignition** engines on **propane** (separate **VE / fuel** strategy from gasoline), but you still need **correct gaseous-fuel hardware**: **vapor lockoff**, **regulator / zero governor** as applicable, **mixer or injectors**, and **fault-safe** plumbing consistent with local rules. The same **air–gas mixing** considerations described for **producer gas** in **§4** (mixer before throttle, **MAP**-based load) apply in principle; **tar** and **wood gas** filtration are replaced by **LPG**-appropriate **codes and leak checks**.

### Why MAP-Based Speed Density

Wood gas composition varies with feedstock and gasifier conditions. A MAF sensor would constantly mis-meter the diluted mixture. MAP + TPS + IAT speed-density fueling is fuel-agnostic by nature:

```
Fuel delivery = f(RPM, MAP, IAT correction, CLT correction)
```

Separate VE tables are built for each fuel mode. Megasquirt handles this natively.

For **Megasquirt** builds that need **continuous altitude / weather correction** while driving (not only a **key-on** baro snapshot), the **MapDaddy** upgrade from **DIYAutoTune** replaces the board’s MAP sensor footprint with a module that combines a **4 bar MAP** die and a separate **barometric** sensor. **MS2** and **MS3** firmware can use that **real-time barometric** input for live fueling correction. Wiring is board-revision specific (for example **JS5** on V3.x, **X7** on V2.2); follow the vendor [MapDaddy documentation](https://diyautotune.com/support/original-mapdaddy-documentation).

<a id="sec-04-gasifiers"></a>
### Vehicle producer-gas (syngas) sources — verified links

**Syngas for a vehicle ICE is made in a gasifier** (solid fuel to producer gas). **Air–gas mixers, venturis, and EFI plumbing** are downstream of that; see **Section 8** (charge path and forced induction) for how gas meets the throttle body.

The URLs below were checked with **`curl -L`** and returned **HTTP 200** from this environment. If a host later returns errors, use the **Internet Archive** or the site’s current landing page. **HTTPS** is preferred; one supplier only served a valid certificate on **HTTP** at the time of checking (`offgrid48.com`).

| Resource | Link | Vehicle relevance |
|---|---|---|
| Drive On Wood — community | [driveonwood.com](https://www.driveonwood.com/) | Central US hub for **pickup/truck woodgas** builds, forums, and operating experience. |
| Drive On Wood — plan library | [Free gasifier plans](https://www.driveonwood.com/library/free-gasifier-plans/) | Downloadable documentation for **self-built** vehicle-scale gasifiers. |
| Thrive Off Grid | [thriveoffgrid.net](https://www.thriveoffgrid.net/) | Commercial **charcoal / wood** gasifier kits often used in **mobile** and pickup-class projects (confirm rating vs your engine). |
| Thrive Off Grid — CXF series | [CXF Crossfire gasifier](https://www.thriveoffgrid.net/cxfcrossfiregasifier) | Product line page for a common **kit** form factor. |
| All Power Labs — GEK kits | [Gasifier kits](https://www.allpowerlabs.com/products/gasifier-kits) | Packaged **gas-making** systems; **vehicle installs** appear in public project write-ups—validate packaging and mass for **road** use yourself. |
| OffGrid48 — DIY kits | [DIY wood gasifier kits](http://www.offgrid48.com/diy-wood-gasifier-kits.html) | **HTTP** link: HTTPS for this host failed certificate verification here; laser-cut kits sized for **small-engine** projects that are often adapted to **vehicle** frames. |
| FEMA wood gas generator (1989) | [Internet Archive PDF](https://archive.org/download/femasimplifiedwoodgasgeneratormar1989withbiomassenergyfoundation2001/FEMA_Simplified_Wood_Gas_Generator-Mar_1989_With_Biomass_Energy_Foundation_2001_text.pdf) | Historical **emergency / convoy** downdraft design and **carburetion** chapter—documentation, not a vendor; intake layout differs from modern **EFI** routing. |

**Not in this table:** natural-gas/LPG **mixer** vendors, **stationary CHP** gasifiers (see **Commercial Kits and Systems — Suppliers** later in this document), or links that **404**’d or **TLS-failed** here (e.g. some third-party IMPCO PDF mirrors).

<a id="sec-04-mixers"></a>
### Gas mixers and actuators (EFI-oriented, 12 V DC)

These are **downstream of the gasifier**: they meter **producer gas and air** (or drive a **common throttle / mixture path**) while a **MAP/TPS/IAT** EFI strategy (above) supplies load information. Catalog parts are usually spec’d for **natural gas / propane / biogas**; **tar-safe** plumbing and filtration from Sections **13** and **8** still apply.

**12 V DC:** Prefer order codes or modules explicitly rated **12 V** (or **12–24 V**). Many industrial gas-engine trains use **24 V** only—use a suitable **DC–DC converter** from the vehicle **12 V** bus only after you match **noise, isolation, and fuse** requirements.

| Resource | Link | 12 V DC (verify before buy) | EFI-oriented use |
|---|---|---|---|
| GAC ATB integral throttle bodies | [GAC / Governors America product catalog PDF](https://governorshop.com/wp-content/uploads/2021/02/5ced91bfe8b1186d7fe16e60_PMB7040_Catalog.pdf) | **Yes:** ATB T1/T2 families list **12 or 24 V DC**; part numbers with **`-12`** are the 12 V variants. | Electric actuator drives an integrated plate for **air or air/fuel mixture** on **gaseous-fueled** engines; spring return to low-fuel position when de-energized—interface as you would an **electronic throttle** / position demand from the ECU or governor. |
| GAC AFR200 / FIMS (closed loop) | [Same GAC catalog PDF](https://governorshop.com/wp-content/uploads/2021/02/5ced91bfe8b1186d7fe16e60_PMB7040_Catalog.pdf) (Fuel & Ignition Management section) | **Yes:** AFR200-family description is paired with **12 or 24 V** hardware in the same catalog—confirm on the **specific controller and harness** you order. | **Closed-loop air–fuel** using an exhaust **O₂** probe; complements **wideband λ** or narrowband strategies when converting a **gaseous-fuel train** beside an EFI. |
| Northern Self Reliance — automated mixture | [Wood gas automated mixture control](https://northernselfreliance.com/biomass/woodgas/wood-gas-automated-mixture-control/) | **Vehicle 12 V bus** typical; hobby **servos** often run at **5–6 V** via a BEC or regulator off that bus. | Documents **λ feedback** driving a **mixture valve**—same control pattern as **ECU trim** on a separate gas valve or PWM valve driver. |
| HEINZMANN gas mixers | [Gas mixers](https://heinzmann.com/en/products/gas-mixers) | **Confirm** on datasheet: many site **24 V** actuators; **12 V** may be available on some SKUs or via **DC–DC**. | Commercial **venturi / gas train** mixers for **stationary / industrial** gas engines—useful reference architecture for **dual-fuel** intake design. |
| HEINZMANN electric actuators | [Actuators for gas engines](https://heinzmann.com/en/products/actuators-gas) | **Confirm** supply voltage per **StG** model. | Pair with a **mixer / throttle valve** and ECU or governor output for **position-controlled** gas or mixture control. |
| Woodward — mixer application PDFs | [Publication 03379](https://www.woodward.com/products/publication-asset/03379/), [Publication 51540](https://www.woodward.com/products/publication-asset/51540/) | N/A (documentation). | Application notes for **air–gas mixer** families on SI gas engines; use to cross-check **EFI MAP sampling** vs mechanical **mixer–throttle** layout. |

---

<a id="sec-05"></a>
## 5. Sensor Suite

### Primary Engine Sensors

| Sensor | Purpose | Notes |
|---|---|---|
| TPS | Load input, | Standard |
| MAP | Speed-density fueling, boost monitoring | 0–300 kPa range for turbo; protect reference ports (see **MAP and baro reference line protection** below) |
| IAT | Charge temp correction | Mount post-intercooler |
| CLT | Cold start lockout for syngas mode | Syngas only above 80°C |
| Wideband O₂ (λ) | Closed loop AFR, tuning | Innovate LC-2 or AEM 30-0300 |
| EGT per cylinder | Mixture distribution, lean detection | K-type, one per cylinder |
| Knock sensor | Timing safety, max advance | Closed loop retard |
| Crankcase pressure | Ring wear / blowby monitoring | 0–10 kPa sensor |
| Oil dilution sensor | Tar/fuel contamination in oil | Refractive index or conductivity |
| Barometric pressure | Altitude compensation | Often internal to MAP sensor or key-on only; **Megasquirt:** [MapDaddy](https://diyautotune.com/support/original-mapdaddy-documentation) adds a **live baro** channel for **real-time** correction on **MS2/MS3**; use the same **inline filter** practice on any **dedicated baro hose** |

### MAP and barometric reference line protection

Install **small inline filters** in the **vacuum / boost hoses** that feed the **MAP sensor** and any **separate barometric** reference (for example **MapDaddy** or an external **baro** transducer). **Disposable paper- or fabric-element filters** intended for **low-pressure gasoline EFI** (inline **fuel filters** for small engines or automotive low-pressure lines) are **suitable**: they trap **fine particles** so grit does not reach the sensor’s **restrictor orifice** or internal passages and **clog** them. **Wood gas / syngas** operation increases exposure to **dust, ash, and oil mist** in the intake tract; without a filter, **slow or erratic MAP** readings and **stuck** pressure traces are common failures. Service or replace the element on a fixed interval or when **transient response** degrades; choose a filter size that keeps **pressure lag** acceptable for boost control and tuning.

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

<a id="sec-06"></a>
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

<a id="sec-07"></a>
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

<a id="sec-08"></a>
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

### Why high octane and forced induction recover power

**Octane (knock resistance) does not add chemical energy** to the charge. It **raises the knock limit**, so syngas often allows **more boost, more compression, and/or more ignition advance** than pump gasoline on the same hardware without detonation. That **unlocks** the extra power margin when you add a turbo or supercharger.

**The charger compensates for low volumetric energy.** Producer gas has **much lower heating value per mixture volume** than gasoline; a naturally aspirated cylinderful releases less work per cycle. Increasing **manifold pressure** increases **trapped charge mass** per cycle, which is what pulls output back toward gasoline-like power.

In short: **low energy per volume** calls for **more boosted airflow**; **high effective octane** helps you **use that boost** (and timing) without knock-limited retreat on the engine. This is not automatic: you still need **reliable gas quality**, **intercooling** when boost heats the charge, **correct mixture control**, and appropriate **boost and safety limits**. **Superchargers** also pay **parasitic** shaft work that **turbos** largely avoid (exhaust-driven).

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

<a id="sec-09"></a>
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

<a id="sec-10"></a>
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

<a id="sec-11"></a>
## 11. Gasifier Warmup and Fuel Switch Procedure

### Pre-start inspection (filters, separators, and condensate)

Before every **gasifier startup**, treat the **gas train** as critical path equipment:

- **Inspect filters and separators** (cyclones, coalescers, demisters) **on a regular schedule** and when **ΔP** across the train **rises** or **flow** falls for a given setting. **Plugged** media or **blinded** mesh **starves** the engine of gas and can **pull liquid slugs** if pressure swings.
- **Drain condensate** from **coolers**, **condensers**, **water seals**, and **low-point traps** **before** ignition. **Tar–water and acid condensate** left in sumps can be **entrained** into downstream filters or piping on **first draw**, **fouling** the **final filter** in one shot.
- Keep **spare filter elements** (or a second **cleaned** cartridge / foam set) **in stock**. **Producer gas** loading varies with **feedstock moisture** and **tar**; **field replacement** is faster than an **unplanned** shutdown.

### Warmup Time

A gasifier requires 10–15 minutes from cold start to establish stable thermal zones and produce engine-quality gas. During this period gas quality is poor — high in tar vapor, low in CO and H₂ — and must not be directed to the engine.

### The Classic Method (WW2)

The original operator procedure was elegant and reliable:

```
1. Start gasifier
2. Run 10–15 minutes to build heat zones
3. Open vent valve → syngas bleeds to atmosphere
4. Attempt ignition at vent pipe outlet
5. Flame burns stable and blue/clear → gas is ready
   Flame yellow / weak / unstable   → not ready, wait
6. Close vent valve → open engine valve → start engine
```

Flame color and stability was the quality sensor. A clean blue flame indicated sufficient CO and H₂. A lazy yellow flame indicated tar vapor dominance — not ready.

### Modern Equivalent — Sensor Readiness

The sensor suite formalizes the same judgment:

```
GASIFIER READY — ALL CONDITIONS MUST BE MET:

  Reduction zone temp  > 700°C     Good gas production confirmed
  CO sensor            > threshold  Fuel gas present
  H₂ sensor            > threshold  Gas quality confirmed
  O₂ pre-engine        < 1%         No air contamination
  Gas pressure         stable        Consistent flow
  Tar dewpoint temp    safe          No condensate risk
  Warmup timer         > 10 min      Minimum thermal soak

ALL MET → Green light illuminates → operator switches fuel
```

### Manual Switch Philosophy

The final switch is always manual — the green light is the modern flame test, the operator remains in command:

```
Automatic switching risks:
  Switching on a momentary sensor spike
  Switching before operator is ready
  Switching without operator awareness

Manual switch with green light:
  Operator makes the final decision
  Same human-in-the-loop logic as the original flame test
  Simple, reliable, proven concept
```

### Warmup Phase Display States

| Phase | Time | Light | Display |
|---|---|---|---|
| Ignition | 0–3 min | Red | NOT READY |
| Building | 3–10 min | Amber | WARMING — X min |
| Ready | 10–15 min | Green | READY — SWITCH WHEN PREPARED |
| Syngas active | — | Green steady | SYNGAS ACTIVE |
| Auto reverted | — | Red flash | FAULT — GASOLINE MODE |
| CO alarm | — | Red solid | CO ALARM — SHUTDOWN |

### Warmup Sequence Logic

```
PHASE 1 — Ignition (0–3 min)
  Gasifier lit, all sensors begin logging
  Vent valve open — purges startup gas to atmosphere
  Red light — NOT READY

PHASE 2 — Building (3–10 min)
  Temps rising through all zones
  Gas quality sensors rising
  Amber light — WARMING UP
  Vent valve remains open

PHASE 3 — Ready (10–15 min)
  All ready conditions met simultaneously
  Green light — READY
  Vent valve still open — operator can observe/light vent
  System waits for manual switch

SWITCH EVENT — manual operator action
  Operator flips fuel switch
  Vent valve closes
  Engine syngas valve opens
  ECU receives GPIO → switches to syngas VE/IGN tables
  Injectors cut or reduced
  Full sensor snapshot logged at switch event

REVERT — manual or automatic
  Manual: operator flips switch back
  Auto triggers: EGT > 950°C, CO alarm, flame loss,
                 pressure drop, RPM hunting > ±200 RPM
  Engine valve closes, vent opens
  ECU reverts to gasoline tables, injectors restored
  Green light extinguished — manual re-engagement required
```

### Vent Valve Specification

```
Type:      Solenoid valve, 12V DC
Fail mode: Normally OPEN — fails safe to venting if power lost
Material:  316 stainless body, Viton seals
Size:      Match gas line diameter
Vent pipe: Routed outside / upward — away from operator
           Never vent into enclosed space
```

---

<a id="sec-12"></a>
## 12. Engine Breather and Oil Catch Tank

### Why Syngas Demands a Catch Tank

Wood gas combustion byproducts are significantly dirtier than gasoline. Without a catch tank the PCV system recirculates contaminated blow-by directly onto injector tips and intake valves:

```
Syngas combustion blow-by contains:
  Incomplete combustion residues  → soot, carbon particles
  Tar micro-droplets              → sticky hydrocarbon deposits
  Water vapor                     → condensation in cold oil
  CO traces                       → dissolved in oil
  Char particles                  → abrasive contamination
```

### Why Ethanol Compounds the Problem

```
Ethanol specific issues:
  Higher cylinder wash effect     → strips oil film from bores
  Water absorption                → ethanol attracts moisture
  Ethanol residue in blow-by      → attacks rubber PCV seals
  Oil dilution                    → ethanol dissolves into oil

Injector tip fouling:
  Ethanol + oil mist from PCV
  → deposits bake onto hot injector tips
  → flow restriction and spray pattern distortion
  → rough running, uneven mixture, misfire
```

On a syngas/flexfuel combined system both mechanisms are active simultaneously — a catch tank is mandatory.

### Two-Stage Catch System

```
STAGE 1 — Primary catch tank
  Coarse separation, bulk oil mist and condensate
  Volume: 750–1000 ml
  Labyrinth baffle + stainless mesh pack
  Drain valve at bottom

STAGE 2 — Secondary coalescing filter
  Fine mist separation
  Replaceable filter element
  Last line of defense before intake
  Directly protects injector tips

OPTIONAL STAGE 3 — Inline filter
  Small inline filter on PCV return line
  Very cheap additional insurance
```

### Catch Tank Specification

```
Volume:        750–1000 ml (most engine sizes)
               Oversizing always preferred

Material:      Aluminium — lightweight, heat tolerant
               Avoid plastic — ethanol attacks many types

Seals/hose:    Viton (FKM) — ethanol and tar resistant
               Never standard rubber

Fittings:      Brass or stainless
               Avoid zinc plated — ethanol causes corrosion

Placement:     Between crankcase breather and intake
               Low as practical for condensate collection
               Away from heat sources
               Accessible for inspection and draining

Orientation:   Inlet high, outlet high (clean gas exits top)
               Drain at absolute bottom
               Sight glass or level markings preferred
```

### Baffle Options

| Type | Efficiency | Notes |
|---|---|---|
| Simple baffles | Moderate | Basic |
| Steel wool / mesh pack | Good | Cheap, replaceable |
| Labyrinth baffle | Very good | No maintenance element |
| Labyrinth + mesh | Excellent | Recommended |
| Coalescing filter element | Excellent | Best separation, replaceable |

### Injection Type Consideration

```
Port injection:    Catch tank alone is sufficient
                   Injector tips in cooler location
                   More tolerant of fuel variation
                   Recommended for syngas/ethanol builds

Direct injection:  More critical — injectors fire directly
                   into combustion chamber
                   Carbon buildup severe
                   Catch tank reduces rate but is not a
                   complete solution
                   Periodic walnut blasting required
```

Port injection is the preferred choice for a dedicated syngas/ethanol dual fuel build.

### PCV System Options

```
Standard PCV valve   → Sized for gasoline blow-by only
                       May not pass increased syngas volume

Uprated PCV valve    → Higher flow rating

Deleted PCV (stationary/off-road only)
                     → Open filtered breather to atmosphere
                     → Eliminates intake contamination entirely
                     → Requires regular catch tank emptying
```

### Catch Tank Monitoring

```
Float switch in catch tank:
  Filling faster than normal → increased blow-by
  → Indicates developing ring wear
  → Supervisor logs fill rate trend over time
  → Rising trend = maintenance warning

Correlate with:
  Crankcase pressure sensor (existing)
  Oil dilution sensor (existing)
  Together: early warning of ring wear, bore glazing,
            head gasket issues
```

---

<a id="sec-13"></a>
## 13. Final Filtration System

### Position in Gasification Chain

The final filter must be the last element before the mixer — after all cooling stages so tar is fully condensed and capturable:

```
[Gasifier]
     ↓
[Primary cyclone]          — coarse char/ash removal
     ↓
[Gas cooler]               — tar condenses to droplets
     ↓
[Secondary cyclone]        — condensed tar droplets removed
     ↓
[FINAL FILTER]             ← stainless / ceramic — last element
     ↓
[Mixer / venturi]
     ↓
[Engine]
```

Hot gas carries tar as vapor which passes through any filter. Cooling first is not optional.

### Stainless Steel Mesh Options

#### Woven Wire Mesh (316L Stainless)

| Mesh Grade | Particle Size | Use |
|---|---|---|
| 100 mesh | 150µm | Pre-filter stage |
| 200 mesh | 75µm | General use |
| 325 mesh | 44µm | Recommended final filter |
| 500 mesh | 25µm | Fine — needs large area |

316L preferred over 304 — superior resistance to tar acids and condensate.

#### Sintered Stainless Steel (316L) — Preferred

| Grade | Pore Size | Notes |
|---|---|---|
| Coarse | 40µm | Pre-filter |
| Medium | 20µm | Good general use |
| Fine | 10µm | Recommended final stage |
| Ultra-fine | 5µm | High restriction — large area needed |

```
Sintered SS advantages over woven mesh:
  Consistent pore size — not variable like woven
  3D depth filtration — not surface only
  Rigid — holds shape under pressure cycling
  Cleanable indefinitely — gasoline soak + ultrasound
  Temperature stable to 600°C+
  Non-stick surface treatment possible
```

10µm sintered 316L stainless is the practical optimum for final stage filtration.

### Ceramic Foam Filter — Recommended Final Stage

Originally developed for molten metal filtration — well suited to gasifier final filtration:

```
Structure:   Open cell reticulated foam
             Interconnected 3D pore network
Grades:      10, 20, 30, 40, 50 PPI (pores per inch)
Recommended: 20–30 PPI — balances filtration and flow
```

| Material | Max Temp | Tar Resistance | Notes |
|---|---|---|---|
| Silicon carbide (SiC) | 1400°C+ | Excellent | Recommended |
| Alumina (Al₂O₃) | 1600°C+ | Very good | Slightly more brittle |
| Zirconia (ZrO₂) | 2200°C+ | Excellent | Overkill |
| Cordierite | 1200°C | Good | Lower cost |

Silicon carbide 20–30 PPI is the recommended ceramic choice — non-reactive with tar compounds, thermally shock resistant, fully cleanable.

### Multi-Stage Final Filter Assembly

```
GAS FLOW DIRECTION →

┌──────────────────────────────────────────┐
│  FINAL FILTER HOUSING — 316SS            │
│                                          │
│  [Stage 1]      [Stage 2]    [Stage 3]   │
│  SS 200 mesh → Sintered SS → SiC foam   │
│  75µm           10–20µm      20–30 PPI  │
│  Coarse         Medium        Fine       │
│  Protects       Depth         Final      │
│  stages 2,3     filtration    polish     │
│                                          │
│  Drain at bottom — each stage            │
│  Differential pressure tap — each stage  │
└──────────────────────────────────────────┘
```

### Housing Specification

```
Material:      316 stainless steel — mandatory
               Tar acids attack mild steel rapidly

Configuration: Cylindrical, vertical preferred
               Gas enters top or side (dirty)
               Gas exits top clean side
               Drain at bottom per stage

Seals:         Viton (FKM) — tar and solvent resistant
               Never standard rubber

Access:        Tri-clamp fittings — quick element removal
               No tools needed, common in food/chemical industry
               Widely available in stainless

Ports:         Inlet (dirty), outlet (clean)
               Drain x1–3 (one per stage)
               Differential pressure taps
               Thermocouple pre-filter (confirms gas below dewpoint)
```

### Cleaning Procedures

#### Gasoline Soak

```
1. Remove element from housing
2. Submerge in clean gasoline or diesel
3. Soak 30–60 minutes — tar softens and dissolves
4. Agitate gently — soft brush if needed
5. Rinse with fresh gasoline
6. Hold to light — check flow uniformity
7. Air dry completely before reinstallation
8. Dispose of used gasoline correctly — tar contaminated
```

#### Ultrasonic Cleaning — Most Effective

```
1. Pre-soak in gasoline 15 minutes
   (ultrasound works better on softened deposits)
2. Transfer to ultrasonic bath
3. Cleaning fluid options:
     Isopropyl alcohol    — good tar solvent
     Kerosene             — effective, slower
     Ultrasonic detergent — commercial options
4. Run 20–40 minutes at 40–60°C bath temperature
   (heat significantly improves cleaning)
5. Rinse with clean solvent
6. Compressed air blow-through
7. Inspect and flow test before reinstallation

Frequency: 37–40 kHz
Reaches into sintered matrix depth
Cannot damage stainless or ceramic at normal power
```

#### Ceramic Foam — Special Handling

```
DO:
  Warm element before ultrasound
  Support fully during handling — fragile
  Inspect for cracks after each cleaning
  Replace immediately if cracked — bypass risk

NEVER:
  Thermal burn-off in open flame → thermal shock cracks ceramic
  High pressure air blast when cold → fracture risk
  Mechanical scrubbing → surface cell damage
```

### Filter Monitoring

```
Differential pressure across final filter assembly:

  Clean filter:          0.2 – 0.5 kPa
  Normal operation:      0.5 – 1.5 kPa
  Clean recommended:     2.0 kPa
  Clean immediately:     3.0 kPa
  Engine protect limit:  4.0 kPa → auto-revert to gasoline

Supervisor logs ΔP trend continuously:
  Rate of rise indicates gas quality / tar content
  Faster clogging = wetter wood or poor gasification
  Slow clogging with Alnus incana = system working correctly
```

### Cleaning Intervals

```
Alnus incana dry feedstock:   Every 40–80 operating hours
                               (differential pressure is the true indicator)

Wet or resinous wood:         Interval drops dramatically
                               → Further reason Alnus incana is preferred

Element service life:
  Sintered stainless:  Indefinite — hundreds of cleaning cycles
  Ceramic foam SiC:    Many cycles if handled carefully
                       Replace if cracked
  Woven SS mesh:       Indefinite unless mechanically damaged
```

---

<a id="sec-14"></a>
## 14. Alternative Feedstocks — Locally Available Materials

All feedstocks below are readily available in Norwegian and Scandinavian conditions. Used alone or blended with Alnus incana as the quality anchor.

---

### Wood Industry Waste

#### Sawdust and Wood Shavings
```
Source:         Sawmills, carpentry shops — often free in quantity
Compression:    Excellent — best compression ratio of all materials
                Log splitter produces very dense stable briquette
Moisture:       Fresh sawdust 30–50% — press then air dry
                Target below 15% before use
Ash content:    0.3–1% (clean hardwood species)
Tar yield:      Low–moderate — hardwood preferred over softwood

AVOID:
  MDF, chipboard, plywood sawdust
  → Formaldehyde and urea-formaldehyde resins
  → Toxic gasification byproducts
  Pine/spruce sawdust — higher resin content, more tar
  Hardwood sawdust strongly preferred
```

#### Wood Pellets (EN Plus Standard)
```
Source:         Hardware stores, fuel suppliers — widely available
Compression:    Already compressed — use directly
Ash content:    0.3–0.7% (EN Plus A1 certified)
Moisture:       Below 10% — controlled and consistent
Tar yield:      Low
Size:           6–8mm diameter — excellent hopper flow

Notable:        Most consistent feedstock available commercially
                Certified quality — predictable and stable gas output
                Eliminates feedstock variability entirely
                Excellent reference fuel for initial ECU tuning
                and establishing baseline VE tables
                Cost higher than waste materials but
                quality advantage significant for tuning work
```

#### Wood Pellet Industry Debris and Fines
```
Source:         Pellet mills — fines and broken pellets
                screened out as waste product before packaging
                Often free or very low cost directly from mill
Compression:    Press fines into briquettes with log splitter
                Fines alone do not feed well — must be compressed
Ash content:    Same as parent pellets — 0.3–0.7%
Moisture:       Low — same drying process as pellets
Tar yield:      Low

Notable:        Essentially the same quality as commercial pellets
                at waste stream price
                Ideal — quality of EN Plus at zero or minimal cost
                May need simple screening to remove bark fines
                Contact local pellet mills directly
                Large mills produce significant debris volumes
```

#### Bark
```
Source:         Sawmill debarking waste
Compression:    Good
Ash content:    2–5% — higher than clean wood
Tar yield:      Moderate–high — more extractives than wood
Notable:        Better blended than used alone (max 20%)
                Alder bark lower tar than conifer bark
                Conifer bark particularly resinous — use sparingly
```

---

### Paper and Cardboard

#### Newspaper
```
Source:         Households, printers, distribution centres
Compression:    Excellent — very high density achievable
Ash content:    2–4%
Tar yield:      Low
Moisture:       Usually already dry

Notable:        Modern newspaper ink is soy or water based
                Minimal chlorine concern with current printing
                Older stock may have higher ink loading
                One of the easiest and most available feedstocks
                Compress tightly — expands if not well pressed
```

#### Cardboard and Corrugated Packaging
```
Source:         Supermarkets, retail, warehouses — very abundant
Compression:    Excellent — corrugated compresses to high density
Ash content:    2–5% (clay fillers, coatings variable)
Tar yield:      Low
Notable:        Wax coated boxes acceptable — wax burns cleanly
                Check for foil laminate or plastic windows — remove
                Brown kraft cardboard is the best grade
                Egg cartons — uncoated, excellent feedstock
```

---

### Agricultural Waste

#### Barley and Oat Straw
```
Source:         Farms after harvest — very abundant, often free
Compression:    Excellent — compresses to stable dense briquette
                Log splitter handles easily
Moisture:       Field baled 15–25%, barn stored better
                Press squeezes out significant free moisture
Ash content:    4–6%
Tar yield:      Low–moderate
Silica content: Moderate — lower than wheat straw

Notable:        Barley and oat straw preferred over wheat straw
                  → Lower silica content
                  → Less abrasive ash on grate and agitator
                  → Less glassy clinker formation
                Blend with wood material recommended — max 30–40%
                Straw alone produces high ash volume
                Monitor agitation frequency — increases with straw content
```

#### Wheat Straw
```
Source:         Farms — extremely abundant
Compression:    Excellent
Ash content:    5–8%
Tar yield:      Low–moderate
Silica:         High — highest of the common straws

Notable:        Higher silica than barley/oat straw
                  → More abrasive ash
                  → Glassy clinker at oxidation zone temperatures
                  → Increases grate and agitator wear
                Use as blend component only — max 20%
                Barley or oat straw preferred if choice available
```

#### Hay
```
Source:         Farms — surplus or spoiled bales
Compression:    Excellent
Ash content:    5–9% — variable with grass species
Tar yield:      Low–moderate
Moisture:       Variable — barn stored dry hay preferred
                Spoiled wet hay needs pressing and long drying

Notable:        More variable composition than straw
                Mixed grass species = variable gas quality
                Acceptable blend component — max 20–25%
                Good use for spoiled hay unsuitable for animal feed
```

---

### Forest Industry Waste

#### Conifer Cones (Pine, Spruce)
```
Source:         Forest floor, forestry operations, sawmill yards
                Extremely abundant in Norwegian conditions
Compression:    Moderate — hollow structure, less compressible
                than paper but log splitter handles well
Moisture:       Fresh 30–50% — must be dried
                6–12 weeks open air drying minimum after pressing
                Pressing expels significant free moisture
Ash content:    1–3%
Tar yield:      Moderate–high — resin pockets in scales

Notable:        Resin content is the main concern
                → Higher filter loading than wood
                → Monitor differential pressure — rises faster
                → Increase filter cleaning frequency accordingly
                Best used as blend with alder — max 20–30%
                Dry thoroughly — wet cones produce very poor gas
                and severely increase tar loading
```

#### Wood Chips
```
Source:         Tree surgeons, municipal green waste contractors
                Forest harvesting operations — often free
Compression:    Good when pre-dried and chipped uniformly
Moisture:       Fresh chips 40–55% — significant drying needed
                6–12 months open air drying recommended
                Pressing helps but moisture is deep in chip

Notable:        Size inconsistency can cause hopper bridging
                Chip through log splitter form to standardise size
                Mixed species chips — quality varies
                Request hardwood chips where possible
```

---

### Charcoal

Fundamentally different from other feedstocks — pyrolysis already complete, near-zero tar output.

```
Source:         Commercial charcoal, self-produced from gasifier char,
                charcoal production operations

Properties:
  Ash content:    1–5% (depends on source wood)
  Tar yield:      Near zero — pyrolysis already done
  Moisture:       3–8% — store dry, hygroscopic
  Carbon content: 75–95%
  Warmup time:    5–8 minutes (shorter than wood)

Gas composition vs wood gas:
  CO higher:   28–35% vs 18–25%
  H₂ lower:    5–10%  vs 10–16%
  Tar:         Near zero vs significant

Advantages:
  Near-zero tar → filter stays clean far longer
  Very clean engine operation
  Reduced oil contamination
  Consistent gas quality
  Shorter warmup time

Disadvantages:
  Lower bulk density — larger hopper volume needed
  Fragile — crushes to fines, store and handle carefully
  Fines block gasifier bed — agitation critical
  Hygroscopic — sealed dry storage essential
  Cost higher than raw wood unless self-produced

Self-produced charcoal:
  Gasifier produces char as byproduct
  Collect from ash system
  Process in simple retort kiln
  Feed back as fuel — zero waste loop
  Alnus incana char → low ash charcoal → very clean gas
```

---

### Blending Strategy for Norwegian Conditions

```
PRIMARY FUEL (quality anchor):
  Alnus incana wood cubes          40–50%

WOOD INDUSTRY (free or low cost):
  Sawdust/shavings briquettes      15–20%
  Wood pellet debris/fines         10–15%
    (contact local pellet mill)

AGRICULTURAL (seasonal availability):
  Barley or oat straw briquettes   10–15%
  Hay briquettes (dry surplus)     0–10%

FOREST WASTE:
  Conifer cone briquettes          10–15%
    (well dried — below 15% moisture)

PAPER/CARDBOARD:
  Newspaper + cardboard blend      10–15%

CHARCOAL (periodic):
  Pure charcoal run every 50–100 hours
  Cleans gasifier internals and engine simultaneously

WOOD PELLETS:
  Keep a supply for:
    Initial ECU tuning reference
    Quality baseline verification
    Backup when other stocks low
```

---

### Feedstock Quick Reference

| Material | Source | Cost | Compress | Ash | Tar | Max Blend % |
|---|---|---|---|---|---|---|
| Alnus incana | Woodlot / supplier | Low | Optional | Very low | Very low | Primary — 100% capable |
| Wood pellets | Fuel supplier | Low | No | Very low | Low | 100% — reference fuel |
| Pellet mill debris | Pellet mill | Free | Yes — fines | Very low | Low | 100% if clean |
| Sawdust (hardwood) | Sawmill | Free | Yes | Low | Low | 100% if clean species |
| Cardboard / newspaper | Retail / households | Free | Yes | Low–med | Low | 20–30% |
| Barley / oat straw | Farms | Free | Yes | Medium | Low–mod | 30–40% |
| Wheat straw | Farms | Free | Yes | Med–high | Low–mod | 20% |
| Hay | Farms | Free | Yes | Med–high | Low–mod | 20–25% |
| Conifer cones | Forest | Free | Yes | Low–med | Mod–high | 20–30% |
| Wood chips (hardwood) | Tree surgeons | Free | Optional | Low | Low–mod | 50% if dried |
| Charcoal | Various / self-made | Low–med | No | Low | Near zero | 100% capable |

---

### Golden Rules for Any Feedstock

```
1. Moisture below 15% before use — press then dry
2. No chlorine — no PVC, foil laminates, glossy coatings
3. No synthetic polymers — no polyester, nylon, acrylic
4. No treated materials — no painted, preservative treated wood
5. No MDF, chipboard, plywood — resin binders are toxic
6. Test unfamiliar materials with small open fire first
     Clean flame = proceed
     Black smoke / acrid smell = reject
7. Monitor filter ΔP rate of rise — fastest indicator
   of feedstock quality impact on the system
```

---


---

<a id="sec-14b"></a>
## 14b. Regional Feedstock Alternatives — International

The feedstocks in Section 14 reflect Norwegian and Scandinavian conditions. The gasification principles, EFI system, filtration requirements, and golden rules apply universally. What changes by region is which locally abundant waste materials fill the hopper.

The same evaluation criteria apply everywhere:

- Moisture below 15% before use — press then dry
- No chlorine — no PVC, foil laminates, glossy coatings
- No synthetic polymers
- No treated, painted, or preservative-treated materials
- No MDF, chipboard, or resin-bonded composite wood
- Test unfamiliar materials with a small open flame — clean burn proceeds, black smoke or acrid smell rejects

---

### North America

| Material | Source | Cost | Compress | Ash | Tar | Max Blend % | Notes |
|---|---|---|---|---|---|---|---|
| Corn cobs | Farms — harvest waste | Free | Yes | 1–2% | Low | 100% capable | Excellent feedstock — low ash, consistent size, well documented in NREL literature |
| Corn stover (stalks and leaves) | Farms | Free | Yes | 4–6% | Low–mod | 30–40% | Very abundant; higher ash than cobs; blend with wood |
| Wheat straw | Great Plains farms | Free | Yes | 5–8% | Low–mod | 20% | Same silica caution as Norwegian wheat straw |
| Sorghum stover | Southern/central farms | Free | Yes | 4–7% | Low–mod | 30% | Similar to corn stover |
| Pecan shells | Southern US — processing plants | Free–low | Yes | 1–3% | Low | 50% | Hard dense shell — excellent energy density, low tar |
| Walnut shells | California, Pacific NW | Free–low | Yes | 1–2% | Low | 50% | Very hard — high density briquette |
| Cotton gin trash | Cotton belt | Free | Yes | 8–12% | Mod | 20% | High ash — blend only; widely available in quantity |
| Hardwood chips (oak, hickory, maple) | Tree services, sawmills | Free–low | Optional | Low | Low | 100% if dried | Excellent — well dried hardwood is benchmark quality |
| Cardboard / newspaper | Universal | Free | Yes | Low–med | Low | 30% | Same as Section 14 — universally available |

---

### Southeast Asia

| Material | Source | Cost | Compress | Ash | Tar | Max Blend % | Notes |
|---|---|---|---|---|---|---|---|
| Rice husks | Rice mills — abundant waste | Free | Yes | 15–20% | Very low | 30% | Very high silica ash — blend only; near-zero tar; among the most studied gasification feedstocks globally |
| Coconut shells | Coconut processing | Free–low | No — use whole or cracked | 0.5–1% | Very low | 100% capable | Outstanding feedstock — dense, low ash, very low tar, high fixed carbon |
| Coconut husks / coir | Coconut processing | Free | Yes | 3–6% | Low–mod | 40% | Bulky — compress well; slightly more tar than shell |
| Sugarcane bagasse | Sugar mills — large volumes | Free | Yes | 3–5% | Low | 40% | Very abundant; high moisture when fresh — press and dry thoroughly |
| Bamboo chips | Plantations, construction waste | Free–low | Optional | 1–3% | Low–mod | 60% | Fast growing, good energy density; some species higher silica |
| Palm kernel shells | Palm oil mills | Free–low | No | 1–3% | Low | 100% capable | Excellent — dense, low ash, low tar; widely available in Indonesia, Malaysia, Thailand |
| Cashew nut shells | Processing plants | Low | No | 1–2% | Mod–high | 20% | Contains CNSL (cashew nut shell liquid) — resinous, higher tar; blend only |
| Jute sticks | Bangladesh, India — processing waste | Free | Yes | 3–5% | Low | 50% | Good woody feedstock; well dried essential |

---

### Sub-Saharan Africa

| Material | Source | Cost | Compress | Ash | Tar | Max Blend % | Notes |
|---|---|---|---|---|---|---|---|
| Maize cobs | Farms — harvest waste | Free | Yes | 1–2% | Low | 100% capable | Equivalent to North American corn cobs — excellent feedstock |
| Groundnut (peanut) shells | Processing operations | Free–low | Yes | 3–5% | Low | 60% | Well studied in African gasification projects — good performance |
| Sorghum stalks | Farms | Free | Yes | 5–8% | Low–mod | 30% | High ash — blend with lower ash materials |
| Cotton stalks | Farms | Free | Yes | 6–10% | Low–mod | 25% | Very abundant; high ash, blend only |
| Coffee husks | Coffee processing regions | Free | Yes | 4–7% | Low–mod | 40% | Ethiopia, Kenya, Uganda — large volumes available at processing stations |
| Acacia / Prosopis chips | Invasive species management | Free | Optional | Low | Low–mod | 60% | Prosopis (mesquite) is an invasive pest in many regions — gasification is a productive use |
| Eucalyptus chips | Plantations — widely grown | Free–low | Optional | Low | Low | 80% | Fast growing plantation species; good energy density when dried |
| Dried cattle dung (biomass) | Farms | Free | Yes | 15–25% | Low | 15% | Traditional fuel — very high ash; blend component only; widely available |

---

### South America

| Material | Source | Cost | Compress | Ash | Tar | Max Blend % | Notes |
|---|---|---|---|---|---|---|---|
| Sugarcane bagasse | Sugar mills — enormous volumes | Free | Yes | 3–5% | Low | 40% | Brazil and Argentina produce vast quantities; press and dry thoroughly before use |
| Coffee husks | Processing operations | Free | Yes | 4–7% | Low–mod | 40% | Colombia, Brazil, Peru — large volumes at processing plants |
| Eucalyptus chips | Brazil — world's largest plantations | Free–low | Optional | Low | Low | 100% if dried | Fast-growing, consistent quality, widely planted; excellent feedstock |
| Rice husks | Argentina, Brazil, Uruguay | Free | Yes | 15–20% | Very low | 30% | Same high-silica caution as Southeast Asian rice husks — blend only |
| Macadamia / Brazil nut shells | Processing plants | Free–low | No | 0.5–1% | Very low | 100% capable | Very dense hard shells — excellent energy density, near-zero tar |
| Soybean husks | Processing plants | Free | Yes | 4–6% | Low | 30% | Very abundant in Brazil and Argentina; moderate ash, blend with wood |
| Yerba mate stems | Processing operations | Free | Yes | 3–5% | Low–mod | 30% | Argentina, Paraguay — processing waste available in quantity |

---

### Mediterranean and Middle East

| Material | Source | Cost | Compress | Ash | Tar | Max Blend % | Notes |
|---|---|---|---|---|---|---|---|
| Olive stones (pits) | Olive oil mills — October–December | Free–low | No — use whole | 2–4% | Very low | 100% capable | Outstanding feedstock — very dense, low moisture, low tar, high fixed carbon. Seasonal but storable |
| Olive pruning waste | Orchards — winter pruning | Free | Yes | 1–3% | Low | 80% | Chipped prunings from regular orchard management; excellent quality |
| Almond shells | Processing plants | Free–low | No | 2–3% | Low | 100% capable | Spain, Italy, Turkey, California — dense shell, low tar, excellent energy density |
| Grape marc / pomace | Wineries — post-harvest | Free | Yes | 5–8% | Mod | 25% | Seeds and skins after pressing; dry thoroughly; seeds improve fixed carbon |
| Citrus pulp / peel (dried) | Juice processing | Free | Yes | 3–5% | Low–mod | 25% | Must be thoroughly dried — fresh citrus waste is 80%+ moisture |
| Date palm fronds | Middle East / North Africa | Free | Yes | 4–7% | Low–mod | 30% | Very abundant; chipped or shredded fronds; dry well |
| Date palm pits | Date processing | Free | Yes | 1–3% | Very low | 80% | Dense hard pit — similar to olive stones; excellent quality |
| Pistachio shells | Iran, Turkey, Central Asia | Free–low | No | 1–2% | Low | 100% capable | Dense, low ash, very low tar — comparable to almond shells |

---

### Universal Urban Feedstocks

These are available in essentially every country and urban area regardless of climate or agriculture:

| Material | Notes |
|---|---|
| Corrugated cardboard | Universal — supermarkets, retail, warehouses produce large clean volumes daily |
| Newspaper and office paper | Universal — already sorted in waste streams in most countries |
| Untreated wood pallets | Extremely abundant globally — verify no heat treatment chemical stamps (HT stamped is fine; MB stamped means methyl bromide treatment — reject) |
| Untreated timber offcuts | Construction sites, carpenter workshops — free and abundant |
| Paper sacks and kraft bags | Cement, flour, feed sacks — clean kraft paper, widely available |
| Wood wool / wood shavings (packaging) | Furniture and fragile goods packaging — clean, dry, ready to use |

**Pallet note:** Look for the IPPC stamp on the pallet. **HT** (heat treated) is safe. **MB** (methyl bromide fumigated) must be rejected — toxic gasification byproducts.

---

### Regional Quick Reference — Best Primary Feedstocks

| Region | Best anchor feedstock | Best free bulk material |
|---|---|---|
| Scandinavia / Northern Europe | Alnus incana (grey alder) | Cardboard, sawmill waste |
| North America | Hardwood chips (oak, maple, hickory) | Corn cobs, cardboard |
| Southeast Asia | Coconut shells / palm kernel shells | Rice husks (blend only — high ash) |
| Sub-Saharan Africa | Maize cobs | Groundnut shells, eucalyptus chips |
| South America | Eucalyptus chips | Sugarcane bagasse, coffee husks |
| Mediterranean / Middle East | Olive stones / almond shells | Olive pruning waste, date palm pits |
| Universal urban | Untreated hardwood pallets (HT stamped) | Corrugated cardboard, newspaper |

---

### Golden Rules — Universal

Regardless of region or feedstock, these rules apply without exception:

```
1.  Moisture below 15% before use — press then dry
2.  No chlorine — no PVC, foil laminates, glossy coated paper
3.  No synthetic polymers — no polyester, nylon, acrylic
4.  No treated materials — no painted, preservative-treated wood
5.  No MDF, chipboard, plywood — resin binders produce toxic gas
6.  No MB-stamped pallets — methyl bromide residue
7.  Test unfamiliar materials with small open flame first
      Clean flame = proceed
      Black smoke / acrid smell = reject
8.  High ash feedstocks (rice husks, dung, cotton) — blend only
9.  Monitor filter ΔP rate of rise — fastest indicator of
    feedstock quality impact on the system
10. When in doubt, blend with a known good anchor feedstock
    rather than running unknown material alone
```


---

<a id="sec-14c"></a>
## 14c. Invasive and Weed Species — Free Fuel From Problem Plants

Many of the best gasification feedstocks are plants that landowners, municipalities, and conservation bodies actively want removed. Using them as fuel turns a disposal problem into a fuel supply. In many cases the material is free for collection, and the landowner may welcome the help.

The same filtration and monitoring discipline applies — some invasive species have higher resin or extractive content than domesticated feedstocks, so filter differential pressure monitoring is the key indicator.

---

### The Weed-as-Fuel Principle

```
Problem plant = free biomass + grateful landowner + no fuel cost

The removal effort is already being paid for by someone.
Gasification turns that cost into a benefit.

Practical approach:
  Contact local councils, conservation trusts, farmers,
  road maintenance crews, and forestry operations.
  Offer to collect cleared material before it is burned
  or chipped to waste.
  Most will say yes immediately.
```

---

### Confirmed Gasification Feedstocks — Invasive Woody Species

| Species | Invasive Range | Ash | Tar | Notes |
|---|---|---|---|---|
| **Black locust** (*Robinia pseudoacacia*) | Eastern US / Appalachia, Central Europe | **0.37–0.76%** | Very low | Outstanding feedstock — documented downdraft gasification, 4,505 cal/g calorific value, higher density than poplar or willow (602 kg/m³). Fixes nitrogen so grows aggressively on disturbed land. **Primary anchor species for eastern US builds.** |
| **Mesquite** (*Prosopis glandulosa*) | Texas, Oklahoma, SW US, Africa | 1–2% | Low–mod | Studied at Texas A&M — HHV 8,653 BTU/lb, very low moisture naturally (often 6%), energy equivalent to medium-grade subbituminous coal. 22 million acres affected in Texas alone. Use downdraft gasifier — updraft produces high tar. |
| **Redberry juniper** (*Juniperus pinchotii*) | Texas, Oklahoma, Great Plains | 1–2% | Low–mod | HHV 8,849 BTU/lb — slightly higher than mesquite. Same downdraft caution applies. 20 dry tons/acre standing biomass possible. Resinous — monitor filter ΔP. |
| **Eastern redcedar** (*Juniperus virginiana*) | Eastern US — expanding rapidly | 1–3% | Mod | Very resinous — higher tar than most hardwoods. Use as blend component max 25%. Dry thoroughly. Landowners actively seek removal across the Great Plains. |
| **Prosopis** spp. (various) | Sub-Saharan Africa, South Asia, South America | 1–2% | Low | Same genus as North American mesquite — excellent fuel properties. Invasive pest across vast areas of Africa, India, and South America. Landowners and governments pay for removal. |
| **Tree of heaven** (*Ailanthus altissima*) | Eastern US, Europe, widespread | 1–2% | Low–mod | Fast-growing, aggressive invasive — grows on roadsides, waste ground, even building cracks. Reasonable wood density, moderate energy content. Blend at 40–50% with better quality material. |
| **Autumn olive / Russian olive** (*Elaeagnus* spp.) | Eastern and Central US | 1–3% | Low | Nitrogen-fixing shrub — dense growth on disturbed land. Reasonable fuel properties. Cut and chip — regrows rapidly so sustainable harvest possible. |

---

### Invasive Herbaceous and Semi-Woody Species

These species have hollow or pithy stems and require briquetting before use. They are not suitable fed loose into a hopper.

| Species | Range | Ash | Tar | Notes |
|---|---|---|---|---|
| **Japanese knotweed** (*Fallopia japonica*) | Europe, North America, widespread | 3–5% | Low–mod | Hollow bamboo-like stems — **must be chipped or shredded and briquetted**. Calorific value 16–17 MJ/kg — viable. Stalks contain polyphenols (resveratrol) — no gasification hazard, but adds slightly to extractive content. Grows 3 metres in one season; stands produce 2–3 kg/m² dry biomass annually. In the UK, knotweed arisings are classified as controlled waste — **verify local regulations before moving material**. Blend at 30–40% with woody material. |
| **Giant knotweed** (*Fallopia sachalinensis*) | Europe, North America | 3–5% | Low–mod | Same as Japanese knotweed — larger stems, similar properties. Often hybridises (*Bohemian knotweed* — *Fallopia × bohemica*). Same handling approach. |
| **Giant hogweed** (*Heracleum mantegazzianum*) | Europe, North America | 4–6% | Low | Hollow stems — briquette only. Calorific value 16–17 MJ/kg. **WARNING: fresh plant sap causes severe phototoxic burns — wear full protective clothing, gloves, and eye protection when handling. Dry material is safe.** Dried and briquetted stems are acceptable blend material at 20–25%. |
| **Sosnowsky's hogweed** (*Heracleum sosnowskyi*) | Eastern Europe, Scandinavia | 4–6% | Low | Same family as giant hogweed — same phototoxic sap warning applies. Dried briquettes acceptable. Planted as a livestock crop in the Soviet era; now a major invasive problem across Scandinavia and Eastern Europe. |
| **Kudzu** (*Pueraria montana*) | South-eastern US — spreading north | 4–7% | Low–mod | Covers millions of acres in the US south. Woody vine — growing season biomass is very high moisture; harvest in autumn when lignified. Dry thoroughly. Blend at 25–30%. Can smother and kill mature trees — landowners very keen for removal. |
| **Common reed** (*Phragmites australis*) | Europe, North America, widespread | 4–6% | Low | Annual harvest possible — wetland management operations produce large volumes. Dry well. Blend at 30–40%. Calorific value 16–17 MJ/kg. Already used in pellet production in Eastern Europe. |

---

### Scandinavian and Northern European Invasive Species

Relevant specifically for Norwegian and Scandinavian builds:

| Species | Notes |
|---|---|
| **Sosnowsky's hogweed** | As above — a serious problem across Norway and Scandinavia. Municipalities pay for removal. Dried briquettes viable at 20–25% blend. Wear full PPE when handling fresh material. |
| **Giant hogweed** (*Heracleum mantegazzianum*) | As above — widespread in Norway. Same PPE requirement fresh. |
| **Lupine** (*Lupinus polyphyllus*) | Dense stands on roadsides and disturbed ground across Norway. Annual herbaceous material — high moisture when fresh. Dry and briquette. Blend at 20% maximum. |
| **Japanese knotweed** | Present and spreading in Norway — same handling rules as above. Check local regulations. |
| **Himalayan balsam** (*Impatiens glandulifera*) | Spreading in southern Norway — annual, very high moisture. Dry before use. Low lignin — blend with woodier material at 15–20%. |
| **Sitka spruce** (*Picea sitchensis*) self-seeded escapes | Planted widely in Norway; spreading from plantations into native heathland. Young trees and brush cleared by conservation operations — resinous, higher tar than hardwoods. Use at 20–30% blend maximum, monitor filter ΔP closely. |

---

### Important Notes for Invasive Species Handling

```
REGULATIONS:
  Some invasive species are subject to legal restrictions
  on movement and disposal. Japanese knotweed in the UK
  is the most notable example — arisings are controlled
  waste and cannot be moved without compliance.
  Verify local regulations before collecting or transporting
  any invasive plant material.

PPE FOR HOGWEED SPECIES:
  Giant hogweed and Sosnowsky's hogweed sap causes
  severe phototoxic burns — blistering in sunlight.
  Wear: waterproof gloves, long sleeves, eye protection.
  Dry material after thorough drying is safe to handle normally.
  Never burn fresh hogweed — smoke can cause eye and lung injury.

RHIZOME MANAGEMENT (knotweed):
  Knotweed rhizomes can regenerate from small fragments.
  Gasification destroys the plant completely — one of the
  few truly permanent disposal methods.
  This is a genuine environmental benefit of the application.

RESINOUS SPECIES (juniper, cedar, sitka):
  Monitor filter ΔP more frequently than with clean hardwood.
  Blend at stated maximum percentages.
  Charcoal-only runs every 40–50 hours clean the system.
```

---

### Why Weed Species Are Often Excellent Feedstocks

Invasive plants share several characteristics that make them useful gasification feedstocks:

```
Fast growth      → High annual biomass yield per area
Disturbed land   → Grows where nothing useful would grow anyway
No cultivation   → Zero agricultural input cost
Landowner wants  → Collection is welcomed, not charged for
  removal
Dense stands     → High collection efficiency per hour
Dry naturally    → Many species growing on free-draining disturbed
                   ground have lower initial moisture than
                   plantation crops
```

The parallel with grey alder in Norway is exact — a species that is aggressive, unwanted, and free is precisely the right anchor feedstock for a zero-cost fuel system.

---

### Invasive Species — Regional Quick Reference

| Region | Best invasive feedstock | Caution |
|---|---|---|
| Eastern US / Appalachia | Black locust | None — excellent feedstock |
| Texas / Great Plains | Mesquite, redberry juniper | Use downdraft gasifier; monitor tar |
| South-eastern US | Kudzu | High moisture — dry thoroughly |
| Europe / North America widespread | Japanese / giant knotweed | Check local disposal regulations |
| Norway / Scandinavia | Sosnowsky's hogweed, lupine | PPE required fresh; dry before use |
| Global tropical | Prosopis spp. (mesquite relatives) | Excellent properties; widely available |
| Eastern US roadsides | Tree of heaven (*Ailanthus*) | Blend at 40–50% |

<a id="sec-15"></a>
## 15. Fuel Briquette Production — Converted Hydraulic Log Splitter

A standard hydraulic log splitter converted with a compression end plate is a practical and effective briquette press requiring minimal investment.

### Why a Log Splitter Works Well

```
Typical log splitter force:    10–30 tonnes
Compression ratio achieved:    6:1 to 10:1 depending on material
Density achieved:              500–900 kg/m³
Moisture expulsion:            Significant — free moisture
                               literally squeezed out under pressure
Availability:                  Widely available new or secondhand
Existing hydraulics:           No modification to hydraulic system
                               Only the working end is changed
Production rate:               4–8 briquettes per minute realistic
```

### Conversion — Compression End Plate

Replace the splitting wedge with a flat compression plate and add a forming box:

```
REMOVE:
  Splitting wedge — unbolt or cut off

FABRICATE AND FIT:

  1. COMPRESSION PLATE
     Material:    20–25mm steel plate
     Size:        Match forming box opening exactly
     Edges:       Chamfered slightly — aids ejection
     Attach:      Weld or bolt to ram end in place of wedge
     Drainage:    3–5mm holes drilled through plate
                  Allows moisture to escape during compression

  2. FORMING BOX (fixed to splitter table)
     Material:    5–8mm steel plate, welded construction
     Internal:    100×100mm or 150×150mm square cross section
                  200–250mm depth
     Open top:    Loading — fill from above
     Open bottom: Ejection — briquette pushed through and out
     Drainage:    Slots or holes in sides and base
                  Moisture exits during pressing
     Attachment:  Bolt to existing splitter table
                  Removable for maintenance

  3. OPTIONAL STOP PLATE
     Thin steel plate placed under forming box
     Catches briquette on ejection
     Slide out to collect finished briquette
```

### Forming Box Dimensions

```
100×100mm internal:
  Produces small dense briquettes
  Good for small gasifiers
  Higher production rate per cycle

150×150mm internal:
  Larger briquette — better for larger gasifiers
  Fewer cycles needed for same fuel volume
  Better for coarse materials like straw and cones

200mm depth recommended for both:
  Sufficient compression travel
  Good briquette length for hopper feed
```

### Compression Plate Detail

```
        RAM →  [=======PLATE=======]
                  ○  ○  ○  ○  ○      ← drainage holes
               ___________________
              |                   |   ← forming box walls
              |   MATERIAL        |
              |   being           |
              |   compressed      |
              |___________________|
                        ↓
                  [briquette exits]
                  moisture drains
```

### Material Preparation

```
NEWSPAPER / CARDBOARD:
  Tear or shred to roughly 100mm pieces
  Soak in water 30–60 minutes
    (wet paper compresses better and binds on drying)
  Squeeze out bulk water by hand first
  Load forming box — compress fully
  Hold pressure 10–20 seconds
  Eject — stack on drying rack

SAWDUST / WOOD SHAVINGS:
  No soaking needed if moisture above 20%
  If dry — add small amount of water to aid binding
  Fill box loosely — material compacts significantly
  Compress fully — hold 15–30 seconds
  Eject carefully — dry sawdust briquettes fragile until dried

STRAW / HAY:
  Cut or shred to 50–100mm lengths
  No soaking needed — press dry or slightly damp
  Load loosely — compresses dramatically
  Compress fully — hold 20–30 seconds
  Eject — straw briquettes hold shape well

CONIFER CONES:
  Partially dry first — 4–6 weeks minimum
  Mix with 20–30% shredded newspaper for binding
  Pure cones compress less uniformly
  Mixed cone/paper briquette much more stable

MILK CARTONS / CARDBOARD PACKAGING:
  Flatten and tear to pieces
  Mix with newspaper for better binding
  Compress — PE content in cartons aids cohesion on drying
```

### Binding — Natural Options

```
No chemical binders needed for most materials.
Natural binding occurs through:

  Paper/cardboard:   Cellulose fibres interlock under pressure
                     Hydrogen bonding on drying — very strong

  Sawdust:           Lignin softens slightly under compression heat
                     Binds on cooling and drying

  Straw:             Silica surface aids mechanical interlocking

  PE in cartons:     Slight plasticity aids cohesion

For difficult materials (pure cones, coarse bark):
  Add 20–30% newspaper or cardboard as binder matrix
  Starch paste (potato or wheat starch) — 5–10% addition
    → Traditional briquette binder
    → Fully organic — no gasification concerns
    → Mix with water to thin paste, coat material before pressing
```

### Drying After Pressing

```
Stack briquettes with gaps between for airflow:

  Newspaper/cardboard:   1–3 weeks air drying
  Sawdust:               2–4 weeks
  Straw/hay:             1–2 weeks (dries faster than wood)
  Conifer cone mix:      3–6 weeks
  Mixed materials:       2–4 weeks

Target moisture below 15% before gasifier use.

Accelerate drying:
  Stack near gasifier exhaust heat (not contact)
  Solar drying rack — covered but ventilated
  Barn storage with airflow — ideal

Check with wood moisture meter before loading hopper.
Rising filter ΔP in operation = feedstock too wet.
```

### Production Capacity Estimate

```
One operator, standard 8-tonne log splitter:

  Newspaper briquettes:     60–80 per hour
  Cardboard briquettes:     50–70 per hour
  Straw briquettes:         40–60 per hour
  Sawdust briquettes:       50–70 per hour
  Mixed briquettes:         50–70 per hour

Energy equivalent per hour production:
  ~20–40 kWh thermal fuel value
  Sufficient for several hours of engine operation
  depending on load and engine size

Batch production — press one session per week
adequate for continuous operation of most installations.
```

### Safety During Pressing

```
Hydraulic log splitters are powerful:
  Never place hands in forming box during operation
  Keep forming box securely bolted to table
  Inspect welds on forming box regularly
  Ensure drainage holes clear — blocked holes
    → hydraulic pressure buildup in water
    → sudden ejection risk

Wet material:
  Moisture spray during compression is normal
  Stand clear of ejection direction
  Wear eye protection
```

---

<a id="sec-16"></a>
## 16. EGT Operating Ranges

| EGT Range | Condition | Action |
|---|---|---|
| Below 650°C | Rich or misfiring | Lean mixture / check gasifier |
| 650–800°C | Ideal zone | Normal operation |
| 800–950°C | Approaching lean limit | Enrich mixture |
| Above 950°C | Dangerously lean | Auto-revert to gasoline |

---

<a id="sec-17"></a>
## 17. Notes and Operational Summary

- Wood gas is a viable engine fuel with effective RON 100–120, making it genuinely turbo-compatible without knock risk at moderate boost pressures
- Alnus incana (grey alder) is the benchmark feedstock — low ash, low tar, consistent gas quality; use as the quality anchor in any blend
- Locally abundant feedstocks in Norwegian conditions: sawdust and wood shavings, wood pellets and pellet mill debris, newspaper, cardboard, barley/oat straw, hay, conifer cones, wood chips, and charcoal — all viable alone or blended
- Barley and oat straw preferred over wheat straw — lower silica content, less abrasive ash and clinker formation
- Wood pellets (EN Plus) are the ideal reference fuel for initial ECU tuning — consistent and certified quality eliminates feedstock variables during setup
- Pellet mill debris and fines are essentially free EN Plus quality fuel — contact local mills directly
- Charcoal produces near-zero tar, shorter warmup (5–8 min), and very clean engine operation — periodic charcoal-only runs clean gasifier and engine internals
- A hydraulic log splitter converted with a flat compression plate and forming box is an effective briquette press — no modification to hydraulics, only the working end changes
- Compression simultaneously increases density and expels free moisture — press then air dry to below 15% before use
- Gasoline/syngas blending and flexfuel (ethanol blend)/syngas combinations are viable — ethanol's high octane complements wood gas well
- The system requires active agitation on smooth roads and stationary installations — a timer-controlled 12V chain-driven worm gear motor is the practical solution
- Two-tier control separates engine-critical decisions (ECU) from system-level safety and gasifier management (supervisor)
- All timing values require field tuning with wideband O₂ and knock sensor feedback — tables provided are starting points
- Gasifier warmup is 10–15 minutes — the green light readiness indicator is the modern equivalent of the WW2 vent pipe flame test — same logic, human-in-the-loop, operator makes the final switch decision
- A vent valve must route startup gas safely to atmosphere during warmup — solenoid valve, normally open, fails safe
- An oil catch tank is mandatory on syngas and syngas/ethanol combined operation — tar and ethanol blow-by both foul injector tips and contaminate oil rapidly
- Port injection is preferred over direct injection for syngas/ethanol builds — more tolerant of fuel variation, injector tips in cooler location
- Final filtration must be the last element before the mixer — three stage assembly: woven SS pre-filter, sintered 316L SS 10µm, silicon carbide ceramic foam 20–30 PPI
- Filter elements are cleanable with gasoline soak and ultrasonic bath — stainless and ceramic both have indefinite service life if handled correctly
- Oil change intervals must be shortened significantly on wood gas operation — monitor via oil dilution sensor and catch tank fill rate trend
- CO from wood gas is odorless and lethal — a dedicated electrochemical CO detector with automatic shutoff valve is mandatory for any vehicle or enclosed installation

---

<a id="sec-18"></a>
## 18. Commercial Systems, DIY Plans and Salvage Materials

Target: 1.2–4 litre engine displacement. 316 stainless steel minimum grade throughout — tar acids, condensate and thermal cycling destroy mild steel rapidly. DIY builds in stainless are entirely viable using salvaged and surplus materials.

---

### DIY Plans — Free and Paid

**Drive On Wood — Free Plans Library**
The most important free resource. FEMA stratified downdraft plans, Imbert dimensions, Larry Dobson public domain design, Gary Gilmore charcoal gasifier, and overview schematics. Wayne Keith's vehicle gasifier plans available to premium members — the most proven vehicle design in operation.
- Free plans: https://www.driveonwood.com/library/free-gasifier-plans/
- Full library (research papers, mirrored documents, GENGAS): https://www.driveonwood.com/library/
- GENGAS / historical program materials: https://www.driveonwood.com/library/gengas/

**FEMA Simplified Wood Gas Generator — Free PDF**
Original 1989 FEMA emergency gasifier document. Stratified downdraft design. Free download — historical reference and starting point. Note: known tar producer at variable loads — suitable as educational base, upgrade design before engine use.
- https://archive.org/details/femasimplifiedwoodgasgeneratormar1989withbiomassenergyfoundation2001

**Build-A-Gasifier — Free Plans and Downloads**
Collection of free gasifier plans including FEMA design, Imbert dimensions, downdraft design guidelines based on Swedish WW2 experience, and multiple research PDFs. Good starting library.
- Plans: https://www.build-a-gasifier.com/gasifier-plans/
- FEMA specific: https://www.build-a-gasifier.com/fema-gasifier-plans/

**Ben Peterson — Wood Gasifier Builder's Bible (Paid)**
Most popular paid DIY plan set. 238 pages, sizing charts, critical dimensions, engine conversion guide. Stainless steel build in plans. Self-build cost estimate: $2,800–$3,500 using CNC parts in mild steel, $12,500–$15,000 in stainless turn-key. Plans available for download.
- https://www.woodgasifierplans.com/
- Resources and overview: http://www.woodgasifierplans.com/resources/

**GEK Gasifier Wiki — All Power Labs Open Source (Original)**
Original open-source GEK design files, CAD drawings and documentation before All Power Labs went proprietary. Still freely available. Good engineering reference for reactor geometry.
- http://wiki.gekgasifier.com/ (use **HTTP** if **HTTPS** reports a certificate name mismatch)

**FAO — *Wood gas as engine fuel* (Forestry Paper 72)**
United Nations Food and Agriculture Organisation. Comprehensive technical overview of wood gasification for engine use — theory, gasifier types, downdraft design, fuels, gas cleaning, applications, safety. **Official full text (HTML):**
- Contents: https://www.fao.org/4/t0512e/t0512e00.htm
- Chapter 2 (small gasifiers for engines): https://www.fao.org/4/T0512E/T0512e07.htm
- §2.5 (downdraught gasifier design): https://www.fao.org/4/t0512e/T0512e0c.htm  
Mirrors and PDFs may also appear via [Drive On Wood library](https://www.driveonwood.com/library/).

**Handbook of Biomass Downdraft Gasifier Engine Systems — Reed & Das (SERI/NREL, NREL/SP-271-3022)**
US program handbook: design, testing, operation, and manufacture of small-scale downdraft gasifier systems (about 200 kW class). Gas measurement, cleanup, and engine interfacing. **Authoritative access points:**
- NREL Research Hub (publication record): https://research-hub.nrel.gov/en/publications/handbook-of-biomass-downdraft-gasifier-engine-systems/
- UNT Digital Library (digitized PDF): https://digital.library.unt.edu/ark:/67531/metadc1061385/
- OSTI catalog: https://www.osti.gov/biblio/5206099  
Mirrors: [Drive On Wood library](https://www.driveonwood.com/library/), [Build-A-Gasifier plans page](https://www.build-a-gasifier.com/gasifier-plans/)

---

### Salvage and Surplus Materials for Stainless DIY Build

A 316 stainless gasifier can be built largely or entirely from salvaged components. The following sources provide suitable material at low or zero cost.

#### Dairy and Food Industry — Best Source

```
Dairy industry stainless is almost always 316L — the same
grade required for food contact with acidic products.
Exactly correct for gasifier construction.

WHAT TO LOOK FOR:

Milk tanks and vats
  Large cylindrical vessels — ideal gasifier body
  Heavy wall thickness — 3–6mm typical
  316L as standard — food grade requirement
  Dished or conical ends — ready-made hopper shapes
  Tri-clamp fittings already fitted — ideal for clean-out ports
  Insulated outer jacket on some — bonus thermal benefit

Dairy pipework and fittings
  1.5" to 4" diameter — gas piping, nozzle manifolds
  Tri-clamp couplings — perfect for inspection and clean-out access
  Elbows, tees, reducers — all 316L as standard
  Gaskets — replace with Viton for tar resistance

Cheese vats and processing vessels
  Large flat-bottomed cylindrical tanks
  Heavy gauge 316L
  Often have agitator shaft penetrations already —
    useful as stirrer/agitator shaft entry points

Pasteuriser plates and heat exchangers
  Plate heat exchangers — excellent gas cooler conversion
  Already designed for liquid/gas thermal exchange
  316L plates, high surface area, compact
  Tri-clamp connections standard

Butter churns and cream separators
  Smaller cylindrical vessels
  Useful for filter housings, secondary cyclone bodies

Milk pipeline inline filters
  Some already have filter basket housings in 316L
  Size may suit final filter housing directly

Where to find:
  Farm equipment dealers — used dairy equipment
  Farm auctions — decommissioned dairy farms
  Dairy equipment recyclers
  Online farm machinery classifieds (Finn.no, Blocket.se)
  Local dairy farms upgrading equipment
  Slaughterhouse equipment suppliers
```

#### Water Heater Inner Tanks

```
IMPORTANT: Stainless inner tanks only — not enamel lined.

Stainless inner tank water heaters:
  Typically 1.5–3mm 304 or 316 stainless
  Cylindrical — ideal reactor body form
  Already pressure rated — robust construction
  Typically 100–300 litre capacity
  Domed ends — structurally excellent

How to identify stainless inner tank:
  Look for "stainless steel tank" or "rustfri" marking
  Magnet test — 304/316 is weakly magnetic or non-magnetic
  Cut-open scrapped units — visible at scrap dealers

Note on grade:
  304 stainless in water heaters is common
  Suitable for outer body and hopper sections
  Upgrade to 316L for reactor throat, nozzle ring,
  and reduction zone — highest tar acid exposure

Where to find:
  Plumbing and heating suppliers — damaged/returned units
  Scrap metal dealers — often whole units available
  Heating contractor waste — old boiler rooms
  Building demolition — old plant rooms
```

#### Food Processing Industry

```
Brewing and beverage industry:
  Fermentation vessels — large 316L cylinders
  Bright tanks — pressure rated, heavy gauge
  CIP (clean-in-place) pipework — 316L throughout
  Keg bodies — small pressure vessels, heavy wall

Bakery and food processing:
  Mixing bowl liners — 316L, various sizes
  Conveyor trays and hoppers — flat 316L sheet
  Food grade piping — various diameters

Fish processing industry (relevant in Norway):
  316L is mandatory for fish contact
  Fish tanks, processing tables, pipework
  All 316L — chloride resistance required for
  salt fish environment — same grade needed for gasifier

Pharmaceutical/chemical industry surplus:
  Highest grade stainless — 316L or 316Ti standard
  Reactors, vessels, pipework — all suitable
  Often very heavy gauge — overkill but excellent

Where to find in Norway:
  Nofima (food research) surplus sales
  Fish processing plant equipment recyclers
  Brewery equipment dealers
  Industriloppis / industrial surplus auctions
  Finn.no — søk "rustfri tank", "syrefast", "316 stål"
```

#### Restaurant and Catering Equipment

```
Commercial kitchen equipment:
  Worktops and sinks — 316L sheet material
  Stockpots — heavy gauge 316L, various sizes
  Gastronorm containers — flat sheet, pressings
  Exhaust hoods — large flat 316L sheet

Useful for:
  Flat plate fabrication
  End caps and covers
  Filter housing components
  Heat shield panels
```

#### Scrap and Surplus Metal Dealers

```
Specify when sourcing:
  "Syrefast stål" (Norwegian/Swedish for acid-resistant steel)
  "316L rustfri" — standard specification term
  "AISI 316L" or "EN 1.4404" — European standard designation

Test for stainless at scrap yard:
  Magnet — 316L is non-magnetic or very weakly magnetic
  Angle grinder spark test — stainless gives short, dull sparks
    vs mild steel which gives long bright star-burst sparks
  Ask for material certificate if available — MTR document

Avoid:
  409 stainless — ferritic, magnetic, poor corrosion resistance
  430 stainless — decorative grade, insufficient for gasifier
  Galvanised steel — zinc vapour is toxic in gasifier
  Mild steel — will corrode rapidly in tar acid environment
```

#### Key Components and Suitable Salvage Sources

| Component | Material Needed | Best Salvage Source |
|---|---|---|
| Reactor body | 316L cylinder | Dairy vat, water heater tank, fermentation vessel |
| Hopper | 304 or 316L | Dairy cone bottom tank, water heater top |
| Nozzle ring | 316L tube/plate | Dairy pipework, food grade tube |
| Reduction zone throat | 316L heavy plate | Dairy vessel offcuts, plate heat exchanger |
| Gas outlet pipe | 316L tube | Dairy CIP pipework, brewery pipework |
| Cyclone body | 316L sheet | Dairy vat offcuts, food processing sheet |
| Filter housing | 316L cylinder | Dairy inline filter, small milk vessel |
| Gas cooler | 316L plate HX | Dairy plate heat exchanger — ideal direct use |
| Piping | 316L tube | Dairy/brewery CIP pipe, fish plant pipework |
| Fittings | 316L tri-clamp | Dairy fittings — standard and abundant |
| Fasteners | A4-70 stainless | Marine hardware suppliers |

---

### Commercial Kits and Systems — Suppliers

**ThriveOffGrid (formerly Vulcan Gasifier)**
Longest-running small engine gasifier manufacturer in the US. CXF charcoal cross-fire and DFX dual fuel series cover 1.2–4L engine range. 2025/2026 models include stainless steel hopper lid, water tank and side brackets.
- https://www.thriveoffgrid.net/
- https://www.thriveoffgrid.net/cxfcrossfiregasifier

**All Power Labs — GEK Gasifier Kit**
University and OEM grade system. Complete gas-making system from fuel feed through filter with full automation, wideband O₂ sensor, auger feed, cyclone and filter. Ships assembled and commissioned. Engineered to a high standard — not a hobby kit.
- https://www.allpowerlabs.com/products/gasifier-kits

**Clinch Energy Solutions**
Turn-key charcoal gasifier kit. Five-component assembly, rated to 5.7L engine / 25 kW generator. Targets 10–15 minute startup. Straightforward small engine integration.
- https://www.clinchenergysolutions.com/gasifiers

**OffGrid48 — Ben Peterson Design**
DIY build kits with laser-cut parts available in mild steel or stainless steel, 1/8" and 1/4" thickness. Complete kit includes all structural components, hardware, insulation, motors and electronics. Best stainless DIY kit option in the US market.
- http://www.offgrid48.com/diy-wood-gasifier-kits.html

**WoodGasifierPlans — Plans and Used Units**
Plans for self-build plus occasional used units for sale. Some used units in 304/316 stainless construction. Good resource for downdraft design documentation.
- https://www.woodgasifierplans.com/pages/gasifiers-for-sale

**Drive On Wood — Wayne Keith Community**
Not a supplier but the most important vehicle gasifier resource online. Free plans, build documentation, forums, vehicle conversion experience. Essential reference before any purchase or build decision.
- https://www.driveonwood.com/

---

### European

**Spanner Re² GmbH — Germany**
Leading European wood chip CHP gasifier manufacturer. Over 1,100 installations worldwide. ISO 9001 certified. Wood chips or pellets. HKA series 35–300+ kW electric. Stainless and high-grade steel construction throughout. Refurbished units also available.
- Main site: https://re2.energy/en/
- Biomass power plants overview: https://re2.energy/en/biomass-power-plants/overview-biomass-power-plant
- Refurbished systems: https://re2.energy/en/services/refurbished-systems

**Volter — Finland**
Finnish manufacturer — Scandinavian climate design, well suited to Norwegian conditions and feedstocks. Walter CHP unit: 40 kW electric / 100 kW heat from wood chips or pellets. Indoor and outdoor (shipping container) versions. High automation, remote monitoring capable. Most relevant European supplier for Norwegian installation.
- https://volter.fi/en/volter-products/
- Technology overview: https://volter.fi/en/volter-products/technology/

**Fröling — Austria**
Austrian boiler and gasification manufacturer. Pioneer in wood gasification technology. Partners with MAN Engines for CHP applications. High build quality, well established in European agricultural and industrial market.
- https://www.froeling.com/en/

**Burkhardt — Germany**
Wood pellet gasifier CHP units. Pellet feedstock suits Norwegian supply chain well. Well proven in European farm and light industrial installations.
- https://www.burkhardt.de/en/

---

### Note on 316 Stainless Specification

Most commercial CHP units (Spanner, Volter, Burkhardt) use high-grade stainless and alloy steel in critical hot zones as standard — these are engineered systems, not kits. For kit and self-build suppliers, specifically request 316L stainless for:

```
Reactor inner wall and throat
Reduction zone components
Gas outlet piping
Filter housing and internals
All fasteners in hot zones

316L preferred over 304:
  Superior resistance to tar acids
  Better chloride corrosion resistance
  Suitable for sustained high-temperature cycling
```

Contact suppliers directly to confirm material specifications before ordering — stainless grade is not always stated in general product descriptions.
