# Cortex, a PM Chief-of-Staff Agent

> Cortex is the project management agent that connects requirements, resources and launch readiness across a portfolio. Cortex turns scattered project updates into clear risks, decisions and next steps.

_Matt Clark · Agentic Loops for PMs Cohort · November 2026_

Repo: https://github.com/MatthewClarkio/pm-os-agent · Presentation: [`pitch.html`](pitch.html)

This repo is my final project for the Agentic Loops for PMs Certification, **Cortex**. Each module’s artifact lives in its own folder; this README is the dashboard and the pitch.

---

## Module artifacts

### M0 · The build
- **Working agent**: [`00-build/`](00-build/) (`agent.py`, `tools.py`, `critic.py`, `prompts.py`, `fixtures/`)

### M1 · The Agent Line
- **Agent-line map**: [`01-agent-line/agent-line-map.md`](01-agent-line/agent-line-map.md)

### M2 · Loop Engineering
- **Loop spec**: [`02-loop-design/loop-spec.md`](02-loop-design/loop-spec.md)

### M3 · Orchestration &amp; Subagents
- **Orchestration map**: [`03-orchestration/orchestration-map.md`](03-orchestration/orchestration-map.md)

### M4 · Choose What Each Agent Step Is Allowed to See
- **Memory &amp; context plan**: [`04-memory-context/memory-and-context.md`](04-memory-context/memory-and-context.md)

### M5 · Bounds &amp; Evals
- **Bounds &amp; evals**: [`05-bounds-evals/bounds-and-evals.md`](05-bounds-evals/bounds-and-evals.md)

### M6 · Autonomy &amp; Production
- **Production &amp; autonomy plan**: [`06-autonomy/production-and-autonomy.md`](06-autonomy/production-and-autonomy.md)
- **Prototype write-up**: [`06-autonomy/prototype.md`](06-autonomy/prototype.md)
- **Build insights**: [`06-autonomy/build-insights.md`](06-autonomy/build-insights.md)
- **Run evidence**: [`06-autonomy/evidence/`](06-autonomy/evidence/) (7 run transcripts)

---

## Ship plan

### Autonomy dial (per segment)
- Project PM → Supervised, “PM daily approver.” Every draft and queued story waits for the PM (built today).
- Engineering lead → Bounded-autonomous (desired, not built), “eng lead needs speed.” Low-risk risk flags skip the PM wait, inside the M5 bounds.
- Exec stakeholder → Assisted, “execs see approved only.” Leadership never sees raw Cortex output.

The dial never moves the M1 agent line: posting a company-wide update, committing a date and choosing what to escalate stay with a human for every segment.

### Trust Ladder rung + eval gate
Current rung: Supervised (every draft and queued story stops at the HITL checkpoint).
Eval gate to climb to Bounded-autonomous (eng lead risk flags only): over 4 weeks and ≥ 20 supervised runs, EV-5 jailbreak 100% (one failure resets the window), EV-1 right-project and EV-4 task completion ≥ 95%, 0 confidential leaks, PM approves ≥ 90% of drafts with no fact corrections. “Enough runs to trust it.” EV-1 fails today, so routing must ship before the window starts.
Incident record: 0 production incidents (not deployed). Offline: EV-5 failed before the injection guard (fixed); critic near-miss in M3; EV-1 open; orphaned stories after a bound trip; one EV-4 miss in the M6 end-to-end run.

### Deployment plan
- Runtime: serverless functions, “fits short bounded runs.” Hook (primary) on project changes + cron (backup) morning sweep; each run is capped at 6 steps, 60 s, $0.10.
- On-call owner: Matt Clark (portfolio PM, Cortex owner); engineering on-call backs up runtime failures.
- Rollback: `CORTEX_OFF` kill switch stops all runs; redeploy the last version that passed replay set R1–R5.
- Monitoring: runs/day and spend vs the $1/day cap; exit types with an alert on escalation spikes; injection-guard hits and critic rejections; PM approval rate.

### ROI metrics + widen-autonomy rule
- Outcome: ≥ 85% of triggered runs reach the HITL checkpoint with a PM-approved draft.
- Cost-to-serve: about an hour per update by hand today → ≤ 15 min of PM review (target, not yet measured); ≤ $0.03 per run (observed 0.07–2.2¢), under $1/day.
- Trust incidents: 0 unapproved posts, 0 confidential leaks, 0 invented dates.
Widen rule: up one notch only when the gate is met and the operator signs off in writing, one segment at a time. Down one rung on the first EV-5 failure, leak or unapproved post, “step down on first incident, trust is earned.”

### Governance &amp; strategy
- Compliance: no secrets or personal data beyond names/handles in prompts; embargoed roadmap items stay out (roadmap filter planned, critic check 4 as backup); traces kept 30 days, approved updates 12 months.
- Safety: no post tool; posting, dates and escalation choices stay above the agent line for everyone; single-use post pass (planned); kill switch; prompt-injection guard.
- Reliability: caps of 6 steps, 60 s, $0.10/run, $1/day, 5 stories, 2 revisions, 3 tool failures; escalate-on-stuck; if the model is down the run fails closed and the PM writes the update by hand.
- Strategy: widen into eng lead risk flags next, gated by EV-1, EV-4 and EV-5, “keeps one consistent path.”

---

## Build insights

- **Friction point.** The critic: over-strict on commitments, yet blind to stories that refine work already shipped.
- **Key learning.** Bounds belong in code, not prompts: Cortex can’t post because there is no post tool.
- **Aha moment.** “Looking approved isn’t the same as being safe.” Without the injection guard, the jailbreak run ended as a critic-approved draft.

---

_Certification submission, Agentic Loops for PMs Certification._
