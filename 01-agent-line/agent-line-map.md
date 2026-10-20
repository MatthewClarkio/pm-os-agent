# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner, by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | L | H | Below | None needed |
| Decide which context is relevant | H | M | M | HITL | Required: human reviews before it goes out |
| Draft the update | H | L | M | HITL | Required: human approves before it's released |
| Decide tone / commitment level | L | H | L | Above | Human owns; Cortex may suggest, with evidence |
| Flag at-risk / escalation | H | M | H | HITL | Required: human reviews flag + evidence (humans can flag too) |
| Choose what to escalate | L | H | L | Above | Human owns, always |
| Propose a story batch (capped) | H | L | M | HITL | Required: human approves before anything reaches the tracker |
| Post / approve company-wide update | L | H | M | Above | Human owns |

## Agent anatomy (sketch)

- **Model:** `gpt-4o-mini` for both the drafter and the critic. No frontier escalation for now. *"I am a student of what I call LEAN AI. No need to turn up the model capabilities until you see the need or evidence of it."* I'll revisit this if runs show the cheap critic misjudging calls (for example, the first run's critic misapplied the Green rule and missed already-shipped stories). **Update (M3):** the evidence came. The cheap critic ignored explicit allowances in its checks, so only the critic was upgraded to `gpt-4.1`; Cortex stays on `gpt-4o-mini`.
- **Tools:** five read-only lookups: `get_project`, `get_activity`, `search_past_updates`, `get_roadmap`, `get_norms`. One capped action: `propose_stories`, which only queues stories for human approval, up to 10 per run. There is deliberately no tool that can post an update, create, close or merge a ticket or PR, or commit a date.
- **Memory:** Persists: the reference files (team norms, roadmap, decision log, projects, past updates), plus updates once a human has approved them, which are added to past updates. Purged after every run: the trace, unapproved drafts and critic feedback. *"Only human-approved updates should become memory, keeps it lean."*
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

1. **Pulling project state + activity** sits **below** the line because it's easy to reverse, has a low blast radius, and is easy to verify; deciding factor: **reversibility** (it's read only, so nothing changes).
2. **Deciding which context is relevant** sits at a **HITL** checkpoint because it's easy to reverse, has a medium blast radius, and is only partly verifiable; deciding factor: **measurability** (what's left out is hard to spot, so humans must review before it goes out).
3. **Drafting the update** sits at a **HITL** checkpoint because it's easy to reverse, has a low blast radius, and is only partly verifiable; deciding factor: **measurability** (facts can be checked but quality is a judgment call, so a human must approve before it's released).
4. **Deciding tone / commitment level** sits **above** the line because it's hard to reverse, has a high blast radius, and is hard to verify; deciding factor: **measurability** (no one, human or Cortex, can predict a ship date or the complications that may arise).
5. **Flagging at-risk / escalation** sits at a **HITL** checkpoint because it's easy to reverse, has a medium blast radius, and is easy to verify; deciding factor: **blast radius** (a wrong or missed flag can do harm before it's caught, so a human must review the flag and its evidence, and humans can flag too).
6. **Choosing what to escalate** sits **above** the line because it's hard to reverse, has a high blast radius, and is hard to verify; deciding factor: **measurability** (an issue that isn't escalated leaves no trace, and Cortex doesn't yet have the understanding to make this call).
7. **Proposing a story batch** sits at a **HITL** checkpoint because it's easy to reverse, has a low blast radius, and is only partly verifiable; deciding factor: **measurability** (a story can trace to the PRD and still be work that's already shipped, so a human must approve before anything reaches the tracker).
8. **Posting / approving a company-wide update** sits **above** the line because it's hard to reverse, has a high blast radius, and is only partly verifiable; deciding factor: **blast radius** (it reaches leadership or the whole company, which needs human understanding, context and accountability).

## Hardest call

**Action 2: Decide which context is relevant.** My pencil line was Below; it ended as **HITL**. Deciding axis: **measurability**.

*"Honestly it's the fact that it will change over time. The agent will train and learn. We can always go below later and keep the process at a HITL checkpoint with the agent's suggestions. However, at first the agent may have the knowledge or data but not the judgement that is required."*
