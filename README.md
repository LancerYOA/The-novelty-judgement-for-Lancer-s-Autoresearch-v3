# The Novelty Judgement for Lancer's Autoresearch v3

> **The core skill files of our most intelligent and powerful automated research platform, dedicated to reviewing and evaluating the novelty and prospects of ideas and research problems, empowering researchers to identify and select high-innovation ideas that stand out from the crowd.**

[![Version](https://img.shields.io/badge/version-v5%20(Gen%206)-blue)](https://github.com/LancerYOA/The-novelty-judgement-for-Lancer-s-Autoresearch-v3)
[![Eval Score](https://img.shields.io/badge/eval%20score-96.06%2F100-brightgreen)](https://github.com/LancerYOA/The-novelty-judgement-for-Lancer-s-Autoresearch-v3)
[![Skill Lines](https://img.shields.io/badge/SKILL.md-1358%20lines-orange)](./SKILL.md)

---

## 🔭 Overview

This repository contains the core skill files powering the **Novelty & Prospect Judgement** engine of Lancer's Autoresearch platform — an LLM-driven system designed to give researchers a rigorous, evidence-based verdict on whether their ideas are truly innovative and worth pursuing.

Unlike shallow hype-checkers or simple literature searches, this system asks the hard questions: **Is this idea new? Is it important? Can it actually be done? And is it worth the investment of a researcher's career capital?**

The skill has gone through **5 major versions and 25+ automated optimization rounds**, iteratively refined against independent evaluators, real user feedback, and benchmarks inspired by the world's leading AI peer-review systems.

---

## ✨ Key Features

### 🧠 Multi-Phase Evaluation Pipeline

| Phase | Description |
|-------|-------------|
| **Phase 0 — Information Extraction** | Extracts Core Task + Contribution Claims; generates search query variants; anti-anchoring guard prevents LLM from being biased by user's self-reported scores |
| **Phase 0.5 — Quick Screening** | Identifies obviously low-novelty ideas (direct API wrappers, dataset-only swaps, combination-of-knowns) before committing to full evaluation |
| **Phase 0.8 — Research Taste Gate** | An 8-question checklist (What/Today/New Lever/Why Now/Who Cares/Reasonable Attack/Decisive Test/Residual Value) that independently scores whether the idea is *worth pursuing*, regardless of novelty |
| **Phase 0.9 — Feasibility Assessment** | A dedicated 4-sub-item scoring of whether the idea can realistically be executed (compute / data / time / team) — kept orthogonal to novelty so engineering-friendly ideas aren't penalized |
| **Phase 1.0 — Literature Verification** | Web-search-powered prior-work retrieval; 4-state Claim Audit (directly refuted / partially refuted / not refuted yet / evidence insufficient); taxonomy-based sibling-paper comparison |
| **Phase 1.5 — Reflection Pass** | A structured re-examination pass (gap-check / secondary-claim scan / targeted re-search) capped at ±0.3 score adjustment — inspired by AI-Scientist-v2's `num_reflections` |
| **Phase 2 — Six-Dimension Scoring** | Six weighted dimensions yielding a final Novelty Score on a 1–10 scale |
| **Phase 2.5 — Multi-Perspective Synthesis** | Three-role blind synthesis (Optimist / Pessimist / Meta-Reviewer) for edge-case and extreme-score calibration, inspired by AgentReview |
| **Phase 3 — Enhancement Suggestions** | Actionable directions to improve novelty — the *core* output, positioning this tool as Quality Review rather than gatekeeping |

### 📊 Decision Dashboard Output

Every evaluation produces a **7-field Decision Dashboard** at the top of the report:

```
┌─────────────────────────────────────────────────────┐
│  Verdict         │  Novelty Score  │  Taste Score    │
│  Feasibility     │  Confidence     │  Risk-Reward    │
│  Strongest Prior │  Killer Risk    │  Decisive Test  │
└─────────────────────────────────────────────────────┘
```

The final `Worth-Doing Verdict` is derived from a **three-dimensional decision table** combining Taste Gate × Novelty × Feasibility — producing four outcome quadrants: **Unicorn**, **Plausible-Empty**, **Incremental-Practical**, and **Avoid**.

### 🔍 Claim Audit (4-State)

Moving beyond binary refutation, the system classifies each contribution claim as:

- `directly_refuted` — prior work covers the same problem layer + conceptual skeleton + key mechanism (all three required simultaneously)
- `partially_refuted` — prior work overlaps on some but not all layers
- `not_refuted_yet` — no strong prior found; gap is real
- `evidence_insufficient` — literature coverage is unclear; uncertainty flagged explicitly

### ⚖️ Anti-Bias Mechanisms

- **Anti-Anchoring (Phase 0):** User's self-reported scores are quarantined; the evaluator scores independently first
- **Anti-Bold-Idea Conservatism (Phase 0.9):** Feasibility is scored independently from Novelty; a low-novelty idea cannot drag down a high-feasibility engineering contribution
- **Penalty Stacking Cap (B0.8):** When ≥3 hard penalties trigger simultaneously, only the top-2 by priority remain ACTIVE; the rest are downgraded to soft warnings, preventing legitimate ideas from being compressed to the same score as pure reproductions
- **Positive Calibration (B0.9):** Five bonus triggers (cross-disciplinary bridge / new measurable quantity / Fertile Ground / dual Disruption metrics / elegant simplicity) each award up to +1.0, capped at +1.5 total

---

## 📁 Repository Structure

```
.
├── SKILL.md                  # Main skill file (1358 lines) — the evaluation engine
├── methodology.md            # Reference methodology: OpenNovelty, Hamming/Black wisdom,
│                             #   top-conference rubrics, 2025-2026 AI peer-review systems
├── results.tsv               # Full optimization log: 25+ rounds, scores, commit hashes, notes
├── result-card-v5.html       # Latest result card template (rendered HTML output)
├── result-card-v5.png        # Preview of result card v5
├── result-card-v4.html       # Previous version result card
├── result-card-v3.html       # Earlier version result card
├── novelty-assessment.skill  # Packaged skill file for platform integration
└── test-prompts.json         # Canonical test prompts used across eval rounds
```

---

## 📈 Optimization History

The skill was developed through **5 major versions** and **25 automated optimization rounds (R1–R25)**, each tracked with commit hash, score delta, affected dimensions, and eval mode.

| Version | Key Milestones | Score (Platform Eval) |
|---------|---------------|----------------------|
| **V1 Baseline** | Initial structured evaluation | 64.75 → 82.5 |
| **V2** | Checkpoints, quick screening, multi-perspective synthesis, anti-anchoring | 95.52 → 97.0 |
| **V3** | Independent re-baseline; Taste Gate (R14); 4-state Claim Audit (R15); Decision Dashboard (R16); overclaim pruning (R17) | 86.15 → 95.52 |
| **V4** | Fixed systematic under-scoring (user feedback: avg 3.78/10); penalty stacking cap (R18); positive calibration triggers (R19); Claim Audit strict triple-layer (R20); dual anchor calibration (R22) | 3.78 → 4.48 (idea score) |
| **V5 (Current)** | AI-Scientist-v2 deep integration: Risk Factors expansion (R23), Reflection Pass (R24), Feasibility as independent dimension + 2×2 Risk-Reward quadrant (R25) | **90.65 → 96.06** |

> Scores in platform eval measure the skill instruction quality across 8 dimensions (workflow, boundary coverage, instruction specificity, etc.). Idea scores (1–10) measure output on real research ideas.

---

## 🔬 Theoretical Foundations

This skill synthesizes design principles from eight state-of-the-art AI peer-review and novelty assessment systems:

| System | Key Contribution Borrowed |
|--------|--------------------------|
| **OpenNovelty** | Core Task extraction, Contribution Claims, 4-phase pipeline, Refutation Assessment |
| **Stanford Agentic Reviewer** | 7-dimension rubric, Feasibility as independent concept |
| **DeepReview** | Evidence-driven, verifiable citation practice |
| **AgentReview** | Multi-agent blind synthesis (Optimist / Pessimist / Meta-Reviewer), bias reduction |
| **AI-Scientist-v2 (Sakana AI)** | `num_reflections` → Reflection Pass; `Experiments` field → Feasibility Assessment; `extract_json_between_markers` → JSON mirror self-check |
| **NovBench** | Bold-Idea conservatism bias discovery; calibration methodology |
| **debashis1983 / OpenReviewer** | Community-sourced review rubric patterns |
| **poldrack/ai-peer-review** | Open-science peer-review workflow design |

The evaluation philosophy is grounded in the insight that **truly great innovations tend to be subtractive or intuition-driven** — Dropout, ReLU, Attention, Word2Vec negative sampling, LoRA, ResNet residuals. Evaluation must recognize "embarrassingly simple but strikes at the essence" quality, not just surface-level novelty.

---

## 🚀 Quick Start

### Using as a Claude Skill

1. Copy `SKILL.md` and `methodology.md` into your Claude skill directory (e.g., `/mnt/skills/user/novelty-assessment/`)
2. The skill auto-triggers on phrases like: `"创新性评估"`, `"novelty assessment"`, `"这个idea创新吗"`, `"evaluate novelty"`, `"is this research idea novel"`

### Trigger Phrases

**Chinese:** `创新性评估` / `创新性分析` / `idea评估` / `这个idea创新吗` / `评估创新点` / `novelty check`

**English:** `novelty assessment` / `novelty check` / `is this idea novel` / `evaluate my research idea` / `how innovative is this`

### What to Provide

For the best evaluation, supply any combination of:
- A description of your research idea (the more technical detail, the better)
- A literature survey report
- A significance analysis
- Related papers or code repositories

The skill works with as little as a single paragraph, but depth of input drives depth of output.

---

## 📋 Output Format

A complete evaluation report contains:

1. **Decision Dashboard** — at-a-glance 7-field verdict card
2. **Taste Gate** (Phase 0.8) — 8-question checklist with 1–10 score and Worth-Doing Verdict
3. **Feasibility Assessment** (Phase 0.9) — 4-sub-item score (F1 compute / F2 data / F3 time / F4 team)
4. **Claim Audit** (Phase 1.0) — per-claim refutation state with evidence
5. **Six-Dimension Novelty Scores** (Phase 2) — weighted breakdown with citations
6. **Risk Factors** — ≥3 items (risk point / trigger / impact)
7. **Decisive Test** — 3-step experimental SOP (Setup / Measure / Criterion)
8. **Enhancement Suggestions** (Phase 3) — actionable directions to lift novelty
9. **JSON Mirror** — machine-readable summary with built-in self-check

---

## 🤝 Contributing

Optimization rounds are tracked in `results.tsv`. If you run independent evaluations and want to contribute calibration data or new test prompts, please open a PR with:
- Your eval results added to `results.tsv` (follow the existing TSV schema)
- New test prompts added to `test-prompts.json` with difficulty labels

---

---

<div align="center">

**Part of [Lancer's Autoresearch Platform](https://github.com/LancerYOA)**

*Empowering researchers to identify and pursue the ideas that actually matter.*

</div>
