# Engine Spec — One Pixel Home (Space Mission Design Game)

Tags used on every value:

- **\[NASA:Sx\]** — taken from a NASA source listed in §8 (page numbers where known).  
- **\[PHYS\]** — standard physics formula; constants from \[NASA:S1\].  
- **\[GAME\]** — game-balance value, plausible but simplified. Shown to players as "simplified".  
- **\[VERIFY\]** — must be checked against the cited source or a mentor before the demo.

---

## 1\. Game flow (mirrors NASA's real process)

| Phase | Based on | What players do |
| :---- | :---- | :---- |
| 1\. Mission Idea | CML 1 \[NASA:S4\] | Science role picks an **objective** (scenario). Requirements appear. |
| 2\. Concept & Trades | CML 2–3, MDL "Concept Development / Trade Studies" \[NASA:S4, S5\] | Each role configures its segment. Live margins. Save **Design A / Design B** and compare. |
| 3\. Point Design | CML 4, MDL "Point Design" \[NASA:S4, S5\] | Lock one design. Every role must **sign off** its checklist. |
| 4\. Go / No-Go | — | All active roles vote. Unanimous Go → launch. |
| 5\. Simulation | — | Day-by-day mission timeline with events. |
| 6\. Debrief | Webb "Build it yourself" game idea \[NASA:S7\] | Score, stars, every reason with its source, "closest real NASA mission". |

## 2\. Roles \= NASA's four mission segments \[NASA:S3\]

| Role | Segment | Owns |
| :---- | :---- | :---- |
| Science | User segment | objective, instruments, requirements |
| Space | Space segment | bus size, power (panels \+ battery), ADCS, mass/volume |
| Launch | Launch segment | launch option → orbit (altitude, inclination) |
| Ground | Ground segment | radio band, ground station(s), operations plan |

Solo mode: one player sees all four tabs. Co-op: 2–4 players, each takes ≥1 tab.

## 3\. Types (TypeScript)

```ts
type SourceId = `NASA-S${number}` | 'PHYS' | 'GAME';  // S1–S11, see §8
interface Sourced<T> { value: T; sourceId: SourceId; verify?: boolean }

type Role = 'science' | 'space' | 'launch' | 'ground';

interface Scenario {
  id: string; title: string; story: string;
  maxLatitudeDeg: number;      // highest latitude the target needs
  targetLatDeg: number;        // representative latitude used for revisit
  maxGsdM: number;             // required ground sample distance (m)
  maxRevisitDays: number;      // required revisit at target latitude
  requiredBands: InstrumentBand[];
  minLifetimeMonths: number;
  launchDeadlineMonths: number;
  budgetCredits: number;       // [GAME]
}

type InstrumentBand = 'visible' | 'multispectral' | 'thermal';

interface Instrument {
  id: string; name: string; band: InstrumentBand;
  massKg: number; volumeU: number; powerW: number;
  gsdAtRefM: number; refAltitudeKm: number;   // GSD scales with altitude (§4.6)
  pixelsAcross: number;                       // swath = GSD * pixelsAcross
  dataMbPerImage: number; imagesPerDay: number;
  costCredits: number; sourceId: SourceId;
}

interface BusOption { id: '3U' | '6U' | '12U'; volumeU: number; maxMassKg: number; baseMassKg: number; basePowerW: number; costCredits: number }
interface SolarOption { id: string; areaM2: number; cosFactor: number; massKg: number; volumeU: number; costCredits: number }
interface BatteryOption { id: string; capacityWh: number; massKg: number; volumeU: number; costCredits: number }
interface RadioOption { id: 'UHF' | 'S' | 'X'; rateMbps: number; txPowerW: number; massKg: number; volumeU: number; costCredits: number; licenseDays: number; needsPointing: boolean }
interface LaunchOption { id: string; altitudeKm: number; inclinationDeg: number; maxBusU: number; costPerU: number; delayMonths: number; sourceId: SourceId }
interface GroundStation { id: string; latitudeDeg: number; bands: RadioOption['id'][]; costPerMonth: number }

interface MissionDesign {
  scenarioId: string;
  instrumentIds: string[];
  busId: BusOption['id']; solarId: string; batteryId: string; adcs: 'basic' | 'fine';
  radioId: RadioOption['id']; stationIds: string[];
  launchId: string;
  satelliteCount: 1 | 2 | 3 | 4;   // constellation: revisit / n, hardware + launch cost × n
  signOffs: Record<Role, boolean>;
}

interface Reason { code: string; severity: 'fail' | 'warn' | 'info' | 'good'; role: Role; message: string; sourceId: SourceId }

interface DesignCheck {           // live, recomputed on every change
  massKg: { used: number; max: number };
  volumeU: { used: number; max: number };
  powerW: { orbitAvgGen: number; orbitAvgLoad: number };
  battery: { neededWh: number; capacityWh: number };
  dataMbPerDay: { generated: number; downlinkCapacity: number };
  budget: { spent: number; total: number; reservePct: number };
  timeline: { monthsToLaunch: number; deadline: number };
  coverage: { maxLatReachDeg: number; gsdM: number; revisitDays: number };
  reasons: Reason[];
}

interface MissionResult {
  success: boolean; stars: 0 | 1 | 2 | 3; score: number;
  breakdown: { science: number; efficiency: number; collaboration: number };
  dataReturnedGb: number; lifetimeMonths: number;
  events: { day: number; text: string; sourceId: SourceId }[];
  reasons: Reason[];
  closestRealMission: { name: string; why: string; sourceId: SourceId };
}

// Public API (pure functions)
// evaluateDesign(design, scenario, catalog): DesignCheck
// simulateMission(design, scenario, catalog, seed: number): MissionResult
// compareDesigns(a: DesignCheck, b: DesignCheck): Reason[]
```

## 4\. Formulas

Constants \[NASA:S1\]: R\_E \= 6378.137 km · μ \= 398600.4418 km³/s² (fact sheet: 0.39860×10⁶) · S \= 1361.0 W/m².

### 4.1 Orbit \[PHYS\]

- a \= R\_E \+ h  
- Period T \= 2π·√(a³/μ)  · orbits/day \= 86400 / T

### 4.2 Eclipse \[PHYS, simplified\]

- Worst-case (Sun in orbit plane): eclipse fraction f\_ecl \= asin(R\_E / a) / π  
- Dawn-dusk sun-synchronous orbit: f\_ecl ≈ 0 most of the year \[GAME simplification, VERIFY\].

### 4.3 Power

- Peak W \= S · A · η · packing, with η \= 0.295 (triple-junction cells) \[GAME, VERIFY: S9\], packing \= 0.8 \[GAME\]  
- Orbit-average generation \= Peak · cosFactor · (1 − f\_ecl). cosFactor: body-mounted 0.35, deployable sun-tracking 0.8 \[GAME\]  
- Load \= bus base power \+ instruments (duty-cycled, \[GAME\] 30%) \+ radio (only during contacts) \+ fine ADCS  
- **Fail if** generation \< load.  
- Battery needed (Wh) \= load\_W · t\_ecl\_h / DoD\_max, DoD\_max \= 0.3 \[GAME, VERIFY\]. **Fail if** capacity \< needed.

### 4.4 Ground contact & data \[PHYS \+ GAME\]

- Visibility half-angle (min elevation ε \= 10°): λ \= acos(R\_E·cos ε / a) − ε  
- Max pass duration \= T · λ / π ; average pass \= 0.7 × max \[GAME\]  
- Passes/day per station: lookup table by (station latitude, orbit inclination) \[GAME, VERIFY with GMAT/mentor\]. University mid-latitude station ≈ 4/day · commercial network ≈ 6/day · polar station ≈ 10/day only if inclination \> 80°, else 0\.  
- Downlink capacity (Mb/day) \= Σ passes · avgPass\_s · rate\_Mbps · 0.7 link efficiency \[GAME\]  
- Data generated (Mb/day) \= Σ imagesPerDay · dataMbPerImage · 8  
- **Warn if** generated \> capacity (data stays onboard and is lost when storage fills). **Fail if** capacity \< 50% of generated.

### 4.5 Coverage \[PHYS\]

- Max latitude reachable \= i (if i ≤ 90°) else 180° − i.  
- **Fail if** scenario.maxLatitudeDeg \> max latitude reachable (e.g. ISS orbit 51.6° cannot see the Arctic).

### 4.6 Resolution & revisit \[PHYS, simplified\]

- GSD(h) \= gsdAtRef · h / refAltitude ; swath\_km \= GSD · pixelsAcross / 1000  
- Revisit days ≈ (40075 · cos(targetLat) / orbitsPerDay) / swath\_km / satelliteCount  
- **Fail if** GSD \> scenario.maxGsdM ; **warn if** revisit \> scenario.maxRevisitDays.

### 4.7 Mass & volume

- Mass \= bus base \+ Σ components. **Fail if** \> bus max. Max ≈ 1.33 kg per U ("a 1U CubeSat is a 10 cm cube with a mass of approximately 1 to 1.33 kg") \[NASA:S2 p.4\].  
- Volume \= Σ component U. **Fail if** \> bus U (minus 1U reserved for avionics \[GAME\]).

### 4.8 Launch compatibility

- **Fail if** bus U \> launch.maxBusU. Dispensers: 3U P-POD and 6U dispensers \[NASA:S2 pp.5–6\]. ISS deployment by astronauts "typically 1–3 months after arrival" \[NASA:S2 p.39\].

### 4.9 Budget & reserve

- Spent \= satelliteCount · (components \+ bus \+ U · costPerU) \+ stations cost/month · 12\.  
- **Warn if** reserve \< 10% — NASA guidance: reserves of at least 10% \[NASA:S2 pp.13–14\]. Credits themselves are \[GAME\].

### 4.10 Time

- Months to launch \= design/build (9–24 months depending on complexity; "as little as 9 months", typically 18–24) \[NASA:S2 p.9\]  
  + licensing: RF licence ≥ 90 days \[NASA:S2 pp.49–50\]; Earth-imaging remote-sensing licence ≈ 120 days \[NASA:S2 pp.51–53\] (run in parallel → take max)  
  + launch delay.  
- **Fail if** \> scenario.launchDeadlineMonths.

### 4.11 Lifetime \[GAME, VERIFY with mentor\]

- Altitude → orbital lifetime table: 400 km ≈ 1–2 yr · 550 km ≈ 5–10 yr · 700 km ≈ 25+ yr (solar-activity dependent).  
- **Warn if** lifetime \< scenario.minLifetimeMonths. **Fail if** lifetime \> 25 yr (debris rule; CubeSat 101 requires an Orbital Debris Assessment Report \[NASA:S2 pp.54–56\]; numeric limit to VERIFY — note the FCC adopted a 5-year post-mission rule in 2022).

## 5\. Scenarios (MVP: \#1 fully, \#2 if time)

| \# | Objective | maxLat | targetLat | maxGSD | revisit | bands | lifetime | deadline | budget |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Crop Health Watch | 60° | 30° | 30 m | 30 d | multispectral | 12 mo | 24 mo | 2100 cr |
| 2 | Polar Ice Tracker | 80° | 30°\* | 100 m | 15 d | visible or thermal | 24 mo | 24 mo | 1000 cr |

\*Keep 30° for MVP so balancing below holds; raising it shortens revisit (cos term).

Lessons built in (checked by brute force over the §6 catalog, see §7):

- Scenario 1: no single satellite meets the 30-day revisit (≈45–60 d) → players discover a **2-satellite constellation** (≈1800–1850 cr, 6U \+ multispectral \+ fine ADCS \+ wings, SSO). Several valid variants (S- vs X-band, launch option).  
- Scenario 2: the cheap ISS ride (51.6°) cannot reach 80° → must go SSO; cheapest valid ≈ 2 × 3U visible camera (≈750 cr).

## 6\. Catalog (all \[GAME\] unless tagged — keep in `catalog.ts`)

**Bus:** 3U (max 4.0 kg, base 1.2 kg, base 2 W, 60 cr) · 6U (8.0 kg, 2.5 kg, 4 W, 120 cr) · 12U (16.0 kg, 4.5 kg, 7 W, 220 cr). Max mass ≈ 1.33 kg/U \[NASA:S2 p.4\].

**Instruments:**

| id | band | kg | U | W | GSD @550 km | px across | MB/img | img/day | cr |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| rgb-cam | visible | 0.4 | 0.5 | 2.5 | 40 m | 2000 | 8 | 40 | 60 |
| ms-imager | multispectral | 1.2 | 1.0 | 5 | 20 m | 2500 | 30 | 30 | 180 |
| tir-imager | thermal | 1.5 | 1.5 | 7 | 90 m | 640 | 4 | 40 | 220 |

**Solar:** body-mounted (0.034 m², cos 0.35, 0 kg, 0 U, 20 cr) · deployable 2-wing (0.136 m², cos 0.8, 0.6 kg, 0.3 U, 90 cr). **Battery:** 20 Wh (0.2 kg, 0.2 U, 20 cr) · 40 Wh (0.4 kg, 0.4 U, 40 cr) · 80 Wh (0.8 kg, 0.8 U, 80 cr). **ADCS:** basic (0 extra) · fine (0.5 kg, 0.5 U, \+2 W, 120 cr) — required for X-band and for GSD \< 30 m. **Radio:** UHF 0.0096 Mbps (2 W, 0.1 kg, 0.1 U, 15 cr, amateur coordination) · S-band 2 Mbps (8 W, 0.3 kg, 0.3 U, 70 cr) \[GAME\] · X-band 12.5 Mbps (15 W, 0.5 kg, 0.5 U, 150 cr, needs fine ADCS) — NASA CubeSat X-band transmitter design rate \[NASA:S11\]. Bands UHF/S/X/Ka are the typical small-sat bands \[NASA:S9\]. Licensing ≥ 90 days for all \[NASA:S2 pp.49–50\]. **Launch:**

| id | orbit | maxBusU | cr/U | delay |
| :---- | :---- | :---- | :---- | :---- |
| iss-deploy | 400 km, 51.6° \[NASA:S10\] | 6 | 15 | \+2 mo (1–3 mo after arrival \[NASA:S2 p.39\]) |
| rideshare-sso | 550 km, 97.6° SSO | 12 | 25 | \+4 mo |
| dedicated-small | choose 450–650 km, any inclination | 12 | 60 | \+1 mo |

**Ground stations:** University mid-latitude (lat 35°, UHF/S, 0 cr/mo) · Commercial network (S/X, 15 cr/mo) · Polar station (lat 78°, S/X, 25 cr/mo).

## 7\. Sanity tests (must pass; also great evidence for the "Validity" criterion)

Computed with the formulas above:

- h \= 400 km → T ≈ 92.6 min, 15.56 orbits/day, eclipse ≈ 36.1 min, max pass ≈ 6.2 min  
- h \= 550 km → T ≈ 95.6 min, eclipse ≈ 35.6 min, max pass ≈ 7.9 min  
- h \= 705 km, swath 185 km → revisit ≈ 14.9 days. Landsat 9: 705 km, 98.2°, 185 km swath, 16-day temporal resolution \[NASA:S6\] → our model is within \~7%.  
- 6U with deployable wings @550 km → peak ≈ 43.7 W, orbit-average ≈ 21.9 W.  
- ISS orbit (51.6°) \+ Polar Ice scenario (80°) → coverage FAIL.  
- **Balancing test:** enumerate every catalog combination per scenario → assert ≥ 2 valid designs, cheapest valid leaves ≥ 10% reserve, and no valid design uses 1 satellite in Scenario 1\. Re-run after any catalog change.

## 8\. Sources (cite all on the project page)

- **S1** NASA Earth Fact Sheet — [https\://nssdc.gsfc.nasa.gov/planetary/factsheet/earthfact.html](https://nssdc.gsfc.nasa.gov/planetary/factsheet/earthfact.html)  
- **S2** NASA CSLI, *CubeSat 101: Basic Concepts and Processes for First-Time CubeSat Developers* (2017) — [https\://www\.nasa.gov/wp-content/uploads/2017/03/nasa\_csli\_cubesat\_101\_508.pdf](https://www.nasa.gov/wp-content/uploads/2017/03/nasa_csli_cubesat_101_508.pdf)  
- **S3** NASA, *Design and Develop Science Missions* — [https\://www\.nasa.gov/wp-content/uploads/2025/04/250323-design-and-develop-science-mission.pdf](https://www.nasa.gov/wp-content/uploads/2025/04/250323-design-and-develop-science-mission.pdf)  
- **S4** JPL, *Concept Maturity Levels* — [https\://dataverse.jpl.nasa.gov/dataset.xhtml?persistentId=hdl:2014/44299](https://dataverse.jpl.nasa.gov/dataset.xhtml?persistentId=hdl:2014/44299)  
- **S5** NASA GSFC Mission Design Lab — [https\://etd.gsfc.nasa.gov/capabilities/integrated-design-center/mission-design-lab/](https://etd.gsfc.nasa.gov/capabilities/integrated-design-center/mission-design-lab/)  
- **S6** NASA Landsat 9 mission page — [https\://science.nasa.gov/mission/landsat-9](https://science.nasa.gov/mission/landsat-9)  
- **S7** NASA Webb "Build it yourself" game — [https\://science.nasa.gov/mission/webb/game-build-it-yourself/](https://science.nasa.gov/mission/webb/game-build-it-yourself/)  
- **S8** NASA GMAT — [https\://software.nasa.gov/software/GSC-19468-1](https://software.nasa.gov/software/GSC-19468-1) (reference for orbit checks)  
- **S9** NASA *Small Spacecraft Technology State-of-the-Art*, Communications chapter — [https\://www\.nasa.gov/smallsat-institute/sst-soa/communications](https://www.nasa.gov/smallsat-institute/sst-soa/communications)  
- **S10** NASA ISS FAQ (51.6° inclination) — [https\://www\.nasa.gov/?p=532244](https://www.nasa.gov/?p=532244) ; altitude ≈ 400 km (ESA lists 370–460 km: [https\://m.esa.int/Our\_Activities/Human\_Spaceflight/International\_Space\_Station/ISS\_International\_Space\_Station](https://m.esa.int/Our_Activities/Human_Spaceflight/International_Space_Station/ISS_International_Space_Station))  
- **S11** NASA NTRS, high-rate CubeSat communication system (12.5 Mbps X-band) — [https\://ntrs.nasa.gov/archive/nasa/casi.ntrs.nasa.gov/20150000169.pdf](https://ntrs.nasa.gov/archive/nasa/casi.ntrs.nasa.gov/20150000169.pdf)

## 9\. Still to verify with a mentor (Saturday, Mission 2\)

- Orbital lifetime by altitude (§4.11) and the numeric debris rule (25-year vs FCC 5-year).  
- Battery max depth of discharge 30% (§4.3) and solar cell efficiency 29.5%.  
- Dawn-dusk SSO eclipse ≈ 0 simplification (§4.2).  
- Passes/day table per station (§4.4) — or check one case in GMAT.