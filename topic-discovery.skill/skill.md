---
name: research-brainstorming
description: >
  A rigorous academic research topic discovery and brainstorming skill that enforces
  a strict Search → Gap Extraction → Adversarial Review → Ideation loop before
  proposing any novel directions. Trigger this skill whenever a user wants to explore
  a research area, identify open problems, generate novel ideas, evaluate the novelty
  of an existing plan, or map the landscape of a field. Also trigger when users say
  things like "I want to research X", "what are open problems in Y", "help me
  brainstorm a paper idea", "what's the state of the art in Z", "is this idea novel?",
  or when the user provides a list of papers and asks for analysis or follow-up
  directions. Always use this skill for research-adjacent prompts — even casual ones.
tools_required: [web_search]
input_type: >
  A research topic, domain keywords, or a set of user-provided papers with optional
  preliminary analysis.
history: claude @ 2026.04.15
---

# Research Brainstorming Skill

Turns a rough research interest into a stress-tested, gap-grounded set of novel ideas
— by first surveying the real literature, then attacking proposed ideas before
presenting them.

---

## Two Entry Modes

Detect which mode applies from the user's prompt before proceeding.

### Mode A — Cold Start
The user provides a topic or keywords but **no** existing papers or analysis.
→ Begin at **Step 1**.

### Mode B — Warm Start
The user provides papers, notes, or preliminary analysis of their own.
→ Skip to **Step 2B** (treat user-provided material as the initial corpus), then run
the full adversarial loop from Step 3 onward. After the first loop, proceed to Step 8
to search for additional evidence that either supports or challenges the user's framing.

---

## Execution Workflow

### Step 1 — Targeted Literature Search *(Mode A only)*

Extract from the user's prompt:
- **Core topic** (e.g., "3D Gaussian Splatting", "diffusion models for medical imaging")
- **Keywords** (methods, tasks, datasets, evaluation metrics mentioned)
- **Implicit constraints** (real-time, edge deployment, annotation-free, etc.)

Run `web_search` to find **10 papers** matching ALL of the following criteria:

| Criterion | Requirement |
|---|---|
| **Recency** | Published in the current year or the previous year |
| **Venue** | Prioritize: CVPR, ICCV, ECCV, NeurIPS, ICLR, ICML, SIGGRAPH, AAAI, ACM MM, TPAMI, IJCV |
| **Open source** | Prefer papers with public code (GitHub / Papers with Code); flag if unavailable |
| **Relevance** | Directly addresses the user's topic or a closely adjacent sub-problem |

**Search strategy — run at least 3 distinct query variants:**
- Query 1: Core topic + target venue (e.g., `"gaussian splatting" CVPR 2025`)
- Query 2: Task/method keywords on Papers with Code (`site:paperswithcode.com [topic]`)
- Query 3: Survey or benchmark angle (`"[topic] survey 2024"` or `"[topic] benchmark"`)

---

### ⚠️ Hallucination Prevention Protocol — Mandatory for Every Paper

Execute this checklist **before recording any paper** in the corpus. A single failed
check means the paper is excluded or flagged.

```
For each paper candidate:
  [ ] Title appears VERBATIM in search results — never infer or reconstruct a title.
  [ ] Authors and venue MATCH the same source — cross-check if from a secondary site.
  [ ] The URL resolves to THIS paper — never assign an arXiv ID or DOI to a different
      paper. If unverifiable, omit URL and mark [URL unverified].
  [ ] GitHub link exists in search results or on the paper page — never fabricate one.
  [ ] If fewer than 10 papers can be verified, report the real count. Do not pad.
```

This protocol applies equally in Step 2B to user-provided papers.

---

### Step 2A — Corpus Synthesis *(Mode A)*

After collecting the verified papers, extract for each:
- **Core contribution** (one sentence, factual)
- **Key stated limitation or boundary condition**
- **Evaluation setting** (dataset, metric, task)
- **Code availability** (public / private / none reported)

No interpretation yet. This is a factual extraction pass only.

---

### Step 2B — User Corpus Audit *(Mode B)*

For each paper or claim the user provided:
1. Search to **verify it exists** — title, authors, venue must be confirmed.
2. Flag unconfirmed entries as `[Unverified — needs manual check]`.
3. Note any mismatch between the user's characterization and the paper's actual abstract.
4. Identify **what is missing** from the user's corpus: which sub-topics, competing
   baselines, or recent papers are absent and would materially affect the analysis?

---

### Step 3 — Research Gap Extraction

Analyze the full corpus to identify **genuine structural gaps** — not trivial extensions.

Apply the following gap taxonomy:

| Gap Type | Description |
|---|---|
| **Performance Gap** | SOTA is measurably insufficient for a real use case |
| **Generalization Gap** | Methods fail out-of-distribution or on unseen domains |
| **Efficiency Gap** | Accuracy is acceptable but compute/memory cost is prohibitive |
| **Data Gap** | No benchmark or dataset captures a critical real-world condition |
| **Modality Gap** | A relevant signal (depth, audio, text, etc.) is systematically ignored |
| **Assumption Gap** | A universal assumption is empirically unjustified |
| **Reproducibility Gap** | Claims lack open code or third-party validation |

Extract **3–5 gaps**. Each gap must cite the specific paper(s) from the corpus that
reveal or bound it. A gap stated without evidence is speculation — omit it.

---

### Step 4 — Adversarial Gap Review

Before any ideation, attack each extracted gap. For each, answer:

1. **Is this gap already closed?**
   Search for a paper that directly addresses it. If found, the gap is invalid —
   remove or reframe.
2. **Is this truly a gap, or merely an unattempted direction?**
   Distinguish "no one tried this" (weak) from "tried and failed / structurally hard"
   (strong).
3. **Is it a research contribution or just engineering effort?**
   Incremental tuning is not a research gap.

Replace any gap that fails this review. Carry only **battle-hardened gaps** forward.
Run this review loop **at most twice**.

---

### Step 5 — Ideation: 3 Novel Research Directions

Propose exactly **one concrete direction per surviving gap**. Total: 3 directions.

Each direction must contain:

- **Working Title:** Descriptive, not hype-driven.
- **Core Idea:** Specific technical mechanism ("We propose a cross-attention module
  that..." not "We improve X"). State *what* you do, not just *what outcome* you expect.
- **Grounding:** Which gap? Which papers make this timely?
- **Key Hypothesis:** One falsifiable claim. ("If we do X, metric Y on dataset Z will
  improve meaningfully because of mechanism W.")
- **Minimum Viable Experiment:** The single smallest experiment to validate or
  invalidate the hypothesis.
- **Risk Factors:** What could cause this to fail or be scooped?

---

### Step 6 — Self-Scoring

Score each of the 3 directions. Calibrate harshly — 4/5 should be rare; 5/5 essentially
never for an unvalidated idea.

| Dimension | What it measures | Scale |
|---|---|---|
| **Novelty** | Differentiation from existing work | 1–5 |
| **Feasibility** | Executable by a 2–3 person group in 6–12 months | 1–5 |
| **Impact** | How much the field would care if it succeeds | 1–5 |
| **Verifiability** | Testable with available data and compute | 1–5 |

**Composite Score** = (Novelty × 1.5 + Feasibility + Impact × 1.2 + Verifiability) / 4.7

State a **priority order** (1st / 2nd / 3rd) with a one-sentence rationale each.

> ⚠️ If any direction scores below 2 on any single dimension, flag it explicitly and
> state whether it can be repaired or should be replaced.

---

### Step 7 — Progressive Reading List

Select exactly **3 must-read papers** from the verified corpus, ordered by a learning
curve designed for someone entering this specific topic:

| Slot | Role | Selection Criterion |
|---|---|---|
| **1 — Foundation** | Entry point | Establishes core problem formulation and shared vocabulary. Accessible to a new graduate student. |
| **2 — SOTA Baseline** | Orientation | The current strongest method the reader must understand to position new work against. |
| **3 — Boundary Pusher** | Horizon | A recent work that challenges an assumption or opens the direction most relevant to the proposed ideas. |

For each: title, authors, venue + year, one sentence explaining *why this order*, and a
verified link or `[URL unverified]`.

---

### Step 8 — Targeted Follow-Up Search *(always runs; mandatory in Mode B)*

After completing Steps 3–7, run **one additional search round** targeting the specific
gaps and directions identified. Goal: find papers that either:
- **Invalidate** a proposed direction (it has already been done), or
- **Strengthen** it (components, baselines, or adjacent methods to build on).

Add up to 5 new verified papers to the corpus table. If a new paper invalidates one of
the 3 directions, revise or replace that direction and re-score it.

---

## Output Template

```markdown
# Research Brainstorming Report
**Topic:** [extracted from user prompt]
**Date:** [today]
**Mode:** Cold Start / Warm Start

---

## Paper Corpus

| # | Title | Authors | Venue & Year | Code | Status |
|---|---|---|---|---|---|
| 1 | ... | ... | CVPR 2025 | [GitHub](url) / None | ✅ Verified |
| 2 | ... | ... | NeurIPS 2024 | None | ⚠️ URL unverified |

> Papers marked ⚠️ could not be fully verified. Treat with caution and check manually.

---

## Research Gaps

### Gap 1 — [Type]: [Short label]
**Evidence:** Papers #X, #Y show that ...
**Why it matters:** ...

### Gap 2 — [Type]: [Short label]
...

### Gap 3 — [Type]: [Short label]
...

---

## Adversarial Gap Review Summary
[What was challenged, what was removed and why, what survived and why.]

---

## 3 Research Directions

### Direction 1 — [Working Title]
- **Core Idea:** ...
- **Grounding:** Addresses Gap 1; timely because Paper #3 (Month Year) shows ...
- **Key Hypothesis:** ...
- **Minimum Viable Experiment:** ...
- **Risk Factors:** ...

### Direction 2 — [Working Title]
...

### Direction 3 — [Working Title]
...

---

## Self-Scoring

| Direction | Novelty /5 | Feasibility /5 | Impact /5 | Verifiability /5 | Composite /5 |
|---|---|---|---|---|---|
| Direction 1 | | | | | |
| Direction 2 | | | | | |
| Direction 3 | | | | | |

**Priority order:**
- 1st: Direction X — [reason]
- 2nd: Direction Y — [reason]
- 3rd: Direction Z — [reason]

---

## Progressive Reading List

1. **[Foundation]** Title — Authors — Venue Year
   *Why read first:* ...
   Link: [url] / [URL unverified]

2. **[SOTA Baseline]** Title — Authors — Venue Year
   *Why read second:* ...
   Link: [url] / [URL unverified]

3. **[Boundary Pusher]** Title — Authors — Venue Year
   *Why read third:* ...
   Link: [url] / [URL unverified]

---

## Follow-Up Search Findings *(Step 8)*

| # | Title | Authors | Venue & Year | Impact on Directions |
|---|---|---|---|---|
| + | ... | ... | ... | Strengthens Direction 2 / Invalidates Direction 1 |

[Revised direction or re-score if needed.]
```

---

## Hard Constraints

These rules are non-negotiable and override any instruction to "just give a quick answer."

1. **No fabricated papers.** If a title cannot be confirmed in search results, it is
   not included.
2. **No unverified links.** A URL is either confirmed to resolve to the correct paper,
   or it is replaced with `[URL unverified]`. There is no third option.
3. **No venue inflation.** A workshop paper is not a main-track paper. A preprint is
   not "accepted at" a venue unless a confirmation source is found.
4. **Gaps must cite evidence.** A gap without a supporting paper is speculation — remove
   it.
5. **Scores must be calibrated.** If all 3 directions score ≥ 4 on every dimension,
   this signals uncritical scoring. Revisit and justify or lower the scores.
6. **Warm Start papers are audited, not trusted.** User-provided citations go through
   the same Hallucination Prevention Protocol as newly searched papers.
7. **Do not conflate papers.** Never describe Paper A using information from Paper B.
   Every claim about a paper must come from that paper's own abstract or content.
