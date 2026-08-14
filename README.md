# The Fall of Robespierre: Stylometric Analysis — Re-Verification Guide

A step-by-step guide reflecting all pitfalls and improvements discovered during the previous round of analysis, intended for re-running the study.

rolling_delta.html - analysis app
extract_dialogue.html - for preparing texts
---

## 0. Before You Begin: An Unresolved Premise Raised During Analysis

**It has been suggested that Acts II and III themselves may contain later additions by Coleridge.** Since this bears directly on the foundational assumption — treating Acts II and III as a "pure Southey baseline" — clarify your position on one of the following before starting:

- (a) Set this possibility aside and continue treating Acts II and III as the Southey baseline (appropriate if the goal is to reproduce results under the same conditions as before).
- (b) First investigate bibliographic evidence (textual variants, correspondence, etc.), and exclude from the reference/comparison set any passages in Acts II/III suspected of being Coleridge's additions.

Which stance you take affects part of the procedure below (in particular, whether Step 2 may treat Acts II/III as a "known Southey baseline").

---

## 1. Corpus Preparation

### 1.1 Standardize Word-Count Conventions

- Decide, and apply consistently across **both the test text and all reference texts**, whether to include stage directions (e.g., "[Exit.]") and speaker labels in word counts. In the previous round, CoSou_Act1–3 (which retained stage directions) and other "dialogue_only" reference files (with stage directions stripped) used inconsistent conventions. If you do not standardize, note this explicitly in the report and discount results accordingly.
- Always verify word counts by actual measurement (`wc -w` or equivalent). Discrepancies of a few percent between claimed and measured counts are common (this occurred with *Osorio* and *Wat Tyler*, among others).

### 1.2 Reference Text Selection Criteria

- **Temporal proximity**: The farther a reference text is from the target period (1794), the more noise is introduced by the author's own stylistic drift and by broader diachronic language change. Prioritize 1790s works wherever possible.
- **Genre proximity**: Favor verse drama (dialogue-based). Epic poetry or monologue-heavy verse (narrative-dominant) has a structurally different distribution of third-person pronouns and conjunctions, which can become a larger noise source than genuine authorial difference.
- **Check for contamination**: Verify in advance whether candidate texts contain a collaborator's hand (two concrete examples arose in this study: Coleridge's contribution to *Joan of Arc*, and the mislabeling of Acts II/III of *The Fall of Robespierre* itself).
- **Balance of scale**: Avoid large disparities in total word count between author classes. Where disparities are large (in this study, Baillie's three works totaled ~60,000 words versus Southey's ~9,800), the GI method's `windowImposters` and `equalSampleSize` options can mitigate this, but the underlying imbalance persists and should be kept in mind when interpreting results.

### 1.3 Extracting Analysis Units

- When segmenting at split points detected by Rolling Delta, **always verify boundaries mechanically by word count** (if files were split manually, confirm via Python that they reconstruct the original file exactly).
- Where fragments become very short (under 300 words), treat any results derived from them as indicative only.

---

## 2. Basic Tool Configuration

### 2.1 Role of Each Drop Zone

| Item | Role |
|---|---|
| dz-test | Target of Rolling Delta (Step 1). Exactly one file. |
| dz-ref | Reference text set. **Load all files once, then toggle individual files on/off via checkboxes** — no need to re-upload when narrowing the reference set at different stages. |
| dz-acts | Analysis units for Step 2 (dendrogram + PCA). Multiple files allowed. |
| dz-ctrl | Supplementary comparison set used only in Step 2. Does not affect Step 1 or Step 3. |

### 2.1b Which Options Affect Which Steps (Common Point of Confusion)

| Option | Scope of Effect |
|---|---|
| Window size (W) / Step size (S) | Step 1 (Rolling Delta itself, and building reference centroids); Step 3's GI-method `windowImposters` (expanding the reference-file imposter pool). **Has no effect whatsoever on Step 2** (classic Delta, dendrogram, PCA) |
| MFW (`opt-mfw`) | A single global setting shared by Step 1, Step 2, and Step 3's PHASE 1. Always confirm and record its value when moving between stages |
| Consensus MFW min/max/step | Exclusive to Step 0.5 (Bootstrap Consensus Tree) |

Step 2 builds a single relative-frequency vector from each document's entire token set (no windowing), so changing the W/S fields has no effect on Step 2's output.

### 2.2 Pre-Execution Checklist

- Tokenizer setting: for English text, select "word (whitespace-delimited language / regex)."
- Use the "Preview W/S/MFW contents" button to confirm stage directions are being tokenized as intended (consistent with the convention chosen in 1.1) before running the full analysis.

---

## 3. Analysis Pipeline (Recommended Order, with Files and Options Specified)

> **Execution-status legend**: Step 0.5, Step 1 (Author-Attribution Phase 1, whole-play), and Step 2(a) (act-level) are confirmed by actual measurement (2025 re-verification). Note that Baillie's two reference works are larger than the Coleridge/Southey reference sets, and a corpus-size artifact was confirmed in Step 1/Author-Attribution Phase 2 (see 4.10–4.11) — treat with caution. Step 2(a) additionally revealed a more fundamental phenomenon independent of scale (see 4.12). Step 1 (Author-Attribution Phase 2), Step 2(b) (seven-fragment version), and Step 3 are all based on confirmed empirical data.

### Step 0.5: Bootstrap Consensus Tree (confirmed: W=500, S=50, MFW=50–200 in steps of 25)

#### Data Used

| Item | Setting |
|---|---|
| dz-test | CoSou_Robespierre.txt (whole play combined, 5,810 words) |
| dz-ref | All 9 references ON: Coleridge_Osorio.txt / Coleridge_TriumphofLoyalty.txt / Southey_Wat_Tyler.txt / Southey_BotanyBayEclogue_dialogue_only.txt / Wordsworth_TheBorderers_dialogue_only.txt / HannahMore_Percy_dialogue_only.txt / JoannnaBaillie_Basil_1798_dialogue_only.txt / JoannaBaillie_TheTryal_1798_dialogue_only.txt / JoannaBaillie_DeMonfort_1798_dialogue_only.txt |
| dz-acts | Act-level (CoSou_Act1/2/3.txt) or segment-level (CoSou_分割1〜4_2.txt + Act2/3) — check clustering behavior for both |
| Consensus MFW min | 50 (default) |
| Consensus MFW max | 200 (default) |
| Consensus MFW step | 25 (default) |
| Consensus threshold | 0.5 (default) |

#### Mechanism (Internal Logic)

1. For the specified range of MFW values (default: 50, 75, 100, 125, 150, 175, 200 — **7 points**), an independent UPGMA tree (an ordinary Burrows' Delta dendrogram) is built at each MFW value. These are "replicate trees" — 7 in total under default settings.
2. Every bipartition (clade — a grouping of documents into a subtree) appearing in each replicate tree is recorded.
3. Across all 7 replicate trees, the number of trees in which each identical bipartition appears is counted.
4. Only bipartitions whose frequency (count ÷ total replicate trees) meets or exceeds the "consensus threshold" (default 0.5) are retained, producing a single final consensus tree.

This automates, across MFW=50–200, the kind of "manual cross-MFW consistency check" that was previously performed by hand.

#### How to Read It: Meaning of the Percentages

The red percentage shown at each merge point indicates **"the proportion of the seven MFW-based replicate trees in which this grouping was reproduced"** (the same concept as a bootstrap support value in classical statistics).

| Support Level | Interpretation |
|---|---|
| 90–100% | The grouping holds regardless of MFW choice — extremely robust |
| 60–89% | Reproduced in most MFW settings but breaks down in some — moderate reliability |
| 50–59% (near threshold) | Reproduced in only about half of MFW settings — MFW-dependent and weak; avoid using as grounds for a conclusion |
| (below threshold, not shown) | No majority support at any MFW; the tree renders these documents as an unresolved polytomy instead |

#### How to Read It: Meaning of Height (Vertical Position) — Opposite of a Standard Dendrogram

**This is the point most easily misread.** In Step 2's standard dendrogram, merge height represents Delta distance (closer pairs merge lower). **In Step 0.5, height instead represents the support percentage directly** (the tool's implementation explicitly notes: "height is schematically proportional to support 0–100%, not a Delta distance").

- Merges near the top (near 100%) indicate robust, high-support groupings.
- Merges near the bottom (near the 50% threshold) indicate weak, borderline groupings.

Do not carry over the "lower = closer" intuition from an ordinary dendrogram — doing so inverts the meaning of the vertical axis.

#### Analysis Procedure (for this corpus)

1. First **confirm the robustness of known reference pairs**: check whether Osorio–Triumph (both Coleridge) and Act II–Act III (both Southey, known common authorship) merge with high support (ideally 90%+). If not, reconsider the MFW range or consensus threshold.
2. Check **what support percentage the fragment under investigation (Segments 1–4, or Act I) receives when joining a given clade**.
3. If support is 90% or higher, this can be taken as confirmation that the single-MFW result seen in Step 2 (e.g., MFW=30) is robust and not an artifact of MFW choice.
4. If support is only around 50–60%, or if the relevant bipartition does not appear in the tree at all (remaining an unresolved polytomy), the single-MFW Step 2 result should be treated as MFW-dependent and unreliable — use only as reference information, not as a basis for firm conclusions.
5. Adjust the MFW min/max range as needed (e.g., if short fragments are included, MFW=200 may become unstable; try lowering the maximum to around 100 and check whether results change).

#### Measured Results (2025 re-verification, W=500, S=50, MFW=50–200 in steps of 25)

- **CoSou_Act1, Act2, and Act3 clustered together before joining any external reference** (Act1–Act2: 100%; +Act3: 86%). This means "Acts I, II, and III resemble each other more than any of them resembles any external reference author" — an important finding suggesting that **shared subject matter/topic may dominate over authorial discrimination**.
- Among external references, Southey (*Wat Tyler*, *Botany-Bay Eclogues*) and Baillie's *The Tryal* clustered at 71%, while Coleridge (*Osorio*, *Triumph*) and Wordsworth's *Borderers* also clustered at 71% — **no clean two-group separation into Coleridge vs. Southey was observed**.
- Overall support was in the 71–86% range (excluding the Act1-2-3 group), exceeding the 50% threshold but falling short of the 90%+ "highly robust" level. **Clustering with external references under this configuration (MFW=50–200) should be regarded as only moderately reliable.**

### Step 1 (Author-Attribution Phase 1, confirmed by measurement): Whole-Text Rolling Delta

| Item | Setting |
|---|---|
| dz-test | CoSou_Robespierre.txt (5,810 words) |
| dz-ref | Same 9 files as Step 0.5, all ON |
| dz-acts | CoSou_Act1.txt / CoSou_Act2.txt / CoSou_Act3.txt (submitted together, for use by Step 2) |
| Window size (W) | 500 words |
| Step size (S) | 50 words |
| MFW | 100 (default) |

#### Mechanism (How This Differs from Step 2's Classic Delta)

Step 1's Y-axis (Delta score) is computed on a different basis than Step 2 (whole-document standardization):

1. Each reference text is itself internally windowed using the same W/S as the main scan (e.g., *Osorio* at 16,090 words yields roughly 630 windows; *Triumph* at 3,156 words yields roughly 115).
2. From the window-to-window variation **within that reference text**, an independent μ (mean) and σ (standard deviation) are calculated for that reference alone.
3. For each window of the test text, the distance is computed as: "how many standard deviations, by that particular reference's own typical internal variability, does this window fall from that reference's centroid?"

This eliminates the extreme instability of treating "one document as one sample," but leaves a separate asymmetry: **longer reference texts produce more windows and hence more stable (lower-noise) μ/σ estimates, potentially making them "win" the Delta comparison independent of true stylistic similarity** (this became apparent in the measured results below).

#### What to Look For in Author-Attribution Phase 1

1. Whether the two Coleridge lines cross over to the two Southey lines around the known boundaries (word 2068 = Act I/II; word 4182 = Act II/III).
2. Whether the other five references (Wordsworth, Hannah More, and the three Baillie works) remain consistently distant (supporting their reasonable exclusion as candidates).
3. Whether crossings occur exactly at the boundaries (2068, 4182) or earlier (an earlier crossing would provide a clue linking to an internal Act I split point).

#### Measured Results (2025 re-verification)

CSV analysis revealed **a more complicated picture than initially assumed**.

| Word Range | Actual Nearest Reference |
|---|---|
| 250–500, 600–900, 2400–2450 | Coleridge_Osorio |
| **900–2000** (largest span within Act I, ~1,100 words) | **Baillie_Basil** |
| 2000–2100 | Hannah More_Percy |
| 2100–2200, 4500–5100, 5500–5600 | Southey_Wat Tyler |
| Most of 2450–5450 | **Baillie_Basil / Baillie_De Monfort alternating** |
| 5600–5987 | Southey_BotanyBay |

**Critical problem**: Even within the known-Southey acts (II and III), the most frequent nearest reference was not Southey's own works but Baillie's *Basil* and *De Monfort*. Southey is nearest only within the narrow ranges 2100–2200, 4500–5100, and 5500–5987. Within Act I itself, Baillie_Basil is nearest across the broad range 900–2000 — so it is not accurate to say "the other five references remain consistently distant."

**Interpretation**: Baillie's two works (~39,477 words combined) are far larger than Coleridge's (19,246) and Southey's (9,805) reference sets, and the "longer references produce more stable, easier-to-win estimates" asymmetry described above is strongly suspected of driving this result. This is likely more a corpus-size artifact than a genuine stylistic proximity.

**Consistency with Step 0.5's Bootstrap Consensus Tree**: The finding that Acts I, II, and III cluster together ahead of any external reference (in Step 0.5), and this finding that Baillie "wins" purely through scale, are two separate lines of evidence pointing to the same underlying concern — **factors other than authorship (topic, corpus scale) can dominate the result.**

**Recommended follow-up**: Do not use the full 9-reference Step 1 as direct evidence of authorship, given the reference corpus's scale imbalance. Either re-run with the narrowed 4-reference set (Osorio, Triumph, Wat Tyler, Botany-Bay — whose scale differences are comparatively smaller), or, if using all 9 references, consider trimming Baillie's two works to a matched word count.

### Step 1 (Author-Attribution Phase 2, confirmed by measurement): Act I Split-Point Detection

| Item | Setting |
|---|---|
| dz-test | CoSou_Act1.txt (2,068 words) |
| dz-ref | 6 files ON: Coleridge_Osorio.txt / Coleridge_TriumphofLoyalty.txt / Southey_Wat_Tyler.txt / Southey_BotanyBayEclogue_dialogue_only.txt / Wordsworth_TheBorderers_dialogue_only.txt / JoannnaBaillie_Basil_1798_dialogue_only.txt (HannahMore_Percy, Tryal, DeMonfort are OFF) |
| Window size (W) | **300 words** |
| Step size (S) | **25 words** |
| MFW | 100 (default; also re-run at 30 to check reproducibility of the crossing points) |
| Output | Split Point 1 (word 850–1150, center 1000), Split Point 2 (word 1175–1475, center 1325), Split Point 3 (word 1700–2000, center 1850) |

#### Critical Verification: False Nearest-Neighbor from Baillie_Basil Contamination

Mechanical CSV analysis with the above 6 references (including Baillie_Basil) revealed that **at all three split points, the actual nearest reference was not Coleridge or Southey but Baillie_Basil** — the same corpus-scale artifact identified in Author-Attribution Phase 1 (*Basil* = 21,806 words, noticeably larger than the other references).

| Split Point | Nearest with Baillie Included | Runner-up |
|---|---|---|
| Word 1000 | Baillie_Basil (0.827) | Wat Tyler (0.851) |
| Word 1325 | Baillie_Basil (0.817) | Osorio (0.964) |
| Word 1850 | Baillie_Basil (0.847) | Wat Tyler (0.928) |

The chart's "Southey 1 / Coleridge / Southey 2" labels reflect crossings restricted to the four Coleridge/Southey lines, not the true nearest neighbor across all six lines.

#### Verification: Re-Running with Baillie_Basil Excluded

After removing Baillie_Basil, **the nearest reference reverted to plausible alternation between Coleridge and Southey**.

| Split Point | 1st | 2nd | Margin |
|---|---|---|---|
| Word 1000 ("Southey 1") | Wat Tyler 0.826 (Southey) | Botany-Bay 0.863 (Southey) | Clear gap |
| Word 1325 ("Coleridge") | Wat Tyler 0.924 (Southey) | Osorio 0.936 (Coleridge) | **Narrow (0.012)** |
| Word 1850 ("Southey 2") | Wat Tyler 0.921 (Southey) | Osorio 0.923 (Coleridge) | **Essentially tied (0.002)** |

**Conclusion**: Excluding Baillie confirmed that the Coleridge/Southey crossing narrative underlying Split Points 1, 2, and 3 is substantively real, not an artifact. This provides good corroboration for the validity of the Step 3 detailed verification (which examined Split Points 1 and 3 via GI and PHASE 2).

However, the crossings at words 1325 and 1850 are narrow — word 1850 in particular is essentially a coin flip. This narrowness is itself consistent with the non-trivial behaviors observed in Step 3's PHASE 2 (MFW sensitivity analysis) — e.g., "diverging uniformly" or "converging selectively toward only one text" — rather than a clean, decisive signal.

**Lesson**: When running Rolling Delta with a full reference set (9 or 6 references), **always compare the split points/crossings both with and without large-scale references (Baillie, Wordsworth, etc.) included**.

### Step 2: Overall Structural Overview (Two Configurations)

#### Important Correction: Step 2 Is Unaffected by Window Size / Step Size

Step 2 (classic Delta, dendrogram, PCA) builds a **single relative-frequency vector from each document's entire token set** — no windowing occurs, unlike Step 1's Rolling Delta or the GI method's windowImposters (Step 3). Consequently:

- The window-size (W) / step-size (S) fields have **no effect whatsoever** on Step 2's output (any values entered are simply ignored).
- Step 2's output is determined **solely by MFW**.
- Step 1, Step 3 (GI windowImposters), and the MFW field (`opt-mfw`) are shared; confirm and record this value before running Step 2.

**(a) Act-Level (part of Author-Attribution Phase 1, confirmed by measurement)**

| Item | Setting |
|---|---|
| dz-test | **CoSou_Robespierre.txt (whole play, 5,810 words)**. Because this uses the same session/input configuration as Author-Attribution Phase 1's Step 1, this field is meaningful for that Step 1's whole-play Rolling Delta (though it does not directly affect Step 2, which prioritizes dz-acts, this field should still hold a substantively meaningful value rather than a placeholder). |
| dz-acts | CoSou_Act1.txt / CoSou_Act2.txt / CoSou_Act3.txt (submit all three together in a single run — do not run each file separately; submitting them together enables direct pairwise comparison among the acts and a shared dendrogram) |
| dz-ref | All 9 references ON (same as Step 0.5 and Step 1). Also run with Baillie/Wordsworth/HannahMore excluded (4-reference version) for comparison |
| MFW | Use 100 as the initial value, but **also re-run at MFW=30** to check whether the Act I–II–III mutual clustering (observed in Step 0.5) reproduces independent of MFW |

#### Measured Results: Acts Are Consistently Closer to Each Other Than to Any External Reference (A More Fundamental Phenomenon, Distinct From the Corpus-Scale Artifact)

CSV analysis showed that, **for all three acts, the nearest neighbor was always another act — never an external reference**.

| | 9-reference set: Nearest | 4-reference set (Baillie et al. excluded): Nearest |
|---|---|---|
| Act 1 | Act3 (1.060) < Osorio (1.080) | Act2 (1.164) < Osorio (1.182) |
| Act 2 | Act3 (1.042) < Osorio (1.171) | Act1 (1.164) < Triumph (1.244) |
| Act 3 | Act2 (1.042) < Botany-Bay (1.245) | Act2 (1.175) < Triumph (1.421) |

**Important implication**: The Baillie issue confirmed in Author-Attribution Phase 2 was an artifact that "reverts to normal once Baillie is excluded." **This "acts cluster together" phenomenon does not disappear even after excluding Baillie et al.** — it is not a scale artifact but a **more fundamental phenomenon: the three acts of the same play share subject matter, characters, and vocabulary.**

Note that "Acts II and III merge first" (as an ordering claim) holds only in the 9-reference version (Act2–Act3 = 1.0416, the smallest pair). In the 4-reference version, **Act1–Act2 (1.1635) is actually marginally the smallest**, with the three act-pair distances confined to a narrow band (0.011–0.012 apart) — a razor-thin margin. The broader conclusion ("the three acts cluster together ahead of any external reference") is robust; do not over-interpret the finer detail of merge order.

**What this means**: Step 2(a)'s dendrogram topology (which act merges with which first) is dominated by topic effects and should not be used as direct authorship evidence. Instead, prioritize **the ranked distance from each act to each individual reference** — the same approach consistently used in Step 1 and Step 3.

**(b) Act I Segments + Acts II & III (seven-segment comparison, confirmed by measurement)**

| Item | Setting |
|---|---|
| dz-acts | CoSou_分割1.txt (792 words) / CoSou_分割2.txt (299 words) / CoSou_分割3.txt (488 words) / Cosou_分割4_1.txt (323 words) / Cosou_分割4_2.txt (166 words) / CoSou_Act2.txt (2,114 words) / CoSou_Act3.txt (1,628 words) |
| dz-ref | **Only 4 ON**: Coleridge_Osorio.txt / Coleridge_TriumphofLoyalty.txt / Southey_Wat_Tyler.txt / Southey_BotanyBayEclogue_dialogue_only.txt (Borderers, Percy, and the three Baillie works are OFF) |
| dz-test | CoSou_Act1.txt (irrelevant to Step 2's output since dz-acts takes priority, but required as a field) |
| MFW | **30** |

#### Verification of Measured Results (from the distance matrix)

| Segment | Verification |
|---|---|
| Segment 1 (792 words) | ✓ Confirmed: nearest is Triumph (0.836), then Osorio (0.902). Clusters cleanly with Coleridge |
| Segment 3 (488 words) | ✓ Confirmed: nearest is Triumph (0.865), consistent whether or not Acts II/III are included. A stable lean toward Coleridge |
| **Segment 2 (299 words)** | **△ Requires correction**: Among references alone, the nearest is Triumph (1.321); but once Acts II/III are included, **Act III (0.964–1.042) is substantially closer than Triumph**. Claiming that "the single-reference nearest agrees with the whole-structure position" is inaccurate — proximity to Act III (known Southey) dominates. This correction actually reinforces the "ABA" pattern narrative, since Segment 2 was already understood to lean "Southey-ward" |
| Segment 4-2 (166 words, political-return scene) | ✓ Confirmed: best link (to Act3, 1.257) is markedly weaker than genuine cluster-forming links elsewhere (0.546–0.920) — uniformly distant from everything. Consistent with isolation |
| Segment 4-1 (323 words, private scene) | △ Conditionally confirmed: best link (to Act3, 0.908) matches the strength of other genuine cluster-forming links, and in the PCA plot it sits near Segments 2/3, supporting the view that it is "not isolated." **However, a dendrogram generated from the same underlying data classified it as an isolated branch in a separate session** — the assessment differs between PCA and dendrogram (see 4.13) |

**Lesson**: PCA scatter plots and dendrograms can present different pictures from the same distance matrix (due to differences in dimensionality reduction and clustering linkage method). For fragments where the assessment differs (like Segment 4-1), always **cross-check both visualizations and treat the raw distance matrix as the final authority**.

### Step 3: Detailed Split-Point Comparison (Author-Attribution Phase 3 + Influence Analysis, confirmed by measurement)

| Item | Setting |
|---|---|
| dz-ref | Same 4 references as Step 2(b) (Borderers and Basil removed from the 6 used in Author-Attribution Phase 2) |
| Split Point 1 | Position 1000 (tokens 850–1150) |
| Split Point 2 | Position 1350 (tokens ~1175–1475; nearly the same location as the "Coleridge" label in Author-Attribution Phase 2's chart) |
| Split Point 3 | Position 1850 (tokens 1700–2000) |
| MFW (PHASE 1 baseline / ranking table) | 100 as baseline, also re-run at 30 for comparison (both completed for Split Point 1) |
| PHASE 2 (MFW sensitivity) | Automatic sweep at 30/50/100/200/261 (fixed in the tool's implementation; 261 rather than 300 due to the ceiling on vocabulary shared across the four active references) |

**GI Method Options (within Step 3)**

| Option | Setting | Note |
|---|---|---|
| Iterations | 200 | |
| Imposter sampling ratio | OFF: 1.0 / ON: default (~0.5) | OFF+1.0 was used as a control configuration for verification purposes; the production run uses ON |
| windowImposters | **ON** | The OFF configuration (imposterRatio=1.0) alone was found to eliminate diversity and be invalid |
| Window/step size (for pool expansion) | 300 words / 25 words (inherited from Author-Attribution Phase 2's setting; a change to roughly 600/400 words would be preferable but was not implemented) | The imposter count of 1,170 derives from this setting |
| Equal sample size | ON (default) | |

#### Qualitative Comparison Across the Three Split Points (Summary of Measured Results)

Comparing the PHASE 2 (MFW sensitivity) Before→After changes across split points revealed **three qualitatively distinct patterns underlying what superficially all look like "approach toward Southey."**

| Split Point | Behavior at MFW=30–50 | Interpretation |
|---|---|---|
| **Split Point 1** (position 1000) | **Both** Southey works (Wat Tyler and Botany-Bay) move closer (Wat Tyler +0.1185, Botany-Bay +0.0616) | A broad lean toward Southey generally. Harder to explain away as a coincidence involving a single text — of the three split points, this has the strongest evidentiary weight |
| **Split Point 2** (position 1350) | **Only Botany-Bay** converges consistently across every MFW (−0.327 at MFW=30, the largest magnitude of change). Wat Tyler actually moves farther away | Suspected to be a thematic coincidence specific to *Botany-Bay Eclogues* (loss of home/family), not a general Southey effect. PHASE 3 also shows the character-cast change ("Couthon" disappearing, "Adelaide" appearing), consistent with this being a product of scene transition |
| **Split Point 3** (position 1850) | **All four references move substantially farther away** (Osorio −0.727, Triumph −0.530, etc. — the sign convention here indicates divergence despite appearing negative) | Not an approach toward Southey, but the After segment (374 tokens, short) becoming isolated from all authors considered. Consistent with the isolation of Segment 4-2 confirmed in Step 2(b), and most plausibly reflects instability from a short fragment |

**Conclusion**: Of the three split points, **only Split Point 1 provides evidence with genuinely convincing support for a lean toward Southey**. Split Point 2 more plausibly reflects a thematic coincidence specific to *Botany-Bay Eclogues*; Split Point 3 more plausibly reflects isolation due to a short fragment. **Do not use the results from Split Points 2 and 3 to further reinforce the "Southey influence" hypothesis.**

---

## 4. Mandatory Checkpoints at Each Stage (Pitfalls Identified in the Previous Round)

### 4.1 Timing of Reference-Text Narrowing

When changing the reference configuration (which files are ON/OFF) between Step 1 (split-point search) and Step 3 (detailed verification), **always confirm the checkbox states immediately before running Step 3**. It is easy to accidentally proceed with a configuration left over from an earlier stage.

### 4.2 GI Method Settings

| Option | Recommendation | Reason |
|---|---|---|
| Imposter sampling ratio | 0.3–0.5 when `windowImposters` is used; 1.0 when not | With a small pool (~4 files), leaving the ratio at 0.5 produces unstable results |
| `windowImposters` | **Always ON** | It was found that with OFF and a ratio of 1.0, randomness on the imposter side is eliminated, and the method effectively fails to function as intended |
| Window/step size (for pool expansion) | Do not simply carry over the fine-grained values used in Step 1 (e.g., 300/25). Set separately to roughly 600/400 | Carrying over fine values causes adjacent windows to overlap by more than 90%, producing only pseudo-diversity |
| Equal sample size | ON (default) | Essential for absorbing the large disparities in reference-text length (from a few thousand to over ten thousand words) |
| Iterations | 200 or more | Variance is high with small texts and small imposter pools |

### 4.3 Order for Reading Results

Do not draw conclusions from a single indicator alone. Check in the following order:

1. PHASE 1 (basic Delta) → Does change appear to be present?
2. Reference ranking → In which direction is the change?
3. GI method → Is the change statistically robust (reproducible across settings/repeated runs)?
4. PHASE 2 → Does it hold even at MFW=30–50, or does it appear only at high MFW?
5. PHASE 3–5 → What specifically changed?

### 4.4 Mandatory Check When Reviewing PHASE 3 (Top-50 Vocabulary Change)

**Always check whether character names or scene-specific forms of address appear near the top of the changed-word list.** In the previous round, changes such as "Couthon" → "Adelaide" and increases in "Barrere" repeatedly appeared near the top — reflecting **a change in the cast of characters present, not authorship**. When such words are present, prioritize the explanation "scene transition / character-cast change" over "stylistic influence."

### 4.5 Divergence Between Burrows' Delta and Eder's Delta

When the two methods diverge at high MFW (200+), **prefer Eder's Delta** (it is less likely to be misled by unstable low-frequency-word estimates).

### 4.6 Caution Regarding the Tool's Auto-Generated Summary Text

Auto-generated summary lines (e.g., "Nearest reference text to the After segment: XX") can sometimes **contradict the actual numeric table** (this occurred in practice). Treat such summaries as reference only, and always verify against the raw numbers.

### 4.7 Disentangling Narrative Convergence (Genre Effects) from Genuine Stylistic Influence

If the latter part of Act I moves toward a political climax (handing off to the next act), **lexical proximity may simply reflect convergence toward the same climactic content** rather than any authorial signal. To distinguish this from genuine influence:

- Run a control comparison at a different split point that stays within the same scene genre (political content), as Split Point 1 did in this study.
- Determine whether the change is explicable by political-vocabulary shifts, versus by function-word changes (pronouns, second-person forms, etc.).

### 4.8 Handling Short Fragments (Under 300 Words)

Results at MFW=100 or higher carry low statistical reliability. Center interpretation on MFW=30–50, and do not treat such results as the primary basis for conclusions.

### 4.9 Notes on Interpreting Step 0.5 (Bootstrap Consensus Tree)

- Merge-point **height represents support percentage, not distance** — the opposite convention from Step 2's standard dendrogram (height = distance). Do not confuse the two.
- Bipartitions below the threshold do not appear in the tree and are instead shown as an unresolved polytomy; if you find yourself wondering "why are several documents merging at a single point," this means "no robust two-way split was found."
- Do not use a 50–60% support merge as grounds for corroborating a single-MFW result seen in Step 2.

### 4.10 Step 1: Reference-Corpus Scale Asymmetry (Mandatory Check When Using the Full 9-Reference Set)

Because Step 1 builds μ/σ from internal windowing of each reference text, **longer references produce more windows, more stable μ/σ estimates, and thus become easier to "win" the Delta comparison**. Measured results confirmed that, with the full 9-reference set (including all three Baillie works), Baillie (with ~39,477 words combined, roughly four times Southey's ~9,805) became the most frequent nearest reference even within the known-Southey acts (II and III) — a suspected scale artifact. Do not use the full 9-reference Step 1 as direct evidence for authorship. Where possible, balance total word counts across author classes, or prioritize results from the narrowed 4-reference set (whose scale differences are smaller).

### 4.11 Always Check Both With and Without Large-Scale References (Baillie, etc.) Included

The same phenomenon was confirmed in Author-Attribution Phase 2 (Act I alone, 6 references): with Baillie_Basil (21,806 words, noticeably larger than the other references) included, the actual nearest reference at all three detected split points was Baillie_Basil — not Coleridge or Southey. The chart's "Southey/Coleridge" labels reflected crossings restricted to the four Coleridge/Southey lines only, not the true nearest neighbor across all six lines.

**Standard procedure**: When Rolling Delta is run with a configuration including a large-scale reference (as a rule of thumb, one containing twice the word count of the other references or more), **always verify whether the same split points/crossings reproduce after excluding that reference**. If the same crossing location reproduces after exclusion, the split point can be judged as substantively real rather than an artifact. In this study's case, the Split Point 1/2/3 crossings largely persisted after excluding Baillie, but two of them (words 1325 and 1850) were narrow (0.002–0.012), requiring caution in how strongly the conclusion is stated.

### 4.12 Step 2(a): Acts Consistently Closer to Each Other Than to External References (A Distinct Phenomenon from the Scale Artifact)

Measured results from Step 2(a) (both the 9-reference and 4-reference versions) showed that **for all three acts, the nearest neighbor was always another act, never an external reference**. This phenomenon does not disappear even after excluding Baillie et al., indicating it is **a more fundamental phenomenon — the three acts of the same play sharing subject matter, characters, and vocabulary — distinct from the scale-driven artifacts in 4.10/4.11**.

- Do not use Step 2(a)'s dendrogram clustering topology (which act merges with which first) as direct evidence of authorship. It was confirmed that merge order itself can flip depending on the reference configuration (9 vs. 4 references) — a razor-thin margin.
- Instead, examine **the ranked distance from each act to each individual reference** (whether it leans toward Coleridge-side or Southey-side references) — the same approach consistently used in Step 1 and Step 3.

### 4.13 Handling Fragments Where PCA and Dendrogram Assessments Diverge

Even from the same distance matrix, PCA (2D projection) and a dendrogram (hierarchical clustering) can present different pictures. In measured results, Segment 4-1 (the private-scene fragment) appeared, via PCA, to sit "near Segments 2/3, not isolated" — but a separately generated dendrogram classified it as "an isolated branch." **The same raw data produced opposite impressions.**

**Response**:
1. For such fragments, check whether the fragment's best link (minimum distance) is on par with other genuine cluster-forming links (e.g., a known same-author reference pair, or known same-author acts).
2. Note that even with a strong best link, if the fragment's maximum distance (to its most dissimilar counterpart) is disproportionately large, certain dendrogram linkage methods (e.g., complete linkage) may still render it as an isolated branch.
3. Do not rely on PCA or the dendrogram alone — **always cross-check both, and ultimately prioritize the raw numeric distances**.

---

## 5. Overall Flow (Summary)

```
0. Clarify your position on the Acts II/III authorship premise
        |
1. Corpus preparation (standardize word counts, select references, check scale balance)
        |
2. Step 0.5: Bootstrap Consensus Tree (multi-MFW UPGMA majority-vote check via 7 replicate trees;
   remember that height = support %, not distance)
        |
3. Step 1: Whole-text Rolling Delta (identify candidate split points)
        |
4. Step 2: Act-level and segment-level dendrogram + PCA (grasp overall structure)
        |
5. Step 3: Detailed verification at each split point
   (PHASE1 -> reference ranking -> GI -> PHASE2 -> PHASE3-5)
        |
6. Cross-compare results across split points (disentangle genre effects from stylistic influence)
        |
7. Overall conclusion (state clearly whether multiple lines of evidence converge or conflict)
```

---

## 6. Lessons from the Overall Analysis (Summary)

- Even when a "closer/farther" statistical result appears, **whether the cause is authorship, genre, character-cast composition, or narrative convergence cannot be determined without additional content analysis**.
- If the reference corpus lacks a comparandum in the same genre as the target passage (e.g., no Coleridge text with a comparable private/domestic monologue), a result showing "uniformly distant from every author" may reflect **a gap in the reference corpus** rather than genuine stylistic distinctiveness.
- Bear in mind at all times that the premise itself (the established authorship of Acts II and III) may become an object of re-verification.
- **When the reference corpus's scale (total word count) differs substantially across author classes, even a windowing-based method like Step 1 can produce an artifact favoring the larger reference** (confirmed by measurement, with Baillie's two works four times the size of Southey's). Treat results from the full 9-reference set with caution on this basis.
- The phenomenon in Step 0.5's Bootstrap Consensus Tree — Acts I, II, and III clustering together ahead of any external reference — is a finding not to be overlooked, suggesting that **shared subject matter/topic may be exerting a stronger effect than authorial discrimination**.
- **Any split-point detection involving a large-scale reference must always be verified by re-running with that reference excluded.** In Author-Attribution Phase 2, including Baillie_Basil caused the nearest neighbor at all three split points to shift to Baillie; excluding it largely restored validity as a Coleridge/Southey crossing, though two locations (0.002–0.012 margin) remained narrow. Do not overstate a conclusion based merely on "a crossing occurred."
- **Step 2(a)'s measured results confirmed that Acts I, II, and III are consistently closer to each other than to any external reference. This is a more fundamental phenomenon (a shared-topic effect across the three acts) that persists even after excluding Baillie et al., and should be distinguished from a scale artifact.** Avoid using dendrogram clustering topology itself as evidence of authorship; instead examine the ranked distance from each act to each individual reference.
- **What superficially looks like "approach toward Southey" turned out, upon comparing multiple split points, to be qualitatively different in each case**: Split Point 1 showed approach toward both Southey works (relatively strong evidence); Split Point 2 showed selective approach toward a single work, *Botany-Bay Eclogues* (suspected thematic coincidence); Split Point 3 showed isolation from all references (suspected instability from a short fragment). **Do not simply sum up "change was detected" across split points to reinforce the hypothesis.** Check the pattern of behavior via MFW sensitivity analysis for each split point individually, and take the qualitative differences into account when evaluating.
