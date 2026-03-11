# ODD Verification Report: CHIME ABM

> Verified by lazyodd:check | Date: 2026-03-11
> ODD Document: lazyodd/draft/odd.md
> Overall Grade: **B** (3.9 / 5.0)

## Summary

The ODD document is a comprehensive, well-structured description of the CHIME ABM that successfully covers all seven ODD elements and all eleven design concepts. The writing is precise, equations are fully specified in LaTeX, and nearly every claim carries an inline citation and confidence annotation. However, code verification revealed several discrepancies: the final risk assembly equation omits a two-stage weight application present in the code, the state variable table omits five declared citizen-agent variables, and the adaptive decision interval description is incomplete. These issues are fixable and do not fundamentally undermine the document, but they would lead to a subtly incorrect reimplementation if taken literally.

## Scores by Element

| Element | Completeness | Precision | Traceability | Consistency | Overall |
|---------|-------------|-----------|--------------|-------------|---------|
| 1. Purpose and Patterns | 5 | 4 | 5 | 5 | 4.75 |
| 2. Entities, State Variables, and Scales | 4 | 4 | 5 | 4 | 4.25 |
| 3. Process Overview and Scheduling | 5 | 4 | 5 | 4 | 4.50 |
| 4. Design Concepts | 5 | 4 | 5 | 5 | 4.75 |
| 5. Initialization | 4 | 4 | 5 | 4 | 4.25 |
| 6. Input Data | 5 | 4 | 4 | 5 | 4.50 |
| 7. Submodels | 4 | 3 | 5 | 3 | 3.75 |
| **Average** | **4.6** | **3.9** | **4.9** | **4.3** | **4.11** |

### Grading Scale
- **A** (4.5-5.0): Publication-ready, minor polish only
- **B** (3.5-4.4): Good quality, some improvements needed
- **C** (2.5-3.4): Adequate but significant gaps
- **D** (1.5-2.4): Major revisions needed
- **F** (1.0-1.4): Fundamental issues, consider re-drafting

---

## Issues Found

### Critical (must fix)

1. **[Element 7.7]: Final risk assembly equation does not match code's two-stage weight application**
   - **Problem:** The ODD presents the final risk as a single weighted sum:
     $$\text{final\_risk} = w_f \cdot (1.1 \cdot \mathcal{N}(\text{risk}, 0.5)) + w_e \cdot (\text{trust\_authority} \cdot 6 \cdot \text{orders} \cdot \text{zone\_factor}) + w_c \cdot (3 \cdot \text{environmental\_cues})$$
     However, the actual code (lines 1967-1999) uses a **two-stage** process: (1) compute unweighted components, accumulate them, then (2) apply weights in a second pass. The intermediate unweighted values are stored for bookkeeping (`temp-f-risk`), and the final weighted sum uses these stored components.
   - **Evidence:** CHIME.nlogo lines 1967-1999:
     - Line 1967: `set final-risk-assesment random-normal risk .5`
     - Line 1971: `set final-risk-assesment 1.1 * final-risk-assesment`
     - Line 1978: `set final-risk-assesment final-risk-assesment + (trust-authority * 6 * official-orders * zone)`
     - Line 1984: `set final-risk-assesment final-risk-assesment + (3 * environmental-cues)`
     - Lines 1994-1999: Second pass applies weights per-component and re-sums.
   - **Fix:** Rewrite the equation as a two-stage process. Stage 1: compute `temp-f-risk = 1.1 * N(risk, 0.5)`, `temp-e-risk = trust_authority * 6 * orders * zone_factor`, `temp-c-risk = 3 * environmental_cues`. Stage 2: `final_risk = (temp-f-risk * forc-weight) + (temp-e-risk * evac-weight) + (temp-c-risk * envc-weight)`. Show both stages explicitly. The current single-equation form is mathematically equivalent when weights = 1.0 but misrepresents the code structure and the intermediate bookkeeping.

2. **[Element 7.7]: dist_trk zeroing for citizens inside the cone is omitted**
   - **Problem:** The ODD presents the dist_trk formula but omits the critical prior step: if the citizen is inside the cone of uncertainty (i.e., `scale * distance < error_bars`), `dist_trk` is set to 0 before the formula is applied. This means the dist_trk formula as written only applies to citizens *outside* the cone.
   - **Evidence:** CHIME.nlogo line 1951: `if (scale * dist-trk) < error-bars [set dist-trk 0]` — this occurs before the main dist_trk formula at line 1954.
   - **Fix:** Add a conditional before the dist_trk formula: "If the citizen is within the cone of uncertainty ($\text{scale} \cdot \text{distance} < \text{error\_bars}$), then $\text{dist\_trk} = 0$. Otherwise, compute $\text{dist\_trk} = ...$" This is essential for reimplementability — without it, citizens inside the cone would have incorrect risk values.

3. **[Element 2.2]: Citizen-agent state variable table omits 5 declared variables**
   - **Problem:** The ODD's citizen-agent state variable table omits five variables that are declared in the code: `distance-to-storm-track`, `previous-dm-interval`, `forecast-options`, `when-evac-1st-ordered`, `risk-watcher`.
   - **Evidence:** CHIME.nlogo lines 112, 115, 121, 127, 128 — all five are declared in the `citizen-agents-own` block.
   - **Fix:** Add these variables to the state variable table. Note that some may be unused or vestigial (e.g., `risk-watcher` appears to be for plotting only; `previous-dm-interval` stores the pre-feedback interval), but all declared variables should be documented per ODD completeness requirements.

### Major (should fix)

4. **[Element 7.7 / 4.3]: Adaptive decision interval description is incomplete**
   - **Problem:** The ODD states that `decision-module-interval` halves "when risk exceeds `info-up`" and doubles "when risk falls below `info-down`". However, the code halves the interval in **two** branches: (1) when `risk-property-threshold < risk < risk-life-threshold` (the "other protective action" branch) AND (2) when `info-up < risk < risk-property-threshold`. The ODD only describes the latter.
   - **Evidence:** CHIME.nlogo lines 2018 and 2021 both halve `decision-module-interval`.
   - **Fix:** Update the threshold table in Section 7.7 Step 5 and Section 4.3 to show that halving occurs in *both* the "other protective action" and "increase information collection" branches.

5. **[Element 3 / 7.6]: Evacuated citizen behavior oversimplified**
   - **Problem:** The ODD states that evacuated citizens "still collect and process information (Process-Forecasts)". This implies they run Process-Forecasts every tick. The actual code shows that evacuated citizens are *also* gated by the `decision-module-turn < decision-module-interval` counter — they only run Process-Forecasts when their turn counter expires, not every tick.
   - **Evidence:** CHIME.nlogo lines 268-273: the else branch for evacuated citizens also checks `decision-module-turn < decision-module-interval`.
   - **Fix:** Clarify in Section 3 step 8 and Section 7.6 that evacuated citizens run Process-Forecasts on the same staggered schedule as non-evacuated citizens, not every tick.

6. **[Element 5.1]: Coastal patch definition discrepancy**
   - **Problem:** The ODD states coastal patches are "land patches adjacent to ocean patches" and later says "within a neighborhood of 3 patches." The code at line 396 uses `any? neighbors with [land? = false]`, where `neighbors` in NetLogo means the 8 immediately adjacent patches (Moore neighborhood), not 3 patches distance.
   - **Evidence:** CHIME.nlogo line 396: `set coastal-patches land-patches with [any? neighbors with [land? = false]]`
   - **Fix:** Remove the "within a neighborhood of 3 patches" mention. Simply state "land patches that have at least one ocean patch in their Moore neighborhood (8 adjacent patches)."

7. **[Element 7]: Missing submodel — Environmental Cues calculation**
   - **Problem:** The environmental cues calculation (how `environmental-cues` is set to 0 or 1 based on 34-kt wind radius and quadrant) is described in Section 4.7 (Sensing) but is not given its own submodel entry in Section 7. Given that this is a distinct algorithmic process with spatial/geometric calculation (quadrant determination, distance comparison against quadrant-specific wind radii), it warrants a submodel description.
   - **Evidence:** CHIME.nlogo lines 1896-1905 implement a non-trivial algorithm.
   - **Fix:** Add a "7.10 Environmental Cues" submodel section describing the quadrant determination algorithm and the distance-based 34-kt radius check.

8. **[Element 8]: Census modification slider defaults not specified**
   - **Problem:** The interface parameter table lists census modification sliders (e.g., `under-18-assessment-increase`) with "—" for default values. For reimplementability, these values need to be specified.
   - **Evidence:** The interface section of CHIME.nlogo would contain these defaults, but the ODD does not extract them.
   - **Fix:** Read the interface section of CHIME.nlogo and populate the default values for all census modification sliders.

### Minor (nice to fix)

9. **[Element 5]: Legend placement step omitted from initialization sequence**
   - **Problem:** Between Load-GIS and Load-Hurricane, the Setup procedure loads a legend image based on a `where-to-place-legend?` chooser. This is a visualization step and is minor, but for completeness it should be mentioned.
   - **Evidence:** CHIME.nlogo lines 187-191.
   - **Fix:** Add a brief note after the Load-GIS step: "A legend image is loaded for the NetLogo interface display."

10. **[Element 7.2]: Cone error table source year imprecise**
    - **Problem:** The ODD says cone values match "NHC official 5-year average forecast track errors circa 2009-2013 and 2014-2018 respectively" but admits this is INFERRED. The two different tables (for older vs. newer storms) suggest different NHC error periods were used.
    - **Evidence:** Code comments at approximately line 1600 reference NHC average errors but do not cite specific years.
    - **Fix:** No action required beyond the existing {INFERRED} annotation, but consider adding a note that the two tables likely correspond to different NHC 5-year error periods.

11. **[Element 2.3]: `when-issued` description could be more precise**
    - **Problem:** The ODD says `when-issued` records "Tick at which evacuation orders were issued." The code actually records hours-before-storm-arrival at the official's location, which is different from the raw tick number.
    - **Evidence:** CHIME.nlogo line ~1860: `when-issued` is set relative to storm arrival timing.
    - **Fix:** Clarify whether `when-issued` stores the absolute tick or the relative time-before-arrival.

12. **[Element 9]: Morss et al. 2017 citation incorrect in References**
    - **Problem:** The reference for Morss et al. 2017 in the ODD References section gives the title as "Flash Flood Risks and Warning Decisions..." which appears to be a different paper. The README cites Morss et al. 2017 with the title "Hazardous Weather Prediction and Communication in the Modern Information Environment."
    - **Evidence:** README.md line 13 provides the correct citation.
    - **Fix:** Replace the Morss et al. 2017 reference with the correct citation from the README.

13. **[Element 9]: Watts et al. 2019 co-author list differs from README**
    - **Problem:** The ODD References list the authors as "Watts, J., Morss, R.E., Bowden, C.M., Lindell, M.K., & Murray-Tuite, P." but the README lists "Watts J., Morss R. E., Barton C. M., Demuth J. L."
    - **Evidence:** README.md line 15.
    - **Fix:** Use the author list from the README, which matches the actual publication.

---

## Verification Details

### A. Structural Completeness: PASS

- [x] All 7 ODD elements present (Sections 1-7)
- [x] All 11 design concepts addressed under Section 4 (4.1-4.11)
- [x] No empty or placeholder sections
- [x] Rationale provided where available (as inline notes and interpretation, though not as formal "Rationale" subsections)
- [x] Element ordering follows ODD+2 standard
- [x] ODD citation statement present ("The model description follows the ODD...")
- [x] Additional sections (8: Interface Parameters, 9: Knowledge Gaps) are appropriate extensions

**Note:** Formal "Rationale" subsections are not used; instead, rationale is woven into the prose. This is acceptable per ODD+2 guidance, which states rationale may be omitted when describing a model developed by someone else or when the ODD serves as documentation based on code review.

### B. Source Traceability: PASS

- [x] Every factual claim has an inline citation (verified across all sections)
- [x] Citations reference real files that exist (CHIME.nlogo, README.md, How To Use CHIME.md)
- [x] Traceability matrix is comprehensive (~247 claims)
- [x] Traceability matrix is consistent with inline citations
- [x] No orphaned claims found
- [x] Confidence categories present on all claims

**Citation verification sample (10 claims verified against code):**

| Claim | Source Cited | Verified? |
|-------|-------------|-----------|
| self-trust = 0.6 + random-float 0.4 | CHIME.nlogo:1132 | CONFIRMED |
| risk-life-threshold ~ N(14, 2) | CHIME.nlogo:1142 | CONFIRMED |
| Gaussian risk formula | CHIME.nlogo:1962 | CONFIRMED |
| center ~ N(36, 3) | CHIME.nlogo:1934 | CONFIRMED |
| Zone A <= 1.5 grid points | CHIME.nlogo:1181 | CONFIRMED |
| 20% zone error rate | CHIME.nlogo:1179 | CONFIRMED |
| Aggregator 1/3 update chance | CHIME.nlogo:257 | CONFIRMED |
| Coastal alerts require 3 conditions | CHIME.nlogo:1835 | CONFIRMED |
| height = sqrt(dist_trk + 0.003*zone + 0.000525*intensity) | CHIME.nlogo:1959 | CONFIRMED |
| zone_factor = 1 if "A", 0.4 otherwise | CHIME.nlogo:1977 | CONFIRMED |

### C. Semantic Consistency: PARTIAL

- [x] Terminology used consistently throughout (citizen-agent, forecaster, etc.)
- [x] Parameter names match between sections
- [x] Process order in Element 3 consistent with Element 7 submodel descriptions
- [x] Entity types in Element 2 match those in Element 4
- [ ] **Issue:** Element 2 state variable table is incomplete vs. actual code declarations (5 variables missing)
- [ ] **Issue:** `when-issued` description differs between Element 2 (tick) and Element 7.5 (hours before arrival)

### D. Code Alignment: PARTIAL

- [x] Process execution order (Go procedure) matches ODD Element 3
- [x] Entity state variables mostly match code
- [x] Initialization values match code defaults
- [x] Most decision rules match implemented logic
- [ ] **Issue:** Final risk equation omits two-stage structure (Critical #1)
- [ ] **Issue:** dist_trk zeroing for within-cone citizens omitted (Critical #2)
- [ ] **Issue:** Adaptive interval halving incomplete — occurs in two branches, not one (Major #4)
- [ ] **Issue:** Evacuated citizen scheduling gated by turn counter, not every tick (Major #5)

### E. Reimplementability: PARTIAL

- [x] Most parameters specified with values and ranges
- [x] Gaussian risk equation fully specified with all variables defined
- [x] Stochastic distributions specified with parameters
- [x] Spatial and temporal structure fully specified
- [ ] **Issue:** dist_trk conditional zeroing missing — reimplementation would compute incorrect risk for citizens inside the cone
- [ ] **Issue:** Two-stage weight application — reimplementation from single equation would produce same result only when weights = 1.0
- [ ] **Issue:** Census slider defaults not provided — reimplementer cannot set defaults
- [ ] **Issue:** Environmental cues algorithm not given as submodel — quadrant determination logic not described

### F. Confidence Audit: PASS

- [x] CODE_VERIFIED claims verified against actual code (10/10 sample confirmed)
- [x] INFERRED claims have documented reasoning
- [x] UNVERIFIABLE claims explain why (missing publications)
- [x] No claims marked CODE_VERIFIED that contradict code (the discrepancies are in claims correctly marked CODE_VERIFIED but with incomplete description, not incorrect description)
- [x] Distribution of confidence is reasonable: ~210 CODE_VERIFIED, ~18 DOC_STATED, ~14 INFERRED, ~5 UNVERIFIABLE

**Note on CODE_VERIFIED vs. code contradictions:** The critical issues (#1, #2) involve claims that are marked {CODE_VERIFIED} but present an incomplete or simplified version of the code logic. The claimed equations are not *wrong* per se — the single-equation form is mathematically equivalent when weights = 1.0 — but they omit structural details that matter for reimplementation and understanding. This is an accuracy-of-description issue, not a confidence-misclassification issue.

---

## Recommendations

Prioritized by impact on ODD quality:

1. **Fix the final risk equation (Critical #1)** — This is the most impactful improvement. Show the two-stage computation explicitly. A reimplementer following the current equation would get correct results only for default weights, and would miss the intermediate bookkeeping structure.

2. **Add the dist_trk conditional zeroing (Critical #2)** — Without this, a reimplementer would compute wrong risk values for the most critical population: citizens inside the cone of uncertainty.

3. **Add the 5 missing state variables (Critical #3)** — Completeness requirement for ODD. Even if some are vestigial, they should be documented.

4. **Correct the adaptive interval description (Major #4)** — The interval halves in two branches, not one. This affects behavior for citizens between `info-up` and `risk-life-threshold`.

5. **Clarify evacuated citizen scheduling (Major #5)** — They don't run Process-Forecasts every tick; they're gated by the same turn counter.

6. **Add Environmental Cues submodel (Major #7)** — The quadrant-based 34-kt wind radius check is a non-trivial spatial algorithm that belongs in Section 7.

7. **Fix reference citations (Minor #12, #13)** — Morss et al. 2017 title is wrong; Watts et al. 2019 author list doesn't match README.

8. **Populate census slider defaults (Major #8)** — Extract from the CHIME.nlogo interface section.
