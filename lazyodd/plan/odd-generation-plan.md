# ODD Generation Plan for CHIME ABM

## Instructions for the Drafting Agent

You are an ODD technical writer. Execute this plan to produce a complete ODD+2 protocol document for the CHIME ABM (Communicating Hazard Information in the Modern Environment) model.

Your task: read all source materials listed below, then produce two output files by following the section-by-section instructions in this plan. Every piece of knowledge you need is embedded in this document. Do NOT attempt independent code analysis unless explicitly instructed.

**Writing approach:** Describe what the program does with formal precision. Never paraphrase technical concepts into simpler language. Every claim must cite its source. Every parameter must include value, units, and range.

## Context

- **Model name:** CHIME ABM (Communicating Hazard Information in the Modern Environment)
- **Authors:** Watts J., Morss R.E., Barton C.M., Demuth J.L. (and collaborators)
- **Platform:** NetLogo 6.2.1
- **Model complexity:** Moderate (6 entity types, ~10 submodels)
- **ODD format:** ODD+2 with extensions (rich design rationale needed given theoretical grounding in PADM)
- **Input quality:** Code (11,831 lines) + minimal documentation (README, How-To guide)
- **Research mode:** Autonomous reverse-engineering from code; no modeler interview conducted

## Source Materials

Read these files before drafting:

| File | Contains |
|------|----------|
| `CHIME.nlogo` | Full model code — 11,831 lines. Primary authoritative source for all behavioral claims. |
| `README.md` | Model overview, publication references (Morss et al. 2017, Watts et al. 2019) |
| `How To Use CHIME.md` | User guide with example scenarios and parameter descriptions |
| `STORMS/` directory | 9 storm scenario subdirectories with best-track and forecast advisory data |
| `REGION/` directory | 3 geographic regions (Florida, Gulf, Gulf+SE) with GIS raster/vector data |

**Note:** The publications referenced (Morss et al. 2017, Watts et al. 2019) are NOT available in the repository. Do not fabricate content from these papers. Where the ODD would benefit from paper-sourced information, note this as a knowledge gap.

## Terminology

| Term | Use This | NOT This | Definition |
|------|----------|----------|------------|
| citizen-agent | citizen-agent | citizen, household, person, resident | A NetLogo turtle representing a household that collects information and makes evacuation decisions |
| forecaster | forecaster | NHC, weather service, meteorologist | Single agent that publishes NHC-style forecast advisories on the historical advisory schedule |
| official | official | emergency manager, government | Agent representing a county emergency manager who issues evacuation orders |
| broadcaster | broadcaster | media, news, TV station | Agent representing traditional media that translates forecaster output |
| aggregator | aggregator | social media, online source, website | Agent representing social media / online information aggregators |
| hurricane | hurricane | storm, cyclone | Visualization turtle tracking the storm's best-track position (not behavioral) |
| interpreted-forecast | interpreted-forecast | mental model, belief, perception | Trust-weighted blend of forecasts from all of a citizen-agent's information sources |
| risk-life-threshold | risk-life-threshold | evacuation threshold, danger threshold | Per-agent threshold above which the agent evacuates |
| decision-module-interval | decision-module-interval | check frequency, update interval | Hours between a citizen-agent's decision module executions |
| best-track data | best-track data | actual track, real track, observed track | HURDAT2-format historical hurricane position/intensity data interpolated to hourly |
| forecast advisory | forecast advisory | forecast, prediction, outlook | NHC-format advisory with forecast positions, intensity, and wind radii at multiple lead times |
| cone of uncertainty | cone of uncertainty | error cone, forecast cone, uncertainty range | Forecast track error bounds based on NHC historical average errors |
| environmental cues | environmental cues | weather observations, felt conditions | Binary (0/1) — whether a citizen-agent is within the 34-kt wind radius of the actual hurricane |
| evacuation zone | evacuation zone | risk zone, danger zone | Perceived zone (A/B/C/none) based on coastal distance, with 20% error rate |
| PADM | Protective Action Decision Model (PADM) | decision framework, decision theory | Lindell & Perry (2012) framework grounding the model's decision-making structure |
| BehaviorSpace | BehaviorSpace | parameter sweep, experiment, batch run | NetLogo's built-in experiment tool for systematic parameter exploration |

## Knowledge Base

---

### Element 1: Purpose and Patterns

**What to write:** A section explaining the model's purpose, the questions it addresses, and the patterns it is designed to reproduce or explore.

**Findings:**

The CHIME ABM was designed as a **virtual laboratory** to investigate the dynamics of hazardous weather communication and decision making during hurricane threats. Specifically, it examines how information flow through the modern information environment affects household evacuation decisions. [source: README.md:3; How To Use CHIME.md:1-3]

The higher-level purpose is **explanation and theoretical exploration**: understanding how the modern information environment — with multiple, sometimes conflicting information sources — shapes household-level hurricane evacuation decisions. [source: README.md:3] {DOC_STATED}

The model is grounded in the **Protective Action Decision Model (PADM)** framework (Lindell & Perry, 2012). [source: README.md:3] {DOC_STATED}

Users can modify parameters to explore the relative effects of:
- Weighting of different information types (forecast, evacuation orders, environmental cues) via `forc-weight`, `evac-weight`, `envc-weight`
- Inclusion of census demographic information via toggle switches
- Different hurricanes affecting different regions (9 storm scenarios x 3 regions)
- Perfect versus imperfect forecasts (IDEAL vs. REAL vs. BAD forecast files)
[source: How To Use CHIME.md:1-3; CHIME.nlogo BehaviorSpace experiments] {DOC_STATED}

**Patterns (from output procedures):**
The model does not produce a single verifiable outcome. Output consists of timing and location of household evacuations, evaluated against: [source: CHIME.nlogo:2373-2516, Save-Global-Evac-Statistics] {CODE_VERIFIED}

1. **Evacuation rates by exposure zone**: Percentage evacuated among coastal/inland residents within 64-kt wind radius, within 34-kt wind radius, and outside wind radii (6 categories total)
2. **Evacuation timing relative to landfall**: Histogram in 6-hour bins from 90 hours before to 18 hours after landfall
3. **Official order timing**: When officials issued orders relative to landfall
4. **Sensitivity to information weights**: BehaviorSpace experiments systematically sweep `forc-weight`, `evac-weight`, `envc-weight`

**Instructions:** Write this section in two subsections: "Purpose" and "Patterns." State the purpose clearly and concisely. For patterns, describe each output metric and note that quantitative target values from historical records are not encoded in the code.

**Knowledge gaps to acknowledge:**
- KG-1: Quantitative target evacuation rates/timing from historical records are not specified in the code {UNVERIFIABLE — would require access to Watts et al. 2019}
- KG-2: Patterns the model failed to reproduce cannot be determined from the repository {UNVERIFIABLE}
- KG-3: Specific hypotheses from Watts et al. 2019 are not available {UNVERIFIABLE — paper not in repository}

---

### Element 2: Entities, State Variables, and Scales

**What to write:** A comprehensive section listing all entity types, their state variables (with types, ranges, initialization values, and descriptions), and the spatial and temporal scales.

**Findings — Entity types:**

| Entity | NetLogo Breed | Count | Description |
|--------|---------------|-------|-------------|
| citizen-agent | `citizen-agents` | User-set (0-5000, default 2000) or census-derived | Households making evacuation decisions |
| forecaster | `forecasters` | 1 | Publishes NHC-style forecast advisories |
| official | `officials` | 1 per county (from GIS county-seats shapefile) | County emergency managers issuing evacuation orders |
| broadcaster | `broadcasters` | User-set (0-10, default 10) | Traditional media translating forecasts for citizens |
| aggregator | `aggregators` | User-set (0-10, default 10) | Social media / online information aggregators |
| hurricane | `hurricanes` | 0-1 | Visualization of hurricane position; not behavioral |

Utility breeds (non-behavioral, visualization/data only): `forcstxs` (forecast cone visualization), `drawers` (storm track drawing), `tracts` (census tract points).
[source: CHIME.nlogo:46-171] {CODE_VERIFIED}

**Findings — Citizen-agent state variables:**

| Variable | Type | Initialization | Description |
|----------|------|---------------|-------------|
| `evac-zone` | String ("A","B","C","") | Check-Zone procedure | Perceived evacuation zone based on coastal distance |
| `self-trust` | Float [0.6, 1.0] | 0.6 + random-float 0.4 | Confidence in own forecast interpretation |
| `trust-authority` | Float [0, 1] | random-float 1 | Trust in officials' evacuation orders |
| `risk-life-threshold` | Float | Normal(mean=14, sd=2) | Threshold for evacuating (risk to life) |
| `risk-property-threshold` | Float | Normal(mean=0.7*risk-life-threshold, sd=0.5) | Threshold for other protective action |
| `info-up` | Float | Normal(mean=0.4*risk-life-threshold, sd=0.5) | Threshold to increase info collection frequency |
| `info-down` | Float | Normal(mean=0.1*risk-life-threshold, sd=0.5) | Threshold to decrease info collection frequency |
| `decision-module-interval` | Integer | round(Normal(mean=12, sd=2)) | Hours between decision module runs |
| `decision-module-turn` | Integer | random 10 | Counter for scheduling decision module |
| `my-network-list` | List of [agent, trust-factor] | Scale-free network algorithm | Social network connections with trust weights |
| `broadcaster-list` | List of [agent, trust-factor] | Random subset of broadcasters | Media connections with trust weights |
| `aggregator-list` | List of [agent, trust-factor] | Random subset of aggregators | Online info source connections |
| `interpreted-forecast` | List | [] | Agent's blended interpretation of the hurricane forecast |
| `memory` | List | [self-trust, interpreted-forecast] | Previous forecast interpretation |
| `environmental-cues` | 0 or 1 | 0 | Whether agent is experiencing tropical storm winds |
| `risk-estimate` | List | [0] | History of risk calculations |
| `completed` | List | [] | History of decisions made (e.g., ["evacuate" clock counter]) |
| `evacuated?` | Boolean | false | Whether agent has evacuated |
| `evac-day` | Number | 0 | Day of evacuation |
| `final-risk-assesment` | Float | — | Current total risk value (note: spelling matches code) |
| `risk-forecast` | Float | — | Forecast component of risk (weighted) |
| `risk-official-orders` | Float | — | Evacuation order component (weighted) |
| `risk-environmental-cues` | Float | — | Environmental cue component (weighted) |
| `risk-packet` | List | [risk, env-cues, 0, 0] | Summary risk info for output |
| Census variables (when use-census-data = On) | Various | From GIS census tracts | `kids-under-18?`, `adults-over-65?`, `limited-english?`, `food-stamps?`, `no-vehicle?`, `no-internet?`, `county-fips`, `census-tract-number`, `tract-information`, `my-tract-population`, `my-tract-household` |

[source: CHIME.nlogo:46-171, 1082-1167] {CODE_VERIFIED}

**Findings — Other entity state variables:**

| Entity | Variables |
|--------|-----------|
| official | `orders` (0 or 1), `distance-to-track`, `county-id`, `when-issued` |
| forecaster | `current-forecast` |
| broadcaster | `broadcast` (current interpreted forecast) |
| aggregator | `info` (current interpreted forecast) |

[source: CHIME.nlogo:46-171] {CODE_VERIFIED}

**Findings — Patch (environment) state variables:**
`density` (population density from GIS), `elev` (elevation from GIS), `county` (county ID from GIS), `alerts` (0/1 evacuation alert), `land?` (boolean)
[source: CHIME.nlogo:46-171] {CODE_VERIFIED}

**Findings — Key global variables:**
`clock` [day, hour], `best-track-data`, `hurricane-coords-best-track`, `forecast-matrix`, `scale` (nautical miles per grid cell), `grid-cell-size` (degrees per grid cell), `re0-0` (coordinate system origin), `land-patches`, `ocean-patches`, `coastal-patches`
[source: CHIME.nlogo:46-171] {CODE_VERIFIED}

**Findings — Spatial scale:**
- Grid: GIS-derived raster, world extent -100 to 100 (x) by -75 to 75 (y) patches [source: CHIME.nlogo:3005-3030] {CODE_VERIFIED}
- Grid cell size: variable, derived from GIS envelope (~0.1 degrees, approximately 6 nautical miles) {INFERRED}
- Three regions available: Florida, Gulf states, Gulf+Southeast
- Coordinate system: projected from lat/lon to NetLogo grid via GIS extension

**Findings — Temporal scale:**
- 1 tick = 1 hour [source: CHIME.nlogo:231-309] {CODE_VERIFIED}
- Simulation runs until hurricane passes through domain (typically 100-140 ticks depending on storm)
- Decision module runs every `decision-module-interval` ticks per agent (default ~12 hours)

**Instructions:** Present entities in a table. Then present each entity's state variables in sub-tables. Use the exact variable names from the code. Describe spatial and temporal scales precisely with units.

---

### Element 3: Process Overview and Scheduling

**What to write:** A numbered process list showing the exact execution order per tick, with scheduling notes.

**Findings — Per-tick execution order:**

Each time step (1 hour), the following processes execute in this exact order:

1. **Move-Hurricane** — Hurricane visualization advances to next position from interpolated best-track data
2. **Publish-Forecasts** (asked of forecasters) — Forecaster publishes current NHC-style advisory matched to simulation time
3. **Publish-New-Mental-Model** (reporter) — Interpolates forecast to hourly data, creating an hourly forecast track with intensity, location, and cone of uncertainty
4. **Coastal-Patches-Alerts** — Coastal patches evaluate if storm is within distance/intensity thresholds to trigger alerts
5. **Issue-Alerts** (asked of officials whose county has alerted patches) — Officials issue evacuation orders
6. **Broadcasters update** — All broadcasters receive the interpolated forecast
7. **Aggregators update** — Each aggregator has a 1/3 chance of updating its information each tick
8. **Citizen-agent decision cycle** (asked of all citizen-agents):
   - If NOT evacuated: check if it's their turn (`decision-module-turn` < `decision-module-interval`); if yes, run **Decision-Module**
   - If ALREADY evacuated: still collect and process information (Process-Forecasts) so their network connections get updated info
9. **Color updates** — Visual updates for agent display
10. **Clock update** — Update simulation clock from best-track data
11. **Data saving** — Conditional saving of timestep data, images, county evacuation data
12. **Stop check** — If hurricane has passed, save final output and stop

[source: CHIME.nlogo:231-309, Go procedure] {CODE_VERIFIED}

**Scheduling notes:**
- All processes are **synchronous** within each tick
- Citizen-agents execute in NetLogo's default random order within `ask` {INFERRED — standard NetLogo behavior, no override in code}
- Each citizen-agent has a staggered schedule: `decision-module-turn` starts at random(10), increments each tick, resets to 0 when it reaches `decision-module-interval`
- The `decision-module-interval` is **adaptive**: it halves (more frequent checks) when risk exceeds `info-up` threshold, doubles (less frequent) when risk falls below `info-down` threshold. Clamped to [1, 32].
- Aggregators update probabilistically (1/3 chance per tick)
- New forecasts from the forecaster are published every advisory period (matched to historical NHC advisory timing, typically every 6 hours)

**Instructions:** Present as a numbered list with a scheduling diagram or table. Include a subsection on the adaptive decision interval mechanism. Make clear that execution is synchronous within ticks and random-order within `ask` blocks.

---

### Element 4: Design Concepts

**What to write:** Address each of the 11 design concepts with specific findings. Each concept should be a subsection.

#### 4.1 Basic Principles

**Findings:** The model is grounded in the Protective Action Decision Model (PADM) framework (Lindell & Perry, 2012). Citizens collect information from multiple sources, form a mental model (interpreted-forecast) of the hurricane threat, assess risk, and make protective action decisions. The model embodies the hypothesis that the modern information environment — with multiple competing information channels — significantly affects evacuation behavior. An ABM approach was chosen because evacuation decisions are inherently individual, heterogeneous, and influenced by social networks and information cascading.
[source: README.md:3; CHIME.nlogo:22-31] {DOC_STATED} + {INFERRED from code structure}

**Instructions:** Describe the PADM framework briefly. Explain why an ABM was chosen. Note the central hypothesis about the modern information environment.

#### 4.2 Emergence

**Findings:** Aggregate evacuation patterns (timing, rates, spatial distribution) emerge from individual-level decision-making. No evacuation rate is imposed top-down; it results from heterogeneous agents independently assessing risk based on their unique thresholds, information sources, social networks, and timing. The percentage evacuated (total and by zone) is an emergent outcome.
[source: CHIME.nlogo:2373-2516, Save-Global-Evac-Statistics] {CODE_VERIFIED}

**Instructions:** State clearly that aggregate evacuation behavior is emergent, not imposed. List the specific emergent patterns.

#### 4.3 Adaptation

**Findings:** Citizens adapt their behavior in two ways:
1. **Information collection frequency**: When risk exceeds `info-up`, agents halve their `decision-module-interval` (check more often). When risk drops below `info-down`, they double it (check less often). Interval is clamped to [1, 32].
2. **Protective action**: When `final-risk-assesment` > `risk-life-threshold`, they evacuate. When between property and life thresholds, they take "other protective action" and increase info collection.

Citizens use: interpreted-forecast, official evacuation orders, environmental cues, evacuation zone, and census-derived demographic factors.
[source: CHIME.nlogo:2011-2028] {CODE_VERIFIED}

**Instructions:** Describe both adaptive mechanisms precisely. Include the clamping bounds.

#### 4.4 Objectives

**Findings:** Citizens do not explicitly optimize a utility function. They have implicit satisficing behavior: they evacuate when perceived risk exceeds their `risk-life-threshold`. Risk thresholds are heterogeneous and fixed per agent (not adaptive). There is no explicit objective function being maximized.
[source: CHIME.nlogo:1962-1999] {CODE_VERIFIED}

**Instructions:** State that there is no explicit optimization. Describe the satisficing threshold mechanism.

#### 4.5 Learning

**Findings:** Citizens have a `memory` variable that stores their previous interpreted-forecast and self-trust. This memory is included in the next round of forecast processing as one input (weighted by self-trust). However, agents do not change their decision rules or thresholds over time — there is no learning in the sense of updating adaptive traits. Memory serves as inertia in forecast interpretation, not as learning.
[source: CHIME.nlogo:1139, 1924 (memory assignment); CHIME.nlogo:2048 (memory used in Process-Forecasts)] {CODE_VERIFIED}

**Instructions:** Distinguish between memory-as-inertia and learning-as-adaptation. Be precise that thresholds do not change.

#### 4.6 Prediction

**Findings:** Citizens form a prediction of the hurricane's future path and intensity through `interpreted-forecast`, which is a trust-weighted blend of forecasts from their information sources. This includes projected position, intensity, and cone of uncertainty at hourly intervals. The citizen uses this prediction to estimate time-to-arrival and spatial proximity, which feed into the Gaussian risk function.
[source: CHIME.nlogo:2038-2122, Process-Forecasts; CHIME.nlogo:1913-1930] {CODE_VERIFIED}

**Instructions:** Describe how predictions are formed and how they feed into decision-making.

#### 4.7 Sensing

**Findings:**
- Citizens sense **environmental cues** directly: they detect if they are within the 34-kt wind radius of the actual hurricane (based on their quadrant relative to the storm). This is binary (0/1). [source: CHIME.nlogo:1896-1905] {CODE_VERIFIED}
- Citizens access forecast information **indirectly** through broadcasters, aggregators, and social network connections — not directly from the forecaster. [source: CHIME.nlogo:2044-2053] {CODE_VERIFIED}
- Citizens know their **evacuation zone** with ~20% error: 80% of the time zone is correctly determined by coastal distance, 20% randomly assigned. [source: CHIME.nlogo:1170-1187] {CODE_VERIFIED}
- Officials check their county's coastal patches for alerts. [source: CHIME.nlogo:1840-1868] {CODE_VERIFIED}
- There is no explicit sensing cost or delay.

**Instructions:** List each sensing mechanism. Distinguish direct sensing (environmental cues) from indirect (forecasts via intermediaries). Note the zone error rate.

#### 4.8 Interaction

**Findings:**
- **Social network**: Scale-free network via modified preferential attachment. Each citizen has `my-network-list` with trust factors. Network distance (default 50) and size (default 10) are user-configurable. Triadic closure adds density. [source: CHIME.nlogo:1452-1518] {CODE_VERIFIED}
- **Media interaction**: Citizens subscribe to random subset of broadcasters and aggregators, each with random trust factors [0, 1]. [source: CHIME.nlogo:1082-1167] {CODE_VERIFIED}
- **Information flow chain**: Forecaster -> Broadcasters/Aggregators -> Citizens -> Citizens (via social network). Citizens blend all received forecasts using trust-weighted averaging. [source: CHIME.nlogo:2044-2053] {CODE_VERIFIED}
- **Official interaction**: Officials issue county-wide evacuation orders; citizens check the official in their county. [source: CHIME.nlogo:1840-1868] {CODE_VERIFIED}
- Social network interactions are **local** (bounded by `network-distance`); media interactions are **global** (broadcasters/aggregators serve all subscribing citizens).

**Instructions:** Describe the information flow chain clearly — this is central to the model. Include a diagram or structured list showing the flow: Forecaster -> Media -> Citizens -> Citizens.

#### 4.9 Stochasticity

**Findings:**

| Process | Stochastic Element | Distribution |
|---------|-------------------|--------------|
| Agent placement | Population-weighted random | Lottery based on patch density |
| self-trust | Uniform | [0.6, 1.0] |
| trust-authority | Uniform | [0, 1] |
| risk-life-threshold | Normal | mean=14, sd=2 |
| risk-property-threshold | Normal | mean=0.7*risk-life-threshold, sd=0.5 |
| info-up threshold | Normal | mean=0.4*risk-life-threshold, sd=0.5 |
| info-down threshold | Normal | mean=0.1*risk-life-threshold, sd=0.5 |
| decision-module-interval | Normal (rounded) | mean=12, sd=2 |
| Evacuation zone error | Bernoulli | 20% chance of random zone |
| Risk function center | Normal | mean=36, sd=3 (hours before arrival) |
| Risk function spread | Normal | mean=24, sd=12 |
| Risk assessment error | Normal | mean=risk, sd=0.5 |
| Aggregator update | Bernoulli | 1/3 chance per tick |
| Social network trust | Uniform | [0, 1] |
| Forecast filtering | Random shuffle + truncation | Random number of forecasts dropped |

[source: CHIME.nlogo:1095-1166, 1932-1967, 2055-2058] {CODE_VERIFIED}

**Instructions:** Present as a table. Note that no random seed is set in production code (a commented-out `random-seed 99` exists in Load-GIS-HPC at line 2605). BehaviorSpace experiments use 100 repetitions to capture stochastic variability.

#### 4.10 Collectives

**Findings:** Citizens form social networks, but these do not have collective-level state variables or behaviors. Networks are fixed after initialization (no dynamic formation/dissolution). Counties function as implicit collectives for evacuation order purposes — an official issues orders for an entire county.
[source: CHIME.nlogo:1452-1518] {CODE_VERIFIED}

**Instructions:** Note the absence of formal collectives. Mention counties as an implicit grouping mechanism for official orders.

#### 4.11 Observation

**Findings:** The model collects:
1. **Global evacuation statistics**: Percentage evacuated by zone (coastal vs. inland) and wind exposure (within 64-kt, 34-kt, or outside wind radii). Evacuation timing histograms (6-hour bins from 90h before to 18h after landfall).
2. **Individual agent records**: Per-agent: location (lat/lon), county FIPS, coastal/inland classification, self-trust, trust-authority, risk thresholds, evacuation zone, completed actions, census info, evacuated status, evacuation day.
3. **County-level daily evacuations**: Per county per day: cumulative evacuations, total residents, daily new evacuations.
4. **Timestep data**: Per-agent per-timestep risk components (forecast risk, official orders risk, environmental cues risk, final risk assessment).
5. **Interface snapshots**: Optional PNG exports of the NetLogo view each timestep.
[source: CHIME.nlogo:2126-2516] {CODE_VERIFIED}

**Instructions:** List all output types. Describe the structure of each output file/metric. Note that these are written to CSV files.

---

### Element 5: Initialization

**What to write:** A section describing the complete setup sequence, how each entity type is initialized, and what varies between runs.

**Findings — Setup sequence:**
`Setup` -> `Load-GIS` -> `Load-Hurricane` -> `Load-Forecasts` -> `Generate-Storm` -> Create agents -> `Social-Network`
[source: CHIME.nlogo:178-228] {CODE_VERIFIED}

**Geographic initialization:**
- GIS raster data loaded for selected region (elevation, population density, counties, county seats)
- Patches assigned: elevation, density, county, land/ocean status
- Ocean patches: where elevation data has no-data values
- Coastal patches: land patches adjacent to ocean patches
[source: CHIME.nlogo:312-398] {CODE_VERIFIED}

**Agent initialization:**
- **Citizens (non-census mode):** Placed probabilistically based on population density (lottery mechanism — probability proportional to patch density) or randomly on land patches. For Hurricane Michael, citizens are constrained to within 40 grid points of landfall location and within 6 grid points of coast. [source: CHIME.nlogo:1082-1167] {CODE_VERIFIED}
- **Citizens (census mode, Florida only):** One citizen-agent per census tract, then additional citizens hatched proportional to tract population / `citizen-to-census-population-ratio`. Census demographic factors assigned probabilistically based on tract-level rates (e.g., if tract has 15% limited-english rate, each citizen has 15% chance of `limited-english? = true`). [source: CHIME.nlogo:1265-1361] {CODE_VERIFIED}
- **Forecaster:** 1, placed by population density [source: CHIME.nlogo:1189-1262] {CODE_VERIFIED}
- **Officials:** 1 per county, placed at county seat locations from GIS shapefile [source: CHIME.nlogo:1189-1262] {CODE_VERIFIED}
- **Broadcasters:** User-set count (default 10), placed by population density [source: CHIME.nlogo:1189-1262] {CODE_VERIFIED}
- **Aggregators:** User-set count (default 10), placed by population density [source: CHIME.nlogo:1189-1262] {CODE_VERIFIED}
- **Hurricane:** Created when best-track coordinates enter the model domain [source: CHIME.nlogo:231-309] {CODE_VERIFIED}

**Social network initialization:** Scale-free network with triadic closure; broadcaster-list and aggregator-list assigned with random trust factors [0, 1].
[source: CHIME.nlogo:1452-1518] {CODE_VERIFIED}

**Variation between runs:** Initial conditions vary due to stochastic agent placement, attribute assignment (all normally/uniformly distributed variables), and network formation. No random seed is set in production runs.

**Instructions:** Describe the setup sequence step by step. Include a subsection on how citizens are placed (density-weighted lottery). Describe the census mode as an alternative initialization path. Note all sources of between-run variation.

---

### Element 6: Input Data

**What to write:** A section describing all external data that drives the simulation during a run, plus static data loaded at setup.

**Findings:**

**1. Hurricane best-track data** (dynamic — drives storm position each tick):
- Source: NOAA/NHC HURDAT2 format
- Format: Text/CSV files with 6-hourly data: status, lat, lon, intensity (kt), pressure, date, hour, 34-kt and 64-kt wind radii (4 quadrants each)
- Processing: Linearly interpolated to 1-hourly data in Generate-Storm
- Storms available: Harvey, Wilma, Wilma_Ideal, Charley_Real, Charley_Ideal, Charley_Bad, Irma, Michael, Dorian
[source: CHIME.nlogo:401-460; STORMS/ directory] {CODE_VERIFIED}

**2. Forecast advisory data** (dynamic — published every ~6 hours during simulation):
- Source: NHC forecast advisories (historical) or user-created perfect/imperfect forecasts
- Two formats: (a) Original NHC text advisories parsed by `Load-Forecasts`, (b) CSV format parsed by `Load-Forecasts-New`
- Content per advisory: forecast positions, max winds, 34-kt and 64-kt wind radii at multiple lead times
- For Michael: multiple alternative forecast files (perfect, NHC, linear, fast/slow intensification, different categories)
- IDEAL forecasts = positions closely track actual best-track (perfect foresight) {CODE_VERIFIED — verified by comparing advisory files}
- BAD forecasts = systematically offset positions (deliberately inaccurate) {CODE_VERIFIED}
[source: CHIME.nlogo:463-803; STORMS/ directory] {CODE_VERIFIED}

**3. GIS data** (static — loaded at setup):
- Elevation: SRTM, .asc raster
- Population density: .asc raster (derived from census tract population, rasterized via GRASS GIS) {INFERRED from code comment at line 332}
- Counties: .asc raster
- County seats: .shp vector points
- Three regions: Florida, Gulf, Gulf+Southeast
[source: CHIME.nlogo:312-398; REGION/ directory] {CODE_VERIFIED}

**4. Census tract data** (static — Florida only):
- Source: US Census Bureau shapefiles (fltractpoint5.shp)
- ~60 demographic variables per tract including population, households, age groups, language, food stamps, vehicle access, internet access
[source: CHIME.nlogo:1265-1361] {CODE_VERIFIED}

**Instructions:** Describe each data source in a subsection. Distinguish static (loaded once) from dynamic (changes during simulation). Include format details. Describe the IDEAL/BAD forecast variants and their purpose. Note KG-5 (exact methodology for constructing ideal/bad forecast offsets not documented).

---

### Element 7: Submodels

**What to write:** A detailed subsection for each submodel with complete equations, pseudocode, parameters, and behavior description. This is the longest and most critical section. Every equation and parameter must be specified precisely enough for reimplementation.

#### Submodel: Generate-Storm

**Purpose:** Translates best-track data from geographic coordinates to model grid coordinates and interpolates from 6-hourly to 1-hourly resolution.

**Algorithm:**
1. Read 6-hourly best-track records (lat, lon, intensity, wind radii)
2. Convert lat/lon to NetLogo grid coordinates using GIS projection
3. For each pair of consecutive 6-hourly records, linearly interpolate all variables to produce 6 intermediate hourly records
4. Store as `hurricane-coords-best-track` list: [x, y, intensity_kt, day, hour, 34kt_NE, 34kt_SE, 34kt_SW, 34kt_NW, 64kt_NE, 64kt_SE, 64kt_SW, 64kt_NW]

**Parameters:** None user-configurable; driven entirely by input data files.

[source: CHIME.nlogo:942-1063] {CODE_VERIFIED}

**Instructions:** Describe the interpolation algorithm precisely. Note that all variables (position, intensity, wind radii) are interpolated linearly.

#### Submodel: Publish-Forecasts

**Purpose:** Forecaster agent publishes NHC-style advisory matched to current simulation time. Generates a forecast cone with size based on NHC historical track error.

**Algorithm:**
1. Match current simulation time to the appropriate forecast advisory
2. For each forecast time point, extract: intensity, position (x,y), 34-kt and 64-kt wind radii
3. Calculate cone of uncertainty size by interpolating from NHC historical error table

**Cone of uncertainty values:**

For Irma/Michael storms:
| Lead time (hours) | 0 | 12 | 24 | 36 | 48 | 72 | 96 | 120 |
|---|---|---|---|---|---|---|---|---|
| Cone radius (nm) | 0 | 26 | 43 | 56 | 74 | 103 | 151 | 198 |

For other storms (older/larger errors):
| Lead time (hours) | 12 | 24 | 36 | 48 | 72 | 96 | 120 |
|---|---|---|---|---|---|---|---|
| Cone radius (nm) | 44 | 77 | 111 | 143 | 208 | 266 | 357 |

**Output per advisory:** [intensity, [x,y], cone_size, [day,hour], [34kt_radii], [64kt_radii]]

[source: CHIME.nlogo:1569-1677] {CODE_VERIFIED}
{INFERRED — cone values closely match NHC official 5-year average forecast track errors circa 2009-2013 per code comment at line 1600. KG-6: exact NHC error table year/source not cited.}

**Instructions:** Describe the advisory matching and cone calculation. Present both cone error tables. Note the source inference.

#### Submodel: Publish-New-Mental-Model

**Purpose:** Interpolates the discrete forecast advisory to hourly resolution. Combines current best-track storm position with forecast positions.

**Algorithm:**
1. Take current forecast from Publish-Forecasts
2. Linearly interpolate between forecast time points for: wind speed, position (x,y), and cone error
3. Produce hourly forecast track combining known current position with forecast future positions

[source: CHIME.nlogo:1680-1804] {CODE_VERIFIED}

**Instructions:** Describe the interpolation process. Note that this creates the hourly interpreted forecast that all information agents use.

#### Submodel: Coastal-Patches-Alerts

**Purpose:** Coastal patches evaluate whether conditions warrant triggering an alert.

**Algorithm:**
1. For each coastal patch, check if the forecasted storm will arrive within `earliest` hours
2. Check if the patch is within the forecast cone of uncertainty
3. Check if forecast intensity exceeds `wind-threshold`
4. If ALL conditions met: set `alerts = 1`

**Parameters:**
| Parameter | Default | Range | Units | Description |
|-----------|---------|-------|-------|-------------|
| `earliest` | 60 | 12-200 | hours | Maximum lead time for triggering alerts |
| `wind-threshold` | 72 | 70-130 | kt | Minimum forecast intensity for alerts |

[source: CHIME.nlogo:1807-1838] {CODE_VERIFIED}

**Instructions:** Describe all three conditions precisely. Include parameter table.

#### Submodel: Issue-Alerts

**Purpose:** Officials in counties with alerted coastal patches issue evacuation orders.

**Algorithm:**
1. For each official whose county contains patches with `alerts = 1`: set `orders = 1`
2. Record `when-issued` (hours before storm arrival at the official's location)

[source: CHIME.nlogo:1840-1868] {CODE_VERIFIED}

**Instructions:** Brief section. Note that orders are county-wide and binary.

#### Submodel: Process-Forecasts

**Purpose:** Citizens collect forecasts from all their information sources and blend them into a single interpreted-forecast.

**Algorithm:**
1. Collect forecasts from: memory [self-trust, previous interpreted-forecast], each broadcaster in broadcaster-list [trust-factor, broadcast], each aggregator in aggregator-list [trust-factor, info], each network connection in my-network-list [trust-factor, partner's interpreted-forecast]
2. Random filtering: shuffle collected forecasts, drop a random number of them (bounded rationality simplification) {INFERRED — code comments at lines 2056-2058 note "We may want to rethink which forecasts are kept"}
3. Trust-weighted averaging: for each forecast variable V at each time t:

$$V_{\text{interpreted}}(t) = \frac{\sum_{i} \text{trust}_i \cdot V_i(t)}{\sum_{j} \text{trust}_j}$$

Applied to: intensity, position x, position y, and cone error at each hourly time point.

4. Store result as `interpreted-forecast`

[source: CHIME.nlogo:2038-2122] {CODE_VERIFIED}

**Instructions:** Describe the information collection process step by step. Write the trust-weighted average formula in LaTeX. Note the random filtering as a deliberate bounded rationality mechanism. KG-7: whether improved filtering was implemented in later work is unknown.

#### Submodel: Decision-Module

**Purpose:** Core decision-making process. Calculates risk and compares against thresholds to select protective action.

**This is the most critical submodel. Write it with full mathematical precision.**

**Step 1 — Calculate time and distance to closest approach:**
Using the citizen's `interpreted-forecast`, find the forecast hour at which the storm passes closest to the citizen's location. Let `x` = hours until this closest approach.

**Step 2 — Calculate Gaussian risk:**

$$\text{risk} = \frac{1}{\text{height} \cdot \sqrt{2\pi}} \cdot \exp\left(-\frac{(x - \text{center})^2}{2 \cdot \text{sd}^2}\right)$$

Where:
- $x$ = hours until storm reaches closest point to citizen-agent
- $\text{center} \sim \mathcal{N}(36, 3)$ — peak risk perception ~36 hours before arrival {INFERRED — corresponds to NHC hurricane watch timing}
- $\text{sd} \sim \mathcal{N}(24, 12)$ — spread of the risk function
- $\text{height}$ is calculated as:

$$\text{dist\_trk} = \left(\text{scale} \cdot 0.5 \cdot \text{distance} \cdot \frac{0.0011}{\text{error\_bars}}\right) + 0.0011$$

$$\text{height} = \sqrt{\text{dist\_trk} + 0.003 \cdot \text{zone} + 0.000525 \cdot \text{intensity}}$$

Where:
- `distance` = distance from citizen to closest point on forecast track
- `error_bars` = cone of uncertainty radius at time of closest approach
- `scale` = nautical miles per grid cell
- `zone` = 0 if evacuation zone is "A" (coastal), 1 otherwise
- `intensity` = 0 if forecast intensity >= 95 kt (Category 3+), 1 otherwise

**Note on height:** Smaller height values produce LARGER risk values (height is in the denominator). Coastal citizens (zone=0), citizens within the cone (small dist_trk), and citizens facing major hurricanes (intensity=0) all have smaller height values and thus higher risk.
{INFERRED — binary intensity treatment references Morss & Hayden (2010) Fig. 5 and Zhang et al. (2007) Fig. 5. Code comment acknowledges "We may want to rethink this."}

**Step 3 — Assemble final risk:**

$$\text{final\_risk} = w_f \cdot \left(1.1 \cdot \mathcal{N}(\text{risk}, 0.5)\right) + w_e \cdot \left(\text{trust\_authority} \cdot 6 \cdot \text{orders} \cdot \text{zone\_factor}\right) + w_c \cdot \left(3 \cdot \text{environmental\_cues}\right)$$

Where:
- $w_f$ = `forc-weight` (default 1.0, range [0, 2])
- $w_e$ = `evac-weight` (default 1.0, range [0, 4])
- $w_c$ = `envc-weight` (default 1.0, range [0, 6])
- `orders` = 0 or 1 (whether official has issued evacuation order)
- `zone_factor` = 1 if evacuation zone is "A", 0.4 otherwise
- `environmental_cues` = 0 or 1 (whether within 34-kt wind radius)
- The 1.1 multiplier is a 10% upward bias on raw risk {INFERRED — calibration adjustment}
- The 6 and 3 are scaling constants for orders and environmental cues respectively {INFERRED — calibrated to produce appropriate risk contributions}

**Step 4 — Census demographic modifications** (when `use-census-data` = On):
Applied as additive/subtractive percentage adjustments to `final_risk`:

| Demographic Factor | Direction | Slider Parameter | Effect |
|---|---|---|---|
| `kids-under-18?` | Increase | `under-18-assessment-increase` | +modifier * final_risk |
| `adults-over-65?` | Decrease | `over-65-assessment-decrease` | -modifier * final_risk |
| `limited-english?` | Decrease | `limited-english-assessment-decrease` | -modifier * final_risk |
| `food-stamps?` | Decrease | `foodstamps-assessment-decrease` | -modifier * final_risk |
| `no-vehicle?` | Decrease | `no-vehicle-assessment-modification` | -modifier * final_risk |
| `no-internet?` | Decrease | `no-internet-assessment-modification` | -modifier * final_risk |

{INFERRED — over-65 decrease reflects empirical finding that elderly are less likely to evacuate; kids-under-18 increase reflects parental protective instinct}

**Step 5 — Decision thresholds:**

| Condition | Action | Side Effect |
|-----------|--------|-------------|
| `final_risk > risk-life-threshold` | **Evacuate** | Agent color = orange; record ["evacuate" clock counter] in completed; set `evacuated? = true` |
| `risk-property-threshold < final_risk <= risk-life-threshold` | **Other protective action** | Halve `decision-module-interval` (min 1) |
| `info-up < final_risk <= risk-property-threshold` | **Increase info collection** | Halve `decision-module-interval` (min 1) |
| `info-down < final_risk <= info-up` | **No change** | — |
| `final_risk <= info-down` | **Decrease info collection** | Double `decision-module-interval` (max 32) |

[source: CHIME.nlogo:1872-2035] {CODE_VERIFIED}

**Instructions:** This section MUST include all equations in LaTeX, the complete parameter table with values/units/ranges, the step-by-step algorithm, and the decision threshold table. Use the exact variable names. Include the note about height being in the denominator. Flag KG-4 (calibration methodology) and KG-8 (sensitivity analysis results) as knowledge gaps.

#### Submodel: Social-Network

**Purpose:** Creates a scale-free network using modified preferential attachment at initialization.

**Algorithm:**
1. Each citizen-agent searches within `network-distance` (default 50 patches) for potential partners
2. Partners selected probabilistically: probability proportional to (partner's current network-list length)^`network-size`
3. Triadic closure step: each citizen's network member shares one of its own connections with the citizen
4. Trust factors assigned randomly from Uniform[0, 1] for each connection
5. Network connections sorted by trust factor

**Parameters:**
| Parameter | Default | Range | Units | Description |
|-----------|---------|-------|-------|-------------|
| `network-distance` | 50 | 0-50 | patches | Maximum distance for finding network partners |
| `network-size` | 10 | 1-10 | — | Power law exponent for preferential attachment |

[source: CHIME.nlogo:1452-1518] {CODE_VERIFIED}

**Instructions:** Describe the algorithm step by step. Note that the network is created once at initialization and does not change during the simulation.

#### Submodel: Check-Zone

**Purpose:** Determines each citizen-agent's perceived evacuation zone based on distance from the nearest coastal patch.

**Algorithm:**
1. Calculate distance from citizen to nearest coastal patch
2. Assign zone:
   - Zone A: distance <= 1.5 grid points
   - Zone B: 1.5 < distance <= 3 grid points
   - Zone C: 3 < distance <= 5 grid points
   - No zone: distance > 5 grid points
3. With 20% probability, override the distance-based assignment with a random zone (modeling imperfect zone knowledge)

[source: CHIME.nlogo:1170-1187] {CODE_VERIFIED}

**Instructions:** Present the zone thresholds in a table. Note the 20% error rate and its justification.

---

## Interface Parameters Summary

**What to write:** Include a comprehensive parameter table as an appendix or integrated into relevant sections.

| Parameter | Range | Default | Units | Description |
|-----------|-------|---------|-------|-------------|
| `which-storm?` | Chooser | CHARLEY_REAL | — | Storm scenario selection |
| `#citizen-agents` | 0-5000 | 2000 | agents | Number of citizen-agents |
| `#broadcasters` | 0-10 | 10 | agents | Number of broadcasters |
| `#net-aggregators` | 0-10 | 10 | agents | Number of aggregators |
| `distribute-population` | Switch | Off | — | Place citizens by population density |
| `use-census-data` | Switch | Off | — | Use census tract data (Florida only) |
| `earliest` | 12-200 | 60 | hours | Lead time for official alert trigger |
| `latest` | 0-12 | 12 | hours | (Appears unused in current code) |
| `wind-threshold` | 70-130 | 72 | kt | Wind speed threshold for alerts |
| `forc-weight` | 0-2 | 1.0 | — | Weight of forecast info in risk calculation |
| `evac-weight` | 0-4 | 1.0 | — | Weight of evacuation orders in risk calculation |
| `envc-weight` | 0-6 | 1.0 | — | Weight of environmental cues in risk calculation |
| `network-distance` | 0-50 | 50 | patches | Max distance for social network links |
| `network-size` | 1-10 | 10 | — | Power law exponent for network |
| `citizen-to-census-population-ratio` | 0-10000 | 5000 | — | Census population per agent ratio |
| Census factor switches | On/Off | Off | — | Toggle each demographic factor |
| Census modification sliders | Various | Various | — | Magnitude of demographic risk modifications |
| `display-dem?` | Switch | — | — | Display elevation raster |

[source: CHIME.nlogo BehaviorSpace and interface sections] {CODE_VERIFIED}

---

## Quality Standards

### Reimplementability

A competent modeler who has never seen the code must be able to reimplement the model from the ODD alone. This means:
- All parameters specified with values, units, and ranges
- All decision rules precisely defined with exact conditions and outcomes
- All equations written in LaTeX notation
- All algorithms described in pseudocode or step-by-step logic
- All boundary conditions and edge cases addressed (e.g., clamping of decision-module-interval to [1, 32])
- All stochastic distributions specified with parameters

### Confidence Annotations

Tag every factual claim with one of:
- `{CODE_VERIFIED}` — verified by reading actual code [file:line]
- `{DOC_STATED}` — explicitly stated in documentation [doc:section]
- `{MODELER_CONFIRMED}` — confirmed by modeler during interview [interview Q#]
- `{INFERRED}` — reasonably inferred from code behavior or literature [inference chain]
- `{UNVERIFIABLE}` — cannot be verified from available sources [reason]

### Citation Format

Inline citations: `[source: CHIME.nlogo:42]` or `[source: README.md:7]` or `[source: How To Use CHIME.md:3]`

### Writing Rules

1. Use exact terminology from the Terminology table — never substitute synonyms
2. Never paraphrase technical concepts into simpler language
3. Describe what the program does, not what you think the model does
4. Include rationale subsections where design justifications are available
5. Preserve the exact spelling of code variables (e.g., `final-risk-assesment` with its misspelling)
6. When describing equations, show both the mathematical notation and the variable names used in code

---

## Sub-agent Instructions

**Delegation strategy:** Main agent handles entire ODD (moderate complexity).

A code analysis sub-agent MAY be spawned for Element 7 (Submodels) if the main agent needs to verify specific line references or trace additional code paths. The sub-agent should:
- Tool: Read tool on `CHIME.nlogo`
- Task: Verify specific line references cited in the Knowledge Base
- Return: Exact code excerpts with line numbers

For most purposes, the Knowledge Base above contains all necessary information and sub-agent delegation is optional.

---

## Traceability Matrix

The draft must also produce `lazyodd/draft/traceability-matrix.md` mapping every ODD claim to its source.

Format:
| ODD Section | Claim | Source | Confidence | Notes |
|-------------|-------|--------|------------|-------|

Populate this matrix as you write each section. Every substantive claim in the ODD must appear in the matrix.

---

## Consolidated Knowledge Gaps

These gaps should be acknowledged in the ODD document where relevant:

| ID | Element | Gap | Impact on ODD |
|----|---------|-----|---------------|
| KG-1 | 1 (Purpose) | Quantitative target evacuation rates/timing from historical data | Purpose section notes that targets exist but are not encoded in code |
| KG-2 | 1 (Purpose) | Patterns the model failed to reproduce | Cannot assess model limitations |
| KG-3 | 1 (Purpose) | Specific hypotheses from Watts et al. 2019 | Purpose statement references paper but cannot reproduce specific hypotheses |
| KG-4 | 2, 7 (Entities, Submodels) | Formal calibration methodology for thresholds and constants | Rationale relies on inference rather than documented methodology |
| KG-5 | 6 (Input Data) | Exact methodology for ideal/bad forecast construction | Input data section partially inferred |
| KG-6 | 7 (Submodels) | NHC cone error table year/source | Minor — values match published NHC data |
| KG-7 | 7 (Submodels) | Whether improved forecast filtering was implemented in later work | Minor — current mechanism fully documented |
| KG-8 | 7 (Submodels) | Formal sensitivity analysis results | Rationale for Decision-Module calibration constants partially inferred |

---

## Output Specifications

- Primary document: `lazyodd/draft/odd.md`
- Traceability matrix: `lazyodd/draft/traceability-matrix.md`
- Create `lazyodd/draft/` directory if it does not exist
- Format: Markdown with inline citations and confidence annotations
- Use LaTeX notation ($$...$$) for all equations
- Use tables for parameters, state variables, and structured data
- Target length: 8,000-12,000 words for the ODD document
