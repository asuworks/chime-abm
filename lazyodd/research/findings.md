# ODD Research Findings

## Model Overview
- **Name:** CHIME ABM (Communicating Hazard Information in the Modern Environment)
- **Authors:** Watts J., Morss R.E., Barton C.M., Demuth J.L. (and collaborators)
- **Purpose:** Virtual laboratory to investigate dynamics of hazardous weather communication and decision making during hurricane threats, specifically how information flow through the modern information environment affects household evacuation decisions.
- **Platform:** NetLogo 6.2.1
- **Input Sources:** CHIME.nlogo (11,831 lines), README.md, How To Use CHIME.md, STORMS/ directory (9 storm scenarios), REGION/ directory (3 geographic regions with GIS data)
- **Input Quality Assessment:** Code + minimal docs
- **Model Complexity:** Moderate (6 entity types, ~10 submodels)
- **ODD Format Decision:** ODD+2 with extensions (recommended — model has rich design rationale needs given its theoretical grounding in protective action decision-making)

---

## Element 1: Purpose and Patterns

### Findings
The CHIME ABM was designed to function as a virtual laboratory to test hypotheses about information flow and decision making during weather-related threats, specifically hurricanes. The model investigates how citizen-agents collect, share, and interpret hazard information from multiple sources (forecasters, broadcasters, social media aggregators, social networks) and how that information processing leads to protective action decisions (evacuation, other protective actions, or no action).

The higher-level purpose is **explanation and theoretical exploration**: understanding how the modern information environment — with its multiple, sometimes conflicting information sources — shapes household-level hurricane evacuation decisions.

Users can modify parameters to explore the relative effect of changes on patterns of household evacuation and other protective decisions, including:
- Weighting of different information types (forecast, evacuation orders, environmental cues)
- Inclusion of census demographic information
- Different hurricanes affecting different regions
- Perfect versus imperfect forecasts

The model does not produce a single verifiable outcome. Output consists of timing and location of household evacuations, which can be compared to historical evacuation records.

### Sources
- README.md:3 (model description)
- How To Use CHIME.md:1-3 (purpose as virtual laboratory)
- CHIME.nlogo:1-40 (procedure description header)

### Confidence
- `DOC_STATED` — Purpose is explicitly stated in README and How To Use guide
- `CODE_VERIFIED` — Model structure confirms the stated purpose

### Inferred Patterns (from code observation metrics)
Based on output procedures (Save-Global-Evac-Statistics), the model is evaluated against:
1. **Evacuation rates by exposure zone**: % evacuated among coastal/inland residents within 64-kt, 34-kt, and outside wind radii (6 categories)
2. **Evacuation timing relative to landfall**: Histogram in 6-hour bins from 90h before to 18h after landfall
3. **Official order timing**: When officials issued orders relative to landfall
4. **Sensitivity to information weights**: BehaviorSpace experiments sweep forc-weight, evac-weight, envc-weight

### Knowledge Gaps
- **KNOWLEDGE_GAP_1**: Quantitative target evacuation rates/timing from historical records not encoded in code
- **KNOWLEDGE_GAP_2**: Patterns the model failed to reproduce — not determinable
- **KNOWLEDGE_GAP_3**: Specific hypotheses from Watts et al. 2019 — paper not available in repository

---

## Element 2: Entities, State Variables, and Scales

### Findings

**Agent breeds (entity types):**

| Entity | Count | Description |
|--------|-------|-------------|
| `citizen-agents` | User-set (0–5000, default 2000) or census-derived | Households making evacuation decisions |
| `forecasters` | 1 | Publishes NHC-style forecast advisories |
| `officials` | 1 per county (from GIS county-seats) | County emergency managers issuing evacuation orders |
| `broadcasters` | User-set (0–10, default 10) | Traditional media translating forecasts |
| `aggregators` | User-set (0–10, default 10) | Social media / online information aggregators |
| `hurricanes` | 0–1 | Visualization of hurricane position (not behavioral) |

**Utility breeds (non-behavioral):** `forcstxs` (forecast cone visualization), `drawers` (storm track drawing), `tracts` (census tract points).

**Citizen-agent state variables:**

| Variable | Type | Initialization | Description |
|----------|------|---------------|-------------|
| `evac-zone` | String ("A","B","C","") | Check-Zone procedure | Perceived evacuation zone based on coastal distance |
| `self-trust` | Float [0.6, 1.0] | 0.6 + random-float 0.4 | Confidence in own forecast interpretation |
| `trust-authority` | Float [0, 1] | random-float 1 | Trust in officials' evacuation orders |
| `risk-life-threshold` | Float | Normal(14, 2) | Threshold for evacuating (risk to life) |
| `risk-property-threshold` | Float | Normal(0.7 * risk-life, 0.5) | Threshold for other protective action |
| `info-up` | Float | Normal(0.4 * risk-life, 0.5) | Threshold to increase info collection frequency |
| `info-down` | Float | Normal(0.1 * risk-life, 0.5) | Threshold to decrease info collection frequency |
| `decision-module-interval` | Integer | Normal(12, 2), rounded | Hours between decision module runs |
| `decision-module-turn` | Integer | random 10 | Counter for scheduling decision module |
| `my-network-list` | List of [agent, trust-factor] | Scale-free network algorithm | Social network connections with trust weights |
| `broadcaster-list` | List of [agent, trust-factor] | Random subset of broadcasters | Media connections with trust weights |
| `aggregator-list` | List of [agent, trust-factor] | Random subset of aggregators | Online info source connections |
| `interpreted-forecast` | List | [] | Agent's blended interpretation of the hurricane forecast |
| `memory` | List | [self-trust, interpreted-forecast] | Previous forecast interpretation |
| `environmental-cues` | 0 or 1 | 0 | Whether agent is experiencing tropical storm winds |
| `risk-estimate` | List | [0] | History of risk calculations |
| `completed` | List | [] | History of decisions made (["evacuate" clock counter], etc.) |
| `evacuated?` | Boolean | false | Whether agent has evacuated |
| `evac-day` | Number | 0 | Day of evacuation |
| `final-risk-assesment` | Float | — | Current total risk value |
| `risk-forecast` | Float | — | Forecast component of risk (weighted) |
| `risk-official-orders` | Float | — | Evacuation order component (weighted) |
| `risk-environmental-cues` | Float | — | Environmental cue component (weighted) |
| `risk-packet` | List | [risk, env-cues, 0, 0] | Summary risk info for output |
| Census variables | Various | From GIS census tracts | kids-under-18?, adults-over-65?, limited-english?, food-stamps?, no-vehicle?, no-internet?, county-fips, census-tract-number, tract-information, my-tract-population, my-tract-household |

**Official state variables:** `orders` (0/1), `distance-to-track`, `county-id`, `when-issued`

**Forecaster state variables:** `current-forecast`

**Broadcaster state variables:** `broadcast` (current interpreted forecast)

**Aggregator state variables:** `info` (current interpreted forecast)

**Patch state variables:** `density` (population density from GIS), `elev` (elevation from GIS), `county` (county ID from GIS), `alerts` (0/1 evacuation alert), `land?` (boolean)

**Global variables:** `clock` [day, hour], `best-track-data`, `hurricane-coords-best-track`, `forecast-matrix`, `scale` (nm per grid cell), `grid-cell-size` (degrees per grid cell), `re0-0` (coordinate system origin), `land-patches`, `ocean-patches`, `coastal-patches`

**Spatial scale:**
- Grid: GIS-derived raster, world extent -100 to 100 (x) by -75 to 75 (y) patches
- Grid cell size: variable, derived from GIS envelope (~0.1 degrees, approximately 6 nm)
- Three regions: Florida, Gulf states, Gulf+Southeast
- Coordinate system: projected from lat/lon to NetLogo grid via GIS extension

**Temporal scale:**
- 1 tick = 1 hour
- Simulation runs until hurricane passes through domain (typically 100–140 ticks depending on storm)
- Decision module runs every `decision-module-interval` ticks per agent (default ~12 hours, adjustable via feedback)

### Sources
- CHIME.nlogo:46-171 (variable declarations)
- CHIME.nlogo:1082-1167 (Create-Citizen-Agent-Population)
- CHIME.nlogo:1189-1262 (Create-Other-Agents)
- CHIME.nlogo:312-398 (Load-GIS)
- CHIME.nlogo:3005-3030 (GRAPHICS-WINDOW dimensions)

### Confidence
- `CODE_VERIFIED` — All state variables confirmed from code declarations and initialization procedures

### Inferences and Knowledge Gaps
- **risk-life-threshold Normal(14,2)**: `INFERRED` — Calibrated so that the Gaussian risk function (peak ~10 when height is small) crosses this threshold only for citizens near the coast, within the cone, facing a major hurricane ~36h out. The value 14 ensures only high-risk citizens evacuate, producing realistic 15-20% evacuation rates (per How To Use CHIME.md example).
- **self-trust [0.6, 1.0]**: `INFERRED` — Lower bound of 0.6 ensures all citizens retain meaningful weight on their own memory/interpretation, preventing pure herd behavior. Reflects assumption that people maintain some baseline self-confidence.
- **decision-module-interval Normal(12,2)**: `INFERRED` — Represents that households do not continuously monitor hurricane information; ~12 hours (roughly twice daily) is a realistic check-in frequency during a multi-day hurricane threat.
- **KNOWLEDGE_GAP_4**: Exact calibration methodology for these parameters (empirical data vs. expert judgment vs. sensitivity analysis) not documented in code.

---

## Element 3: Process Overview and Scheduling

### Findings

Each time step (1 hour), the following processes execute in this exact order:

1. **Move-Hurricane** — Hurricane visualization advances to next position from interpolated best-track data
2. **Publish-Forecasts** (asked of forecasters) — Forecaster publishes current NHC-style advisory matched to simulation time
3. **Publish-New-Mental-Model** (reporter) — Interpolates forecast to hourly data, creating an hourly forecast track with intensity, location, and cone of uncertainty
4. **Coastal-Patches-Alerts** — Coastal patches evaluate if storm is within distance/intensity thresholds to trigger alerts
5. **Issue-Alerts** (asked of officials whose county has alerted patches) — Officials issue evacuation orders
6. **Broadcasters update** — All broadcasters receive the interpolated forecast
7. **Aggregators update** — Each aggregator has a 1/3 chance of updating its information each tick
8. **Citizen-agent decision cycle** (asked of all citizen-agents):
   - If NOT evacuated: check if it's their turn (decision-module-turn < decision-module-interval); if yes, run **Decision-Module**
   - If ALREADY evacuated: still collect and process information (Process-Forecasts) so their network connections get updated info
9. **Color updates** — Visual updates for agent display
10. **Clock update** — Update simulation clock from best-track data
11. **Data saving** — Conditional saving of timestep data, images, county evacuation data
12. **Stop check** — If hurricane has passed, save final output and stop

**Scheduling notes:**
- All processes are **synchronous** within each tick
- Citizen-agents execute in NetLogo's default random order within `ask`
- Each citizen has a staggered schedule: `decision-module-turn` starts at random(10), increments each tick, resets to 0 when it reaches `decision-module-interval`
- The decision-module-interval is adaptive: it decreases (more frequent checks) when risk exceeds info-up threshold, increases when risk falls below info-down threshold
- Aggregators update probabilistically (1/3 chance per tick)
- New forecasts from the forecaster are published every advisory period (matched to historical NHC advisory timing, typically every 6 hours)

### Sources
- CHIME.nlogo:231-309 (Go procedure)
- CHIME.nlogo:1872-2035 (Decision-Module)
- CHIME.nlogo:2038-2122 (Process-Forecasts)

### Confidence
- `CODE_VERIFIED` — Process order and scheduling confirmed by reading Go procedure line by line

### Inferences
- **Random execution order**: `INFERRED` — NetLogo's `ask` uses random order by default. This is likely intentional as it prevents artifacts from fixed ordering and is standard ABM practice. No code attempts to override this.

---

## Element 4: Design Concepts

### 4.1 Basic Principles
**Findings:** The model is grounded in the Protective Action Decision Model (PADM) framework (Lindell & Perry, 2012). Citizens collect information from multiple sources, form a mental model of the hurricane threat, assess risk, and make protective action decisions. The model embodies the hypothesis that the modern information environment — with multiple competing information channels — significantly affects evacuation behavior. An ABM approach was chosen because evacuation decisions are inherently individual, heterogeneous, and influenced by social networks and information cascading.

**Sources:** README.md:3 (references Morss et al. 2017, Watts et al. 2019); CHIME.nlogo:22-31 (Go procedure description mentioning mental models, risk assessment, protective action decisions)
**Confidence:** `DOC_STATED` + `INFERRED` from code structure

### 4.2 Emergence
**Findings:** Aggregate evacuation patterns (timing, rates, spatial distribution) emerge from individual-level decision-making. No evacuation rate is imposed top-down; it results from heterogeneous agents independently assessing risk based on their unique thresholds, information sources, social networks, and timing. The percentage evacuated (total and by zone: coastal/inland, within 34-kt/64-kt wind radii) is an emergent outcome.

**Sources:** CHIME.nlogo:2373-2516 (Save-Global-Evac-Statistics — records emergent evacuation categories)
**Confidence:** `CODE_VERIFIED`

### 4.3 Adaptation
**Findings:** Citizens adapt their behavior in two ways:
1. **Information collection frequency**: When risk exceeds `info-up`, agents halve their `decision-module-interval` (check more often). When risk drops below `info-down`, they double it (check less often). Interval is clamped to [1, 32].
2. **Protective action**: When `final-risk-assesment` > `risk-life-threshold`, they evacuate. When between property and life thresholds, they take "other protective action" and increase info collection.

Citizens use the following information for decisions: interpreted forecast (blended from all sources), official evacuation orders, environmental cues (whether they experience tropical storm winds), evacuation zone, and census-derived demographic factors.

**Sources:** CHIME.nlogo:2011-2028 (decision thresholds and feedback loops)
**Confidence:** `CODE_VERIFIED`

### 4.4 Objectives
**Findings:** Citizens do not explicitly optimize a utility function. Instead, they have implicit satisficing behavior: they evacuate when perceived risk exceeds their life-risk threshold. The risk thresholds are heterogeneous and fixed per agent (not adaptive). There is no explicit objective function being maximized.

**Sources:** CHIME.nlogo:1962-1999 (risk calculation and threshold comparison)
**Confidence:** `CODE_VERIFIED`

### 4.5 Learning
**Findings:** Citizens have a `memory` variable that stores their previous interpreted forecast and self-trust. This memory is included in the next round of forecast processing as one input (weighted by self-trust). However, agents do not change their decision rules or thresholds over time — there is no learning in the sense of updating adaptive traits.

**Sources:** CHIME.nlogo:1139,1924 (memory assignment); CHIME.nlogo:2048 (memory used in Process-Forecasts)
**Confidence:** `CODE_VERIFIED`

### 4.6 Prediction
**Findings:** Citizens form a prediction of the hurricane's future path and intensity through `interpreted-forecast`, which is a trust-weighted blend of forecasts from their information sources. This prediction includes the hurricane's projected position, intensity, and forecast uncertainty (cone of uncertainty) at hourly intervals. The citizen uses this prediction to estimate time-to-arrival and spatial proximity, which feed into the risk function.

**Sources:** CHIME.nlogo:2038-2122 (Process-Forecasts); CHIME.nlogo:1913-1930 (using interpreted forecast in Decision-Module)
**Confidence:** `CODE_VERIFIED`

### 4.7 Sensing
**Findings:**
- Citizens sense **environmental cues** directly: they detect if they are within the 34-kt wind radius of the actual hurricane (based on their quadrant relative to the storm). This is a binary (0/1) variable.
- Citizens access forecast information **indirectly** through broadcasters, aggregators, and social network connections — not directly from the forecaster.
- Citizens know their **evacuation zone** (with ~20% error: 80% of the time zone is correctly determined by coastal distance, 20% of the time it's randomly assigned).
- Officials check their county's coastal patches for alerts.
- There is no explicit sensing cost or delay.

**Sources:** CHIME.nlogo:1896-1905 (environmental cues sensing); CHIME.nlogo:1170-1187 (Check-Zone with error)
**Confidence:** `CODE_VERIFIED`

### 4.8 Interaction
**Findings:**
- **Social network**: Scale-free network created via modified preferential attachment. Each citizen has a `my-network-list` with trust factors. Network distance and size are user-configurable (default: distance=50, size=10). Triadic closure is added to increase density.
- **Media interaction**: Citizens subscribe to a random subset of broadcasters and all/some aggregators, each with random trust factors.
- **Information flow chain**: Forecaster → Broadcasters/Aggregators → Citizens → Citizens (via social network). Citizens blend all received forecasts using trust-weighted averaging.
- **Official interaction**: Officials issue county-wide evacuation orders; citizens check the official in their county.
- Interactions are primarily **local** for social networks (bounded by `network-distance`) but **global** for media (broadcasters/aggregators serve all citizens).

**Sources:** CHIME.nlogo:1452-1518 (Social-Network); CHIME.nlogo:2044-2053 (forecast collection)
**Confidence:** `CODE_VERIFIED`

### 4.9 Stochasticity
**Findings:**

| Process | Stochastic Element | Distribution |
|---------|-------------------|--------------|
| Agent placement | Population-weighted random | Lottery based on density |
| Self-trust | Uniform | [0.6, 1.0] |
| Trust-authority | Uniform | [0, 1] |
| Risk-life-threshold | Normal | mean=14, sd=2 |
| Risk-property-threshold | Normal | mean=0.7*risk-life, sd=0.5 |
| Info-up threshold | Normal | mean=0.4*risk-life, sd=0.5 |
| Info-down threshold | Normal | mean=0.1*risk-life, sd=0.5 |
| Decision-module-interval | Normal (rounded) | mean=12, sd=2 |
| Evacuation zone error | Bernoulli | 20% chance of random zone |
| Risk function center | Normal | mean=36, sd=3 (hours before arrival) |
| Risk function spread | Normal | mean=24, sd=12 |
| Risk assessment error | Normal | mean=risk, sd=0.5 |
| Aggregator update | Bernoulli | 1/3 chance per tick |
| Social network trust | Uniform | [0, 1] |
| Forecast filtering | Random shuffle + truncation | Random number of forecasts dropped |

**Sources:** CHIME.nlogo:1095-1166, 1932-1967, 2055-2058
**Confidence:** `CODE_VERIFIED`

### 4.10 Collectives
**Findings:** Citizens form social networks, but these networks do not have collective-level state variables or behaviors. Networks are fixed after initialization (no dynamic formation/dissolution). Counties function as implicit collectives for evacuation order purposes — an official issues orders for an entire county.

**Sources:** CHIME.nlogo:1452-1518 (Social-Network)
**Confidence:** `CODE_VERIFIED`

### 4.11 Observation
**Findings:** The model collects:
1. **Global evacuation statistics**: Percentage evacuated by zone (coastal vs. inland) and wind exposure (within 64-kt, 34-kt, or outside wind radii). Evacuation timing histograms (6-hour bins from 90h before to 18h after landfall).
2. **Individual agent records**: Per-agent: location (lat/lon), county FIPS, coastal/inland classification, self-trust, trust-authority, risk thresholds, evacuation zone, completed actions, census info, evacuated status, evacuation day.
3. **County-level daily evacuations**: Per county per day: cumulative evacuations, total residents, daily new evacuations.
4. **Timestep data**: Per-agent per-timestep risk components (forecast risk, official orders risk, environmental cues risk, final risk assessment).
5. **Interface snapshots**: Optional PNG exports of the NetLogo view each timestep.

**Sources:** CHIME.nlogo:2126-2516 (all output procedures)
**Confidence:** `CODE_VERIFIED`

---

## Element 5: Initialization

### Findings

**Setup sequence:** Setup → Load-GIS → Load-Hurricane → Load-Forecasts → Generate-Storm → Create agents → Social-Network

**Geographic initialization:**
- GIS raster data loaded for selected region (elevation, population density, counties, county seats)
- Patches assigned: elevation, density, county, land/ocean status
- Ocean patches: where elevation data has no-data values
- Coastal patches: land patches adjacent to ocean patches

**Agent initialization:**
- **Citizens** (non-census mode): Placed probabilistically based on population density (lottery mechanism) or randomly on land patches. For Hurricane Michael, citizens are constrained to within 40 grid points of landfall location and within 6 grid points of coast.
- **Citizens** (census mode, Florida only): One citizen per census tract, then additional citizens hatched proportional to tract population / `citizen-to-census-population-ratio`. Census demographic factors assigned probabilistically based on tract-level rates.
- **Forecaster**: 1, placed by population density
- **Officials**: 1 per county, placed at county seat locations from GIS
- **Broadcasters**: User-set count, placed by population density
- **Aggregators**: User-set count, placed by population density
- **Hurricane**: Created when best-track coordinates enter the model domain

**Social network initialization:** Scale-free network with triadic closure; broadcaster and aggregator lists assigned with random trust factors.

**Initial conditions vary between runs** due to stochastic agent placement, attribute assignment, and network formation.

### Sources
- CHIME.nlogo:178-228 (Setup procedure)
- CHIME.nlogo:1082-1361 (agent creation procedures)
- CHIME.nlogo:1452-1518 (Social-Network)

### Confidence
- `CODE_VERIFIED`

### Inferences
- **Random seed**: `CODE_VERIFIED` — No random seed is set in production code. A commented-out `random-seed 99` exists in Load-GIS-HPC (line 2605), suggesting seeds were used during development/testing but not in standard runs. BehaviorSpace experiments use 100 repetitions to capture stochastic variability.

---

## Element 6: Input Data

### Findings

The model uses extensive external input data that drives the simulation:

**1. Hurricane best-track data** (changes over time during simulation):
- Source: NOAA/NHC HURDAT2 format
- Format: Text/CSV files with 6-hourly data: status, lat, lon, intensity (kt), pressure, date, hour, 34-kt and 64-kt wind radii (4 quadrants each)
- Processing: Linearly interpolated to 1-hourly data in Generate-Storm
- Storms available: Harvey, Wilma, Wilma_Ideal, Charley_Real, Charley_Ideal, Charley_Bad, Irma, Michael, Dorian

**2. Forecast advisory data** (changes during simulation, published every ~6 hours):
- Source: NHC forecast advisories (historical) or user-created perfect/imperfect forecasts
- Two formats: (a) Original NHC text advisories parsed by Load-Forecasts, (b) CSV format parsed by Load-Forecasts-New
- Content per advisory: forecast positions, max winds, 34-kt and 64-kt wind radii at multiple lead times
- For Michael: multiple alternative forecast files available (perfect, NHC, linear, fast/slow intensification, different categories)

**3. GIS data** (static, loaded at setup):
- Elevation (SRTM, .asc raster)
- Population density (.asc raster)
- Counties (.asc raster)
- County seats (.shp vector points)
- Three regions: Florida, Gulf, Gulf+SE

**4. Census tract data** (static, Florida only):
- Source: US Census Bureau shapefiles (fltractpoint5.shp)
- ~60 demographic variables per tract including population, households, age groups, language, food stamps, vehicle access, internet access

### Sources
- CHIME.nlogo:401-460 (Load-Hurricane)
- CHIME.nlogo:463-577 (Load-Forecasts)
- CHIME.nlogo:579-803 (Load-Forecasts-New)
- CHIME.nlogo:312-398 (Load-GIS)
- CHIME.nlogo:1265-1361 (Create-Citizen-Agents-From-Census-Tracts)
- STORMS/ directory structure
- REGION/ directory structure

### Confidence
- `CODE_VERIFIED`

### Inferences
- **"Ideal" forecasts**: `CODE_VERIFIED` — Comparing CHARLEY_IDEAL advisory 01 vs CHARLEY_REAL advisory 01: both use NHC format but IDEAL forecasts have positions that closely track the actual best-track path (perfect foresight). Same advisory timing and structure, different position/intensity values. These are synthetic "perfect forecast" scenarios.
- **"Bad" forecasts**: `CODE_VERIFIED` — BAD advisories use systematically offset positions (e.g., 13.42N/69.52W vs real 12.0N/63.0W for the same valid time), representing deliberately inaccurate forecasts. These test the effect of forecast quality on evacuation behavior.
- **Population density raster**: `INFERRED` — Code comments state "calculated by census tract, modified for use w/ GRASS GIS" (CHIME.nlogo:332). Likely derived from census tract population divided by tract area, rasterized to the model grid resolution using GRASS GIS.
- **KNOWLEDGE_GAP_5**: Exact methodology for constructing ideal/bad forecast position offsets not documented.

---

## Element 7: Submodels

### Submodel: Generate-Storm
Translates best-track data to model grid and interpolates from 6-hourly to 1-hourly using linear interpolation for all variables (position, intensity, wind radii). Produces `hurricane-coords-best-track` list: [x, y, intensity, day, hour, 34-kt NE/SE/SW/NW, 64-kt NE/SE/SW/NW].

**Sources:** CHIME.nlogo:942-1063
**Confidence:** `CODE_VERIFIED`

### Submodel: Publish-Forecasts
Forecaster publishes NHC-style advisory matched to current simulation time. Generates a forecast cone with size based on NHC historical cone of uncertainty values. Cone values interpolated from: hours [0,12,24,36,48,72,96,120] → nautical miles [0,26,43,56,74,103,151,198] (Irma/Michael) or [44,77,111,143,208,266,357] (other storms). Reports: [intensity, [x,y], cone_size, [day,hour], [34-kt radii], [64-kt radii]].

**Sources:** CHIME.nlogo:1569-1677
**Confidence:** `CODE_VERIFIED`
**Inference:** The values [0,26,43,56,74,103,151,198] nm at [0,12,24,36,48,72,96,120] hours closely match NHC official 5-year average forecast track errors (circa 2009-2013 per code comment at line 1600). The alternate set [44,77,111,143,208,266,357] appears to be older/larger NHC error values used for pre-Irma storms. **KNOWLEDGE_GAP_6**: Exact NHC error table year/source not cited in code.

### Submodel: Publish-New-Mental-Model
Interpolates the discrete forecast (from Publish-Forecasts) to hourly resolution. Combines current best-track storm position with forecast positions. Linear interpolation between forecast time points for wind speed, position, and cone error.

**Sources:** CHIME.nlogo:1680-1804
**Confidence:** `CODE_VERIFIED`

### Submodel: Coastal-Patches-Alerts
Coastal patches check if the forecasted storm will arrive within `earliest` hours, is within the forecast cone of uncertainty, and intensity exceeds `wind-threshold`. If all conditions met, patch sets `alerts = 1`.

**Parameters:** `earliest` (default 60 hours), `wind-threshold` (default 72 kt)

**Sources:** CHIME.nlogo:1807-1838
**Confidence:** `CODE_VERIFIED`

### Submodel: Issue-Alerts
Officials in counties with alerted coastal patches issue evacuation orders (`orders = 1`). Records `when-issued` (hours before arrival).

**Sources:** CHIME.nlogo:1840-1868
**Confidence:** `CODE_VERIFIED`

### Submodel: Process-Forecasts
Citizens collect forecasts from memory, broadcasters, aggregators, and social network. Each source contributes a [trust-factor, forecast] pair. A random number of forecasts are dropped (shuffle + truncate). Remaining forecasts are blended using trust-weighted averaging across intensity, position (x,y), and cone error for each forecast time.

**Trust-weighted average formula:** For each forecast variable V at time t:
`V_interpreted = sum(trust_i * V_i / sum(trust_j)) for all sources i`

**Sources:** CHIME.nlogo:2038-2122
**Confidence:** `CODE_VERIFIED`
**Inference:** The random forecast dropping is a deliberate simplification modeling bounded rationality — citizens cannot process all available information and selectively attend to a subset. Code comments (lines 2056-2058) note "We may want to rethink which forecasts are kept," suggesting the authors considered this a first approximation. **KNOWLEDGE_GAP_7**: Whether a more sophisticated attention/filtering mechanism was planned or implemented in later work.

### Submodel: Decision-Module
The core decision-making process. Calculates risk using a Gaussian curve:

**Risk function:**
```
risk = (1 / (height * sqrt(2*pi))) * exp(-((x - center)^2) / (2 * sd^2))
```

Where:
- `x` = hours until storm reaches closest point to citizen
- `center` ~ Normal(36, 3) — peak risk perception ~36 hours before arrival
- `sd` ~ Normal(24, 12) — spread of the risk function
- `height` = sqrt(dist_trk + 0.003 * zone + 0.000525 * intensity)
- `dist_trk` = ((scale * 0.5 * distance) * (0.0011 / error_bars)) + 0.0011
- `zone`: 0 if zone "A" (coastal), 1 otherwise
- `intensity`: 0 if >= 95 kt (Cat 3+), 1 otherwise (binary — high intensity reduces height, increasing risk)

**Final risk assembly:**
```
final_risk = forc_weight * (1.1 * random-normal(risk, 0.5))
           + evac_weight * (trust_authority * 6 * official_orders * zone_factor)
           + envc_weight * (3 * environmental_cues)
```

Where `zone_factor` = 1 if zone "A", 0.4 otherwise.

**Census modifications** (additive/subtractive percentage adjustments):
- kids-under-18: +`under-18-assessment-increase` * final_risk
- adults-over-65: -`over-65-assessment-decrease` * final_risk
- limited-english: -`limited-english-assessment-decrease` * final_risk
- food-stamps: -`foodstamps-assessment-decrease` * final_risk
- no-vehicle: -`no-vehicle-assessment-modification` * final_risk
- no-internet: -`no-internet-assessment-modification` * final_risk

**Decision thresholds:**
- `final_risk > risk-life-threshold` → **Evacuate** (color orange, record action)
- `risk-property-threshold < final_risk < risk-life-threshold` → **Other protective action** (halve decision interval)
- `info-up < final_risk < risk-property-threshold` → **Increase info collection** (halve decision interval)
- `info-down < final_risk < info-up` → No change
- `final_risk < info-down` → **Decrease info collection** (double decision interval, max 32)

**Sources:** CHIME.nlogo:1872-2035
**Confidence:** `CODE_VERIFIED`
**Inferences and Knowledge Gaps:**
1. **Gaussian risk function**: `INFERRED` — Models the empirical observation that risk perception peaks 1-2 days before landfall then drops (either because the storm has passed or people have already acted). A Gaussian captures this rise-peak-decline pattern naturally.
2. **Center at 36 hours**: `INFERRED` — Corresponds to roughly 1.5 days before arrival, when evacuation orders are typically issued and media coverage intensifies. Aligns with NHC's typical 36-48 hour hurricane watch timing.
3. **1.1 multiplier**: `INFERRED` — A small upward bias (10%) on the raw risk, likely a calibration adjustment to increase overall evacuation rates to match observed data.
4. **Binary intensity (>=95 kt)**: `INFERRED` — Code comment references Morss & Hayden (2010) Fig. 5 and Zhang et al. (2007) Fig. 5, suggesting empirical evidence that Cat 3+ hurricanes produce a qualitative shift in risk perception. The binary treatment is acknowledged as a simplification (comment: "We may want to rethink this").
5. **Calibration constants (0.003, 0.000525, 0.0011, 6, 3)**: `INFERRED` — These scale the contributions of zone, intensity, official orders, and environmental cues into the Gaussian height parameter. They were likely tuned through sensitivity analysis to produce evacuation rates in the 15-20% range observed for real hurricanes.
- **KNOWLEDGE_GAP_8**: Formal calibration methodology and sensitivity analysis results not available in repository.

### Submodel: Social-Network
Creates a scale-free network using modified preferential attachment:
1. Each citizen searches within `network-distance` for partners
2. Partners selected probabilistically based on (network-list length)^network-size
3. Triadic closure step: each citizen's network member shares one of its connections
4. Trust factors assigned randomly [0,1] and used to sort network connections

**Parameters:** `network-distance` (default 50), `network-size` (default 10)

**Sources:** CHIME.nlogo:1452-1518
**Confidence:** `CODE_VERIFIED`

### Submodel: Check-Zone
Determines citizen's perceived evacuation zone based on distance from nearest coastal patch:
- Zone A: ≤ 1.5 grid points from coast
- Zone B: 1.5–3 grid points
- Zone C: 3–5 grid points
- No zone: > 5 grid points
- 20% of the time, zone is randomly assigned (modeling imperfect zone knowledge)

**Sources:** CHIME.nlogo:1170-1187
**Confidence:** `CODE_VERIFIED`

---

## Interface Parameters Summary

| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| which-storm? | Chooser | CHARLEY_REAL | Storm scenario selection |
| #citizen-agents | 0–5000 | 2000 | Number of citizen agents |
| #broadcasters | 0–10 | 10 | Number of broadcasters |
| #net-aggregators | 0–10 | 10 | Number of aggregators |
| distribute-population | Switch | Off | Place citizens by population density |
| use-census-data | Switch | Off | Use census tract data (Florida only) |
| earliest | 12–200 | 60 | Hours before arrival for official alerts |
| latest | 0–12 | 12 | (Appears unused in current code) |
| wind-threshold | 70–130 | 72 | Wind speed (kt) threshold for alerts |
| forc-weight | 0–2 | 1.0 | Weight of forecast info in risk |
| evac-weight | 0–4 | 1.0 | Weight of evacuation orders in risk |
| envc-weight | 0–6 | 1.0 | Weight of environmental cues in risk |
| network-distance | 0–50 | 50 | Max distance for social network |
| network-size | 1–10 | 10 | Power law exponent for network |
| citizen-to-census-population-ratio | 0–10000 | 5000 | Census pop per agent |
| Census factor switches | On/Off | Off | Toggle each demographic factor |
| Census modification sliders | Various | Various | Magnitude of demographic risk modifications |
| display-dem? | Switch | — | Display elevation raster |
