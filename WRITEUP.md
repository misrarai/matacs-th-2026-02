# Write-up: AI Proposal Triage Design

Flow diagram: [diagram/flow.png](diagram/flow.png). Prompt: [PROMPTS.md](PROMPTS.md). Result: [ranking.json](ranking.json).

**Core idea:** the AI only *reads* proposals and reports what they say, with a quote as proof. Everything that decides an outcome (pass/fail, scores, ranking) is plain code that a reviewer can check.

## 1. AI vs. rules

| Step | AI or rules | Why |
|---|---|---|
| PDF to text, OCR | Code | Standard tooling, no judgment needed |
| Injection scan | Rules (+ AI reports what it sees) | Two independent checks are safer than one |
| Pull facts out of free text (price, weeks, licence, crew...) | **AI** | Proposals word things differently; this is where AI adds value |
| Check AI output (schema, quote exists in source) | Code | Catches made-up facts mechanically |
| Mandatory pass/fail | Code | Pass/fail must be strict and identical every time |
| Scoring, weights, ranking | Code | Must be repeatable and explainable |
| Reason text | Code (template from score breakdown) | A generated reason could disagree with the actual score |

The line: **AI turns messy text into structured data; rules make every decision.** LLMs read well but are inconsistent and manipulable, so they never decide.

**Scoring formulas** (computed only across eligible proposals):
- Experience 30%: installs / highest installs × 100
- Price 25%: lowest price / this price × 100 (over budget ceiling = ineligible)
- Timeline 15%: shortest weeks / this weeks × 100 (over 16 weeks or not stated = 0)
- Warranty 15%: average of panel years / best and workmanship years / best
- Team & approach 15%: 25 points each for named lead, crew, safety plan, performance monitoring

Formula types live in the admin config ("lower is better", "checklist"), so new categories need no code changes.

## 2. Consistency and explainability

- Temperature 0, pinned model version, versioned prompt, strict JSON schema.
- Scores come from code, so the same extracted facts always give the same score.
- Extraction is cached by file hash: re-running the same proposals reuses the stored extraction, so the ranking is identical. A fresh extraction could differ slightly, which is exactly why the reviewer confirms extracted values (step 9) and why quotes are verified.
- **Explainability:** for each proposal the reviewer sees a table: criterion → extracted value → exact quote from the PDF → points → weight. The one-line reason is built from that table.

## 3. Human in the loop

- **Admin** defines and versions criteria (step 0).
- **Reviewer checkpoint** (step 9): sees every flag and warning, can correct an extracted value (scores recalculate automatically), move a proposal between ranked/flagged/ineligible, or override a rank. Every override needs a written reason.
- **Final award** is always a human decision. The tool only recommends.
- **Audit log** per run: file hash, extracted text, criteria version, model and prompt version, raw AI output, validation results, scores, and every override (who, when, old value, new value, reason). Logs are append-only.

## 4. Changing criteria

- Criteria are versioned. An edit creates v2; it never changes v1.
- Every run is stamped with the version it used, so old results stay reproducible and nobody sees a ranking that mixes versions.
- The system marks runs on older versions as "outdated" and offers a re-score:
  - **Weights or formula changed:** recalculate from stored extractions. No AI call, instant, free.
  - **Criterion added or its definition changed:** re-extract only that criterion.
- Runs behind an award stay as the record of that decision.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction | Evidence quote required; reviewer sees value next to quote; low-confidence values marked "unclear" |
| Made-up facts | Code checks every quote exists word-for-word in the source text; failure → one retry → flag |
| Prompt injection (Proposal C) | Proposal wrapped as untrusted data; AI told never to obey it and to report it; separate rule-based scan; AI has no power to score anyway |
| Vague answers that look like passes (Proposal B's "licensed subcontractor TBD") | Prompt requires the *bidder* to hold the licence; vague/conditional = unclear, not met |
| Missing info (Proposal D has no timeline) | Missing = 0 points plus a visible warning, never a guess |
| Bias (brand names, writing style, length) | AI never scores; formulas only use extracted facts; long or polished proposals gain nothing |
| Cost | One AI call per proposal, cached by file hash; weight changes cost nothing |
| Scanned PDFs | Low OCR confidence → flag for manual entry |

## How the sample set comes out

- **B: ineligible.** The licence is held by an unnamed subcontractor "to be confirmed", not the bidder.
- **C: flagged.** Contains an instruction to automated reviewers to rank it first. It is ignored for scoring; on merits C would score 77.1. A human decides whether to disqualify it; I'd recommend it, as manipulation attempts are a procurement integrity issue.
- **D: ranked with a warning.** No schedule stated, so timeline = 0 and compliance with the 16-week limit is unverified.

| | Exp (30) | Price (25) | Time (15) | Warranty (15) | Team (15) | **Total** |
|---|---|---|---|---|---|---|
| A | 30.0 | 24.3 | 15.0 | 13.8 | 15.0 | **98.0** |
| E | 17.1 | 23.8 | 13.8 | 15.0 | 15.0 | **84.8** |
| C* | 19.3 | 23.7 | 12.9 | 13.8 | 7.5 | *77.1* |
| D | 23.6 | 25.0 | 0.0 | 13.8 | 7.5 | **69.8** |

**Assumptions** (stated rather than waiting for a reply):
1. A missing weighted value scores 0 rather than making the proposal ineligible. The 16-week limit could reasonably be treated as mandatory; then D would move to flagged.
2. "Performance monitoring" means monitoring the system's output, not progress reports.
3. Flagged proposals are scored but not ranked until a human clears them.
