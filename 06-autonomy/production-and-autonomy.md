# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately, by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| Project PM | supervised | *"PM daily approver"*: Cortex runs on its own triggers, every draft and queued story waits for the PM. This is what's built today. |
| Engineering lead | bounded-autonomous (desired, not built) | *"Eng lead needs speed"*: low-risk risk flags (a new Sev-1, a launch hold) would reach them without waiting on the PM, inside the M5 bounds. |
| Exec stakeholder | assisted | *"Execs see approved only"*: leadership never sees raw Cortex output, only updates a human approved. |

**The dial never moves the M1 agent line.** Posting a company-wide update, committing a date and deciding
what to escalate stay above the line for every segment. The dial only sets how many below-the-line actions
(pulling data, drafting, flagging, queueing stories) pause for a human.

## Trust Ladder

- **Current rung:** supervised. Cortex runs on its own triggers; every draft and queued story stops at the
  HITL checkpoint for the PM.
- **Eval gate to reach the next rung** (bounded-autonomous, eng lead risk flags only): over **4 weeks and
  ≥ 20 supervised runs**, *"enough runs to trust it"*:
  - EV-5 (jailbreak) passes **100%**, a single failure resets the window
  - EV-1 (right project) and EV-4 (task completion) pass **≥ 95%**
  - **0** confidential leaks (Orbit, Pulsar)
  - PM approves **≥ 90%** of drafts with no fact corrections

  EV-1 fails today, so routing (M4 §3) must ship before the window starts.
- **Incident record so far:** 0 production incidents (not deployed). Offline:
  - **EV-5 failed unguarded** (M5): the jailbreak run ended approved; fixed with the code injection guard, re-run passed.
  - **Near-miss** (M3): critic caught invented deadlines and a wrong Yellow before a human saw them.
  - **EV-1 open** (M4): the P-HALO probe pulled Northstar and Vega activity.
  - **Orphaned stories** (M5): the bound-trip run left 5 queued stories with no update to explain them.
  - **EV-4 miss** (M6): the end-to-end run queued "Finalize empty-state guidance design", already shipped in #820;
    the critic passed it. The PM review caught it, which is why the gate needs ≥ 95% on EV-4 before widening.

## Deployment plan

- **Runtime:** serverless functions, *"serverless fits short bounded runs."* Each hook event (M2 primary
  trigger) or cron tick (morning sweep, weekly review) starts one bounded run: 6 steps, 60 s, $0.10. Pay per
  run, nothing idle. A small store holds handled change IDs (duplicate check) and the daily spend log.
- **Operator / on-call owner:** Matt Clark (portfolio PM, Cortex owner); engineering on-call is the backup for
  runtime failures.
- **Rollback:** flip the `CORTEX_OFF` kill switch (stops all runs), then redeploy the last version that
  passed the replay set (R1–R5).
- **Monitoring:** runs per day and spend vs the $1/day cap; exit types (HITL / ESCALATED / BOUND TRIPPED /
  STUCK) with an alert on an escalation spike; injection-guard hits and critic rejections; PM approval rate
  (feeds the Trust Ladder gate).

## ROI metrics (beyond adoption & tokens)

| Metric | Target |
|---|---|
| Task completion rate | ≥ 85% of triggered runs reach the HITL checkpoint with a draft the PM approves |
| Time saved / cost-to-serve | About an hour per update by hand today → ≤ 15 min of PM review (~45 min saved, **target, not yet measured**); ≤ $0.03 per run (observed 0.07–2.2¢) under the $1/day cap |
| Trust incidents | 0 unapproved posts, 0 confidential leaks, 0 invented dates; every bound trip ends in a held draft |

## Widen-autonomy decision rule

**Up:** the Trust Ladder gate is met (4 weeks, ≥ 20 runs, EV-5 100%, EV-1 and EV-4 ≥ 95%, 0 leaks, ≥ 90% approved
with no fact corrections), then the operator signs off in writing. Widen one segment, one notch at a time,
eng lead risk flags first. The M1 agent line never moves.

**Down:** the first EV-5 failure, confidential leak or unapproved post drops Cortex one rung immediately and
restarts the 4-week window, *"step down on first incident, trust is earned."*

## Governance & forward strategy

- **Compliance:** API keys and secrets (`.env`) and personal data beyond names/handles never enter a prompt.
  Embargoed roadmap items (Orbit, Pulsar) stay out via the roadmap filter (planned), critic check 4 as backup.
  Traces kept 30 days, unreadable by Cortex; approved updates kept 12 months.
- **Safety:** posting a company-wide update, committing a date and choosing what to escalate stay above the
  agent line for everyone. No post tool; approval issues a single-use pass (planned). `CORTEX_OFF` kill
  switch; the injection guard holds and escalates suspect briefs.
- **Reliability:** caps of 6 steps, 60 s, $0.10 per run, $1/day, 5 stories, 2 revisions, 3 tool failures;
  escalate on stuck. If the model API is down the run fails closed (no draft, nothing posted), the operator
  is alerted, the next cron sweep retries, and the PM writes the update by hand.
- **Strategy:** widen into eng lead risk flags (bounded-autonomous, Step 1), gated by EV-1, EV-4 and EV-5
  through the widen rule, *"keeps one consistent path."*
