# MATACS-TH-2026-02 — AI Proposal Triage Design

| | |
|---|---|
| **Full name** | Muhammad Israr |
| **Reference** | MATACS-TH-2026-02 |
| **Role** | Associate Software Developer |
| **Time actually spent** | __2.5 hours __ minutes |

## Repository contents

| File | What it contains |
|---|---|
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions, scoring formulas, assumptions and how the sample set is ranked |
| [diagram/flow.png](diagram/flow.png) | Process flow from upload to ranked output, colour-coded into AI steps, rule-based steps and human checkpoints |
| [PROMPTS.md](PROMPTS.md) | The AI extraction prompt exactly as it would be sent, plus the retry prompt |
| [ranking.json](ranking.json) | Expected ranking of proposals A–E |

No prototype is included, so the repository contains no code or API keys.

## Design in one sentence

The AI only reads each proposal and extracts facts, each backed by an exact quote. Plain code then verifies those quotes, applies the mandatory pass/fail checks, calculates the weighted scores and builds the ranking, and a human reviews the result and makes the final award.

## How I used AI tools

I used an AI assistant as a drafting aid: to brainstorm the process steps, draft the extraction prompt and the diagram, and cross-check each sample proposal against the criteria. I made the key design decisions myself: the AI only extracts facts, while code does all the scoring and ranking. I also recalculated the scores by hand before finalising ranking.json.

Where I corrected the AI: it accepted Proposal C's experience count as fully valid, but C counts installs "since 2020", which is partly outside the required 5-year window. I changed the prompt so that this kind of period is marked "unclear" and must be confirmed by a reviewer. I also chose to keep Proposal D ranked with 0 points for timeline and a visible warning, rather than ineligible, because timeline is a weighted criterion, not a mandatory one. This is stated as an assumption in the write-up.
