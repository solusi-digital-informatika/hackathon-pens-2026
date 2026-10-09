# Test Contract: Demo Scenario and Ground-Truth Affected Shots

**Document Version:** 1.0  
**Status:** Approved Specification  
**Owner:** Tester (`tester`)  
**References:**  
- `project-map/PRD.md` (§11, §12.3, §14, §16)  
- `project-map/product-brief.md` (§ Acceptance scenario, § Metrics)  
- `board/board.md` (Task T-002)

---

## 1. Executive Purpose & Scope

This contract establishes the canonical ground-truth test dataset and end-to-end evaluation scenario for the **AI Production Workflow Assistant**. It serves as the single source of truth for:
1. Validating the AI impact analysis engine (FR-07, FR-08).
2. Verifying the versioning and human approval lifecycle (FR-06, FR-09, FR-10, FR-11).
3. Benchmarking system performance against the quantitative metrics mandated in PRD §14 without fabricating unverified claims.

Before any backend or frontend implementation of change analysis is accepted, automated and manual tests must evaluate against this defined ground-truth baseline.

---

## 2. Production Scenario Definition

### 2.1 Project Profile
- **Project ID:** `proj_cyberpulse_demo`
- **Project Name:** *CyberPulse: The Neon Courier*
- **Format:** 6-shot short narrative sequence (approx. 15–20 seconds total preview).
- **Subject:** Aria, an independent courier transporting an encrypted data core across a futuristic metropolis.

---

### 2.2 Direction Version 1 (V1 — Baseline)
- **Direction ID:** `dir_v1_cinematic`
- **Version Number:** `1`
- **Title:** Realistic Cinematic Cyberpunk (Moody Night)
- **Status:** Approved / Active
- **Creative Specifications:**
  - **Art Style (`REQ-STYLE-01`):** Photorealistic live-action cinema, heavy anamorphic lens flare, shallow depth of field, naturalistic micro-textures, film grain.
  - **Lighting & Color Palette (`REQ-LIGHT-01`):** Moody nocturnal atmosphere; dominant cool blue (`#0A192F`) and cyan (`#00F0FF`) rim lighting; deep wet-asphalt contrast.
  - **Atmosphere & Weather (`REQ-ENV-01`):** Heavy rain, wet pavement reflections, foggy steam vents.
  - **Main Character Identity (`REQ-CHAR-01`):** Protagonist "Aria": athletic build, dark messy undercut, subtle chrome cybernetic left eye implant with circular cyan optic ring, high-collar weather-worn dark courier duster jacket.
  - **Key Artifact (`REQ-PROP-01`):** Hexagonal encrypted data core slate with amber indicator display.

---

### 2.3 Direction Version 2 (V2 — Proposed Revision)
- **Direction ID:** `dir_v2_stylized`
- **Version Number:** `2` (Draft / Pending Review)
- **Title:** Stylized Anime / Cel-Shaded Sunset
- **Change Note:** "Artistic pivot: Transition from gritty photoreal night to vibrant cel-shaded anime aesthetic set during warm golden sunset. Main character identity and courier jacket silhouette must remain strictly consistent."
- **Creative Specifications & Delta:**
  - **Art Style (`REQ-STYLE-02` — CHANGED):** 2D/3D hybrid stylized animation, crisp lineart, graphic cel-shading, vibrant saturated palette, expressive non-photoreal rendering (NPR).
  - **Lighting & Color Palette (`REQ-LIGHT-02` — CHANGED):** Warm golden hour / dusk; glowing amber (`#FF9900`), warm tangerine (`#FF5E00`), and magenta highlights; soft warm ambient fill.
  - **Atmosphere & Weather (`REQ-ENV-02` — CHANGED):** Clear dusk sky, golden particulates/dust motes, dry urban surfaces (rain completely removed).
  - **Main Character Identity (`REQ-CHAR-01` — INVARIANT / PRESERVED):** Aria's facial geometry, hair shape, cybernetic eye placement, and high-collar courier jacket silhouette **must remain consistent**.
  - **Key Artifact (`REQ-PROP-01` — INVARIANT / PRESERVED):** Hexagonal data core geometry and interface layout remain unchanged.

---

## 3. Six-Shot Master Specification & Ground-Truth Mapping

Every shot is defined with explicit metadata, V1 state, V2 delta, assigned ground-truth action, and the rigorous rationale required by PRD §12.3.

```
Action Vocabulary (PRD §12.3):
- RETAIN:      Existing work is fully compatible with V2; no generation or adaptation needed.
- ADAPT:       Core assets/composition can be preserved; prompt or styling needs targeted revision.
- REGENERATE:  Existing asset is fundamentally incompatible with V2 requirements; must be generated anew.
- NEEDS_REVIEW: Evidence or dependencies are missing/conflicting; requires human determination.
```

---

### Shot 1: SHOT-01 — Establishing Aerial Alleyway

| Attribute | Specification |
|---|---|
| **Shot ID** | `SHOT-01` |
| **Sequence Order** | 1 |
| **Title** | Establishing Aerial Alleyway |
| **Scene Narrative** | Extreme wide crane shot looking down into a narrow canyon between skyscrapers down to street level. |
| **V1 Prompt** | `"Cinematic 8k wide crane shot of cyberpunk city canyon, photorealistic skyscrapers, pouring heavy rain, slick asphalt reflections, deep cool blue lighting, neon cyan glow, anamorphic lens, realistic film grain"` |
| **V1 Asset URI** | `assets/shot01_v1_establishing_photoreal.png` (Asset Type: `image/png`, State: `approved`) |
| **V1 Dependencies** | `REQ-STYLE-01` (Realistic Photoreal), `REQ-LIGHT-01` (Cool Blue), `REQ-ENV-01` (Heavy Rain) |
| **V2 Impact Analysis** | All three linked requirements are invalidated: style changes to cel-shaded anime (`REQ-STYLE-02`), lighting changes to warm golden hour (`REQ-LIGHT-02`), and rain is eliminated (`REQ-ENV-02`). There is no character or reusable layer. |
| **Ground-Truth Action** | **`regenerate`** |
| **Ground-Truth Rationale** | The asset is a single-layer photorealistic background plate dominated by cool blue wet rain reflections. None of the pixel data or prompt descriptors can satisfy the warm anime dusk brief. The shot must be generated from scratch. |
| **Confidence Level** | **High (0.95)** — Direct dependency conflict with no reusable components. |

---

### Shot 2: SHOT-02 — Aria Medium Portrait

| Attribute | Specification |
|---|---|
| **Shot ID** | `SHOT-02` |
| **Sequence Order** | 2 |
| **Title** | Aria Medium Portrait |
| **Scene Narrative** | Medium portrait shot of Aria pausing under a doorway canopy, looking at camera, her left cybernetic eye pulsing gently. |
| **V1 Prompt** | `"Medium portrait of female courier Aria, 25 years old, athletic build, messy dark undercut, glowing cyan chrome cybernetic left eye, high-collar dark leather duster, photorealistic skin texture, cool cyan rim lighting, shallow depth of field"` |
| **V1 Asset URI** | `assets/shot02_v1_aria_medium.png` (Asset Type: `image/png`, State: `approved`) |
| **V1 Dependencies** | `REQ-CHAR-01` (Aria Identity), `REQ-STYLE-01` (Realistic Photoreal), `REQ-LIGHT-01` (Cool Blue) |
| **V2 Impact Analysis** | `REQ-CHAR-01` is strictly preserved (same facial structure, hair, eye implant, coat collar). `REQ-STYLE-01` changes to cel-shaded anime (`REQ-STYLE-02`) and `REQ-LIGHT-01` changes to warm amber rim (`REQ-LIGHT-02`). |
| **Ground-Truth Action** | **`adapt`** |
| **Ground-Truth Rationale** | In adherence to the "Adaptation over regeneration" principle (PRD §3, FR-08), the underlying character identity, framing, pose, and silhouette are intentionally preserved. The prompt and asset pipeline should adapt the existing character keyframe via style-transfer / cel-shaded re-prompting and warm color grading rather than discarding Aria's approved character model. |
| **Confidence Level** | **High (0.90)** — Invariant character identity anchors adaptation; prompt requires style and lighting modifier updates. |

---

### Shot 3: SHOT-03 — Holographic Data Core HUD Overlay

| Attribute | Specification |
|---|---|
| **Shot ID** | `SHOT-03` |
| **Sequence Order** | 3 |
| **Title** | Holographic Data Core HUD Overlay |
| **Scene Narrative** | Direct close-up on the hexagonal data core display projecting a minimalist holographic delivery manifest and coordinate map. |
| **V1 Prompt** | `"Diegetic vector graphic user interface, minimalist glowing holographic telemetry HUD, hexagonal blueprint layout, crisp typography, amber and white vector line work, isolated on transparent background, clean motion graphic design"` |
| **V1 Asset URI** | `assets/shot03_v1_data_hud.png` (Asset Type: `image/png`, State: `approved`) |
| **V1 Dependencies** | `REQ-PROP-01` (Hexagonal Data Core), `REQ-UI-01` (Clean Vector HUD) |
| **V2 Impact Analysis** | The HUD is self-illuminated vector graphic typography with amber accents. It contains no character, no photoreal 3D rendering, and no ambient environment lighting dependencies (`REQ-STYLE-01`, `REQ-LIGHT-01`, and `REQ-ENV-01` are absent from its dependency links). |
| **Ground-Truth Action** | **`retain`** |
| **Ground-Truth Rationale** | The graphic HUD asset is completely agnostic to the live-action vs. anime transition and is already tuned to amber/white hues. It satisfies Direction V2 without modification, demonstrating measurable asset reuse. |
| **Confidence Level** | **High (0.95)** — Zero dependency overlap with changed requirements. |

---

### Shot 4: SHOT-04 — Aria Rooftop Sprint

| Attribute | Specification |
|---|---|
| **Shot ID** | `SHOT-04` |
| **Sequence Order** | 4 |
| **Title** | Aria Rooftop Sprint |
| **Scene Narrative** | Dynamic tracking profile shot of Aria sprinting across an elevated duct, coat billowing behind her as she hurdles a conduit. |
| **V1 Prompt** | `"Dynamic profile action tracking shot of courier Aria sprinting across industrial rooftop pipe, coat tails trailing, motion blur, athletic hurdle posture, rainy night background, cool blue cinematic moonlight rim light"` |
| **V1 Asset URI** | `assets/shot04_v1_rooftop_animatic.mp4` (Asset Type: `video/mp4`, State: `approved`) |
| **V1 Dependencies** | `REQ-CHAR-01` (Aria Identity), `REQ-ACTION-01` (Rooftop Motion Layout), `REQ-STYLE-01`, `REQ-LIGHT-01` |
| **V2 Impact Analysis** | Motion blocking, pacing, hurdle choreography (`REQ-ACTION-01`), and Aria's silhouette (`REQ-CHAR-01`) remain valid. The visual render style and lighting must shift to warm cel-shaded animation. |
| **Ground-Truth Action** | **`adapt`** |
| **Ground-Truth Rationale** | The 3D animation path / 2D animatic layout and motion curves remain completely reusable. The prompt and generation parameters must be adapted to replace cool moonlight with warm sunset backlighting and apply anime speed lines/cel-shading. Discarding the existing motion blocking would constitute unnecessary rework. |
| **Confidence Level** | **Moderate-High (0.85)** — Core motion choreography is retained; rendering pass must be updated. |

---

### Shot 5: SHOT-05 — Raindrop Splash Macro Cutaway

| Attribute | Specification |
|---|---|
| **Shot ID** | `SHOT-05` |
| **Sequence Order** | 5 |
| **Title** | Raindrop Splash Macro Cutaway |
| **Scene Narrative** | High-speed macro slow-motion cutaway of a heavy raindrop splashing into a dark puddle, ripple reflecting cyan neon signs. |
| **V1 Prompt** | `"Macro 1000fps slow motion close-up of single raindrop impacting asphalt puddle, intricate fluid crown splash, photorealistic liquid dynamics, dark tarmac texture, distorted reflection of cyan neon sign"` |
| **V1 Asset URI** | `assets/shot05_v1_puddle_macro.png` (Asset Type: `image/png`, State: `approved`) |
| **V1 Dependencies** | `REQ-STYLE-01` (Realistic Photoreal), `REQ-LIGHT-01` (Cool Blue), `REQ-ENV-01` (Heavy Rain) |
| **V2 Impact Analysis** | Direction V2 completely eliminates rain (`REQ-ENV-02`: dry dusk atmosphere) and cool blue neon reflections (`REQ-LIGHT-02`). Fluid dynamics are incompatible with the new visual narrative and art style. |
| **Ground-Truth Action** | **`regenerate`** |
| **Ground-Truth Rationale** | The subject matter itself (a raindrop puddle splash at night) is obsolete under the sunny/dusk dry weather brief. No portion of the image or prompt can be adapted to represent a dry sunset anime sequence. A completely new cutaway shot (e.g., wind blowing dust motes or golden lens flare) must be conceived and generated. |
| **Confidence Level** | **High (0.95)** — Fundamental narrative and aesthetic obsolescence. |

---

### Shot 6: SHOT-06 — Mystery Figure in Alleyway Doorway

| Attribute | Specification |
|---|---|
| **Shot ID** | `SHOT-06` |
| **Sequence Order** | 6 |
| **Title** | Mystery Figure in Alleyway Doorway |
| **Scene Narrative** | Ominous shadowed silhouette of an unidentified figure observing from a recessed industrial doorway. |
| **V1 Prompt** | `"Shadowy silhouette of human figure standing in dark recessed doorway, backlit steam, indistinct features, high contrast noir framing"` |
| **V1 Asset URI** | `assets/shot06_v1_silhouette_stock.mov` (Asset Type: `video/mp4`, State: `draft_imported`) |
| **V1 Dependencies** | **MISSING / UNLINKED** (No explicit character link, no defined lighting tag in dependency graph, imported external reference asset) |
| **V2 Impact Analysis** | Because no character dependency was recorded, the system cannot verify whether this figure is Aria (subject to `REQ-CHAR-01` consistency), a secondary character, or ambient background. The lighting dependency is ambiguous (high-contrast noir backlighting could be warm or cool). |
| **Ground-Truth Action** | **`needs_review`** |
| **Ground-Truth Rationale** | PRD §12.3 and §8.3 state: *"Where dependency data is absent, the system marks the impact as uncertain rather than assuming no impact."* Marking this shot as `retain` would risk shipping an unverified asset, while marking it `regenerate` might discard valid work. The correct ground-truth action is `needs_review` with an explicit notice of missing metadata. |
| **Confidence Level** | **Low / Uncertain (0.40)** — Deliberately engineered boundary test for missing dependency metadata. |

---

## 4. Ground-Truth Summary Matrix

| Shot ID | Title | V1 Dependencies | V2 Affected Items | Ground-Truth Action | Expected System Confidence | Primary Reason |
|:---:|:---|:---|:---|:---:|:---:|:---|
| **`SHOT-01`** | Establishing Aerial Alleyway | Style, Lighting, Rain | All changed (Style, Light, Weather) | **`regenerate`** | High (≥ 0.90) | Total aesthetic incompatibility; no reusable character layer. |
| **`SHOT-02`** | Aria Medium Portrait | Character, Style, Lighting | Style, Light changed; Character invariant | **`adapt`** | High (≥ 0.85) | Aria identity preserved; prompt & color grade adapted to warm cel-shade. |
| **`SHOT-03`** | Holographic HUD Overlay | Core Prop, Vector UI | None (Independent UI vector) | **`retain`** | High (≥ 0.90) | Self-illuminated vector asset unaffected by ambient or style shift. |
| **`SHOT-04`** | Aria Rooftop Sprint | Character, Motion, Style, Light | Style, Light changed; Motion & Character invariant | **`adapt`** | Moderate-High (≥ 0.80) | Motion layout & silhouette preserved; rendering pass adapted. |
| **`SHOT-05`** | Raindrop Splash Macro | Style, Lighting, Rain | Subject obsolete (Rain removed, Dry dusk) | **`regenerate`** | High (≥ 0.90) | Rain premise eliminated in V2; asset cannot be reused or adapted. |
| **`SHOT-06`** | Mystery Figure in Doorway | **None / Missing** | Ambiguous character & lighting links | **`needs_review`** | Low (< 0.60) | Missing dependency metadata; requires human determination. |

### Ground-Truth Distribution Breakdown
- **Total Shots:** 6
- **`retain`:** 1 (16.7%)
- **`adapt`:** 2 (33.3%)
- **`regenerate`:** 2 (33.3%)
- **`needs_review`:** 1 (16.7%)
- **Truly Affected Shots (`adapt` + `regenerate` + `needs_review`):** 5 (83.3%)
- **Truly Unaffected Shots (`retain`):** 1 (16.7%)

---

## 5. Evaluation Criteria & Benchmark Metrics (PRD §14)

Per PRD §14, all metrics must be computed against real execution results. No percentage may be reported without stating the test harness setup, sample size ($N=6$), and execution logs.

### 5.1 Change Impact Identification (Binary Affectedness)
- **Objective:** Evaluate the system's ability to distinguish affected shots from unaffected shots.
- **Classification Mapping:**
  - **Positive (Affected):** Shots classified as `adapt`, `regenerate`, or `needs_review` (Shots 01, 02, 04, 05, 06).
  - **Negative (Unaffected):** Shots classified as `retain` (Shot 03).
- **Formulas:**
  $$\text{Precision} = \frac{TP}{TP + FP}$$
  $$\text{Recall} = \frac{TP}{TP + FN}$$
  $$\text{F1-Score} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$$
- **Acceptance Threshold:**
  - $\text{Recall} = 1.00$ (100%): **Critical safety gate.** No truly affected shot may be falsely marked as `retain` ($FN = 0$).
  - $\text{Precision} \ge 0.80$.

---

### 5.2 Recommendation Quality (Multi-Class Action Accuracy)
- **Objective:** Measure the exact categorical agreement between AI recommendations and the 4-class ground truth.
- **Formula:**
  $$\text{Recommendation Accuracy} = \frac{\sum_{i=1}^{N} \mathbb{I}(\text{Predicted}_i = \text{GroundTruth}_i)}{N}$$
- **Confusion Matrix Format:**
  Rows: Ground Truth (`retain`, `adapt`, `regenerate`, `needs_review`)  
  Columns: Predicted Recommendation (`retain`, `adapt`, `regenerate`, `needs_review`)
- **Safety Penalty Rule:**
  - Severe failure: Predicting `retain` for a `regenerate` shot (silent breakage).
  - Acceptable fallback: Predicting `needs_review` when confidence is borderline is penalized less than false certainty.

---

### 5.3 Asset Reuse Rate
- **Objective:** Track the percentage of baseline assets retained or adapted rather than discarded.
- **Formula:**
  $$\text{Asset Reuse Rate} = \frac{\text{Count}(\text{Retained Assets}) + \text{Count}(\text{Adapted Assets})}{\text{Total Active Assets}} \times 100\%$$
- **Ground-Truth Target:**
  $$\text{Target} = \frac{1 (\text{SHOT-03}) + 2 (\text{SHOT-02, SHOT-04})}{6} = 50.0\%$$
- Any system recommending regeneration for all 6 shots achieves only 0% reuse and fails the core value proposition of PRD §3.

---

### 5.4 Traceability Compliance Rate
- **Objective:** Ensure every recommendation provides verifiable evidence and rationale.
- **Criteria:**
  1. Recommendation contains non-empty, human-readable `reason`.
  2. Recommendation references at least one specific requirement ID or delta attribute (`REQ-STYLE`, `REQ-LIGHT`, etc.).
  3. When confidence is low, uncertainty is explicitly flagged.
- **Threshold:** **100% compliance** across all 6 shots. A recommendation without a cited reason is an immediate test failure.

---

## 6. End-to-End Acceptance Test Procedure

This 5-phase test procedure specifies the exact verification harness for Milestone M1 through M5.

```
Phase 1: Ingestion & Workspace Setup (FR-01, FR-02, FR-03)
Phase 2: Shot Planning & Baseline V1 (FR-04, FR-05, FR-06)
Phase 3: Direction Change & Impact Analysis (FR-07, FR-08)
Phase 4: Human Review & Override Workflow (FR-09)
Phase 5: Storyboard Refresh & Immutability Verification (FR-10, FR-11)
```

### Phase 1: Ingestion & Workspace Setup
1. Create project `proj_cyberpulse_demo`. Verify stable ID assignment.
2. Ingest Brief V1 text.
3. Verify structured extraction of:
   - Summary, Art Style (`REQ-STYLE-01`), Lighting (`REQ-LIGHT-01`), Atmosphere (`REQ-ENV-01`), Character (`REQ-CHAR-01`).
4. Assert user can edit extracted fields and save as active brief.

### Phase 2: Shot Planning & Baseline V1
1. Populate 6 shots (`SHOT-01` to `SHOT-06`) with sequence orders 1 through 6.
2. Attach prompt strings and asset references per §3 of this contract.
3. Save active direction as `dir_v1_cinematic` (Version 1).
4. Verify each shot records `current_version_id` pointing to `ShotVersion` v1 tied to `dir_v1_cinematic`.
5. Verify Shot Board renders all 6 shots in status `approved`.

### Phase 3: Direction Change & Impact Analysis
1. Submit proposed Direction V2 (`dir_v2_stylized`).
2. Verify system generates a diff summary highlighting:
   - Changed: Style (Photoreal → Anime Cel-shaded), Lighting (Cool Blue → Warm Amber), Atmosphere (Rain → Clear Dusk).
   - Invariant: Character Aria identity (`REQ-CHAR-01`), Data Core (`REQ-PROP-01`).
3. Execute change impact analysis on all 6 shots.
4. Capture system outputs:
   - Recommended actions (`retain`, `adapt`, `regenerate`, `needs_review`).
   - Explanation strings and requirement citations.
   - Confidence scores.
5. Evaluate outputs against §4 Ground-Truth Summary Matrix.

### Phase 4: Human Review & Override Workflow
1. For Shots 01, 02, 03, 04, 05: Simulate user **accepting** AI recommendations.
2. For Shot 06 (`needs_review`): Simulate user performing manual triage:
   - User inspects unlinked asset, determines the figure is a background guard requiring warm cel-shaded silhouette, and **overrides** action to `adapt`.
   - Record human decision: `action = "adapt"`, `actor = "human"`, `reason = "Manual triage: Asset is background guard; adapt lighting to warm dusk"`.
3. Assert system stores AI suggestion (`needs_review`) and human approval (`adapt`) as distinct fields (FR-09).

### Phase 5: Storyboard Refresh & Immutability Verification
1. Commit Direction V2 as active.
2. Generate revised shot versions for affected shots:
   - `SHOT-01`: ShotVersion v2 (status: `pending_generation`).
   - `SHOT-02`: ShotVersion v2 (status: `pending_adaptation`).
   - `SHOT-03`: ShotVersion v1 retained (status: `approved`).
   - `SHOT-04`: ShotVersion v2 (status: `pending_adaptation`).
   - `SHOT-05`: ShotVersion v2 (status: `pending_generation`).
   - `SHOT-06`: ShotVersion v2 (status: `pending_adaptation`).
3. **Immutability Check:** Query database/storage to verify:
   - `dir_v1_cinematic` record is unmodified.
   - All ShotVersion v1 records for Shots 01–06 remain intact and queryable.
4. Storyboard View verification:
   - Displays latest version per shot.
   - Outstanding action badges are visible (4 pending adaptation/generation, 1 approved, 0 unreviewed).
   - Project overall status remains `in_progress` (never prematurely marked complete).

---

## 7. Edge Cases & Safety Gates

1. **Missing Dependency Data (Boundary Gate):**
   - If a shot lacks dependency records, the system **must** return `needs_review`.
   - **FAIL condition:** Any system predicting `retain` on an unlinked shot without dependencies.

2. **No False PASS Principle:**
   - Test suites must not mock pass conditions. If an endpoint returns unverified text or dummy classifications, tests must fail.

3. **Client-Side Security & Data Boundaries:**
   - All sample assets must use synthetic or public placeholder URIs.
   - No API keys or internal database paths may be leaked in recommendation explanation payloads.

4. **Schema Validation Gate:**
   - All LLM recommendation outputs must parse cleanly against strict Pydantic/JSON Schema models:
     `action: Literal["retain", "adapt", "regenerate", "needs_review"]`,
     `reason: str`,
     `confidence: float (0.0 to 1.0)`,
     `affected_requirements: List[str]`.
