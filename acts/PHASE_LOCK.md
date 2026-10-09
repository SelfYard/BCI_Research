# Research Phase Plan

> This document outlines a step-by-step plan for a research project focused on
> **cross-subject generalization in non-invasive BCI systems** (Motor Imagery, EEG).
>
> The plan is intentionally agnostic regarding specific technical implementations: 
> it describes the *expected outcome* of each stage and the *logic of transitions* between them, 
> without specifying particular research papers, models, or datasets.

---

## Guiding Principles

1. **One article per cycle.** No attempts at parallel replication 
    until the first cycle is complete.
2. **Binary outcome criterion.** A cycle is considered successful 
    if the model has been replicated (with or without deviations) and analyzed. 
    Publication, correspondence, and establishing collaborations are added benefits, not completion criteria.
3. **Documented deviation is better than a "silent" failure.** 
    If the replication process yields results that differ from the original ones, 
    that very discrepancy becomes the final outcome.
4. **Contingency plans are determined in advance.** 
    Conditions for switching to alternative scenarios are clearly defined for each stage.
5. **Each stage yields a concrete result (artifact).** 
    Whether it is code, notes, graphs, or text, no stage should conclude merely at the level of "understanding."

---

## Phase 0 - Preparation

**Goal:** establish the technical and bibliographic foundation for one cycle.

**Tasks**
- Select **one** candidate paper matching the inclusion criteria:
  - Topic: cross-subject and/or cross-session generalization.
  - Availability: public code, or sufficiently detailed architecture description.
  - Data: public benchmark (e.g., BCI Competition IV 2a/2b or equivalent).
  - Recency: preferably ≥ 2023.
- Audit personal skill gaps (PyTorch, MNE, signal preprocessing, XAI libraries).
- Set up a reproducible environment (Python, PyTorch, MNE, Braindecode,
  SHAP/Captum, MOABB if applicable, others XAI methods).
- Verify that the reference code executes end-to-end on a minimal example.

**Artifacts**
- `SELECTED_PAPER.md` - one paper, with justification.
- `ENV.md` - environment specification.
- Working repository scaffold.

**Exit criteria**
- Code from the selected paper runs without modification on a small subset of data.

**Branching**
- **If code does not run and cannot be fixed within 1 week** 
    return to paper selection, choose next candidate.
- **If skill gaps are severe (cannot train a toy CNN end-to-end)** 
    insert a short bootstrapping sub-phase (2–3 weeks) before Phase 1.
- **If no suitable paper is found after N attempts** 
    broaden inclusion criteria (relax recency, allow alternate public datasets).

**Branching**
- **If the code does not run and cannot be fixed within a week**
    return to the article selection stage and choose the next option.
- **If a significant skill gap is identified (inability to train a simple end-to-end convolutional neural network)**
    add a brief preparatory phase (2–3 weeks) before Stage 1.
- **If a suitable article is not found after N attempts**
    broaden the selection criteria (lower novelty requirements, allow the use of other publicly available datasets).

**Fallback**
- Reduce scope: choose a paper with a minimal architecture (e.g., EEGNet-based) even if the topic is less central.

---

## Phase 1 - Reproduction

**Goal:** obtain a working model on the target dataset with documented fidelity to the source.

**Tasks**
- Receipt and pre-processing of the target data set (filtering, epoching, normalization).
- Agree on pre-processing options with the source paper; document any ambiguity.
- Reproduce the architecture (run the provided code or override from the description).
- Train using a subject-independent (LOSO) or subject-adaptive protocol, e.g. indicated in the source.
- Log metrics, hyperparameters, seeds, and execution times.
- Run verification tests:
    - **Noise test:** training on shuffled data/data with permutation of labels; performance shouldn't exceed the chance.
    – **Overtraining test:** Compares the gap between training and testing.
    - **Leakage Test:** Ensure subject/scalp information does not cross boundaries.

**Artifacts**
- Reproducible training scenarios.
- `RESULTS.md` - table of indicators, comparison with the source, notes on deviations.
- `PREPROCESSING.md` - precise pipeline with labeled unknowns.

**Exit Criteria**
- The model is trained and evaluated reproducibly (same initial value -> same metric).
- Performance is either within ** source tolerance or ** tolerance.
quantified and documented.

**Branching**
- **If the characteristics correspond to the original ones (within tolerance)** -> proceed to step 2.
- **If performance is out of tolerance**
    - Contact the original authors with a specific, minimal question (see Step 3)
        outreach template applied in advance.
    - If no response within 2 weeks -> still proceed to step 2, treatment of abnormalities
        as an object of research.
- **If the reproduction is fundamentally broken (the model is not trained)**
    - Attempt to re-implement based on description only.
    - If the second attempt also fails -> contact author before continuing the cycle.
    - If still unsuccessful -> return to step 0 with the next candidate.

**Fallback**
- Downgrade to a simpler baseline architecture (e.g., CSP+LDA, EEGNet) that
    serves as a reference point for later analysis, even if it is not the
    paper's original model.

---

## Phase 2 - Interpretability Analysis (XAI)

**Goal:** produce a mechanistic, testable explanation of *why* the reproduced
model behaves as it does on the target dataset.

**Tasks**
- Select a minimal, complementary set of XAI methods, e.g.:
    - Feature attribution: SHAP / Integrated Gradients.
    - Structural: attention maps (if applicable) or covariance-matrix analysis.
    - Optional: Concept Relevance Propagation or frequency-domain methods.
- Apply each method to the trained model across subjects and folds.
- Visualize spatial (channel), temporal, and - where applicable - spectral attributions.
- Assess reliability of explanations:
    - Stability across folds/seeds.
    - Sensitivity to input perturbations.
    - Agreement (or disagreement) between XAI methods.
- Formulate 3–5 **theses** describing the model's internal behavior, each
    grounded in evidence (e.g., "attribution concentrates on motor channels for
    subjects with high performance, and on frontal channels for subjects with
    low performance, consistent with artifact contamination").

**Artifacts**
- XAI pipeline scripts.
- `THESES.md` - 3-5 claims, each with supporting evidence and confidence.
- Figure set (attribution maps, stability plots).

**Exit criteria**
- At least three theses are supported by reproducible evidence.
- Reliability of at least one XAI method has been assessed and reported.

**Branching**
- **If XAI outputs are unstable or uninterpretable**:
    - Restrict analysis to a subset of subjects or a single fold to isolate noise.
    - Switch to a more robust method (e.g., gradient-based over perturbation-based).
- **If theses contradict each other** -> this is a valid result; document the
    contradiction and hypothesize a cause.
- **If a promising improvement idea emerges** -> record it in a `IDEAS.md` backlog
    for Phase 4; do not interrupt Phase 2.

**Fallback**
- Reduce to single-method analysis (e.g., SHAP only) and single-dataset scope.

---

## Phase 3 - Synthesis and Outreach

**Goal:** convert the cycle's results into external signals: connections,
feedback, and possibly collaboration.

**Tasks**
- Write a concise technical report (2–3 pages):
    - What was reproduced.
    - What was observed (metrics, deviations, theses).
    - Where uncertainty remains.
- Prepare an outreach message to the original authors:
    - Framing: reproducer asking for clarification and offering a small,
        concrete observation*, not a reviewer.
    - Attach or link the report and code.
- If no reply within 2 weeks -> contact co-authors or lab members.
- If still no reply -> initiate **Strategy B**:
    - Reach out to 3–5 researchers/engineers in adjacent groups with a short
        message: what was done, what artifact exists, what kind of collaboration
        is sought.
- Record all outreach attempts and outcomes.

**Artifacts**
- `REPORT.md` - the technical report.
- `OUTREACH_LOG.md` - recipients, dates, replies, follow-ups.

**Exit criteria**
- Report written and at least one outreach message sent.
- Response status recorded (reply, silence, or rejection all count as closure).

**Branching**
- **If positive reply received** -> negotiate next step (joint analysis,
    shared dataset, co-authorship, mentorship).
- **If neutral reply received** -> ask for recommended follow-up papers or
    open problems; treat recommendations as new Phase 0 candidates.
- **If no replies after all attempts** -> proceed to Phase 4 with the report
    as the sole external artifact.

**Fallback**
- Publish the report and code publicly (GitHub, preprint) as a standalone
    open artifact, even without external feedback.

---

## Phase 4 - Decision and Iteration

**Goal:** decide the next cycle's direction based on evidence, not on momentum.

**Tasks**
- Review outcomes of Phases 1–3.
- Classify the cycle result:
    - **A - Reproduction + theses established.**
    - **B - Reproduction established, theses weak.**
    - **C - Reproduction deviated, deviation itself is the result.**
    - **D - Reproduction failed after fallbacks.**
- Select the next action based on classification:
    - **A** consider drafting a short paper (workshop/conference level) or
        expanding to a second paper for comparative analysis.
    - **B** repeat Phase 2 with stronger methods, or add a second model for
        comparison.
    - **C** write a short note on reproducibility in the specific sub-area.
    - **D** return to Phase 0 with a new candidate.
- Update `IDEAS.md` with any accumulated improvement hypotheses.
- Optionally, begin an automation/tooling side-project if the same XAI
    workflow has now been repeated > 2 times.

**Artifacts**
- `CYCLE_REVIEW.md` - classification, rationale, next action.
- Updated `IDEAS.md`.

**Exit criteria**
- A written decision with justification for the next cycle.

**Branching**
- **Toward publication** -> enter a dedicated writing sub-phase.
- **Toward tooling** -> enter a software-project sub-phase (open-source XAI
    pipeline for EEG).
- **Toward accumulation** -> restart Phase 0 with a new paper and the same
    workflow.
- **Toward external engagement** -> enter an outreach sub-phase targeting
    labs, PhD programs, or companies using the accumulated artifacts.

**Fallback**
- If no direction is clear after review, default to **accumulation**
    (restart Phase 0) - the portfolio itself is the asset.

---

## Cross-Phase Notes

### Estimated Duration (part-time, ~15 h/week)

| Phase | Duration     |
|-------|--------------|
| 0     | 2–3 weeks    |
| 1     | 8–12 weeks   |
| 2     | 4–6 weeks    |
| 3     | 2–4 weeks    |
| 4     | 1 week       |
| Total | 16–25 weeks  |

### Success Definition
A cycle is successful if it produces **at least one reproducible artifact and
one written conclusion or publication**, regardless reply, or collaboration.
