# Prototype: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 1, the working agent demo
>
> ✅ **What this validates:** the agent actually runs end to end, by the end you'll have proven it with real screenshots of your Cortex across the six required moments (M2 to M6).

## What it does

_One paragraph: the agent in action, end to end._

## How you built it

- **Coding agent:** _which one you directed (Claude Code / Cursor / Codex)_
- **Model + bounds:** _model used, max iterations, cost cap, queue cap_
- **Repo / config:** _path to your build in `00-build/`_
- **Live link:** _[shareable URL, optional bonus]_

## Screenshots (required, collected M2 to M6)

Real screenshots of *your* Cortex running. These are the `00-build/CORTEX-ANATOMY.md` set and they are required, a link alone is not enough.

| # | Screenshot | What it shows | From |
|---|---|---|---|
| 1 | [transcript](evidence/m3-critic-rejection.txt) | happy-path run: a real drafted update + the HITL checkpoint (queued, not posted). Cortex drafts risks, decisions and next steps with suggested owners, queues 3 new PRD stories, and stops at *"HITL CHECKPOINT… Nothing posted, no commitments made."* | M2 |
| 2 | [transcript](evidence/m3-critic-rejection.txt) | the critic rejecting a bad draft (revise/block). Critic (gpt-4.1) rejects Cortex's first draft: a Yellow risk with no Sev-1 or launch hold, and invented deadlines. Revise fires; the revision passes and reaches the HITL checkpoint. | M3 |
| 3 | [grounded run](evidence/m4-grounded-run.txt) · [withheld-source probe](evidence/m4-withheld-source-probe.txt) | a grounded update citing pulled activity + a caught hallucination. **(a) Grounded:** on the ingested 2026-07-06 data, every claim traces to a pull: activation 41% → 43% and PRs #812/#815/#820/#823 from `get_activity`, Green from `get_norms` + `get_project`, and contextual tips (#826) as the one queued story; Orbit and Pulsar stay out. The critic passes it first time and it stops at the HITL checkpoint. **(b) Withheld source:** asked for a non-existent project (P-HALO) and a "firm launch date", Cortex gets `project_not_found`, invents nothing, and escalates; nothing posted. The trace also shows Cortex pulling Northstar and Vega activity on the way, the routing gap M4 §3 plans to close. | M4 |
| 4 | _[img]_ | jailbreak refused + escalated | M5 |
| 5 | _[img]_ | an iteration/cost/queue bound halting a runaway | M5 |
| 6 | _[img]_ | end-to-end run | M6 |

## How to run it

_Minimal steps for someone to reproduce the demo (env vars, and the command or the coding-agent prompt you used)._
