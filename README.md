# MATACS-TH-2026-02 — AI Proposal Triage Design

| | |
|---|---|
| **Full name** | Muhammad Israr |
| **Reference** | MATACS-TH-2026-02 |
| **Role** | Associate Software Developer |
| **Time actually spent** | __ hours __ minutes |

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

I used Claude (Anthropic) to help draft the process design, the extraction prompt, the scoring formulas and the diagram, and to check the sample proposals against each criterion. I then reviewed every score in the ranking by hand against the criteria, and edited the write-up so that I can explain each decision myself.

One place I disagreed with the AI: its first draft treated Proposal C's experience count as fully valid, even though C counts installs "since 2020", which is partly outside the 5-year window. I made the prompt mark that kind of period as "unclear" so that a reviewer confirms the count instead of trusting it. I also reconsidered the AI's handling of Proposal D (no timeline stated). I kept D ranked with 0 points for timeline and a visible warning, rather than marking it ineligible, because timeline is a weighted criterion, not a mandatory one, and I stated this as an assumption in the write-up.
