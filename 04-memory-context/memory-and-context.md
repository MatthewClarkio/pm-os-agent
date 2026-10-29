# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs, by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.
>
> 🗂️ **How the lab maps to this file:** In **Part A** (before the lecture) you don't edit this file, you rough-draft on scratch, focused on the per-source calls in **section 2** plus a quick remember/forget + "how it rots" sketch. In **Part B** (after the lecture) you complete **all five sections**; the Lab Guide's guided builder writes this file for you to copy in and commit.

## 1. Context budget

Each loop iteration receives Cortex's rules, the task, the team norms, the tool results it pulled
(project, activity, roadmap, past updates), and on a revision, the critic's failed checks.

**Priority order when context is tight** (cut from the bottom up):

1. **Rules + team norms**: never cut, they're the guardrails.
2. **The task**: the job itself.
3. **Activity + project**: the evidence every claim must cite.
4. **Critic feedback** (revisions only): what to fix.
5. **Roadmap slice** for the requested project.
6. **Past updates**: cut first.

*"Rules never get cut, past updates go first. They're format precedent only, activity still has the facts."*

## 2. Retrieve vs. long-context: per source

For each data source, decide: **retrieve** (narrow a large/changing corpus to the relevant slice) or **long-context** (just include a bounded set you can reason over).

| Source | Size / volatility | Decision | Deciding factor | Why |
|---|---|---|---|---|
| `get_activity` (PRs, issues, metrics) | Large, grows weekly / high | **Retrieve** | Volatility | *"It changes daily so preloading goes stale."* |
| `search_past_updates` (updates + decision log) | Large, only grows / low | **Retrieve** | Size | *"It's large and only grows."* |
| `get_roadmap` | Medium / low–medium; confidential flags | **Retrieve, filtered to the requested project** | Citation | *"Keeps confidential items out."* |
| `get_norms` (team playbook) | Small / low, but must stay current | **Long-context**, re-read every run | Cost | *"It's small and always needed."* |
| `get_task` (this run's brief) | Tiny / new each run | **Long-context** | Size | *"It's tiny and defines the job."* |

**Build vs. plan:**
- `get_roadmap` returns the whole file today, so Cortex sees Orbit and Pulsar and relies on the rules to keep them out. **Planned:** filter it to the requested project so confidential items never enter context.
- `get_norms` is a tool Cortex chooses to call today. **Planned:** load it into the system prompt every run so it can't be skipped.
- The task is long-context but is still **data, not instructions** (M5 jailbreak fixture).

## 3. Retrieval quality plan

Each move is there to fix how that source fails. No retrieved source is naive "fetch → stuff".
Long-context sources (`get_norms`, `get_task`) are read whole, so they need no retrieval moves.

| Source | Routing | Grading | Reranking | Self-verification | Caching | Why (your words) |
|---|---|---|---|---|---|---|
| `get_activity` | ✅ | | | ✅ | ❌ | *"Stops wrong-project data and ungrounded claims."* |
| `search_past_updates` | ✅ | ✅ | Later | | | *"Keeps other projects out, reranking later."* |
| `get_roadmap` | ✅ | | | ✅ | | *"Filter keeps confidential out, check 4 backs it up."* |

- **Routing**: every lookup is limited to the project named in the task (plus team-wide decisions
  for past updates). Fixes Vega data appearing in a Northstar run, and Orbit/Pulsar reaching context.
  *Build today:* tools take a `project_id`, but nothing stops Cortex asking for another project;
  `search_past_updates` and `get_roadmap` don't filter at all. **Planned.**
- **Document grading**: past-update matches that aren't relevant are dropped, and "no precedent
  found" replaces today's silent fallback to the first two entries. **Planned.**
- **Reranking**: newest past updates first, so the format follows recent precedent. **Deferred** until
  the archive is large enough to matter.
- **Self-verification**: the M3 critic, a separate `gpt-4.1` call, checks the draft against the
  source log: check 1 (grounded) for activity, check 4 (no leaks) for the roadmap. **Built.**
- **Caching**: **not used.** Activity changes daily, and a cached roadmap would miss a new
  confidential project (Pulsar arrived in the 2026-07-06 pull).

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (in-loop) | Task, tool results, drafts, critic verdicts | **This run.** Wiped at the end; only approved output survives |
| **Episodic** (past runs) | **Human-approved** status updates and decisions only. Never unapproved drafts, traces, or raw critic feedback | **12 months**, then archived out of search |
| **Semantic** (durable facts/prefs) | Norms, roadmap, decision log, **re-read from source every run** (no stored copy). Plus **human-approved lessons** from repeated critic rejections (e.g. *"don't propose stories for merged PRs"*) | Sources: always fresh. **Lessons: 90 days**, then a human re-approves or they expire |
| **Shared** (across agents) | Cortex → critic: the draft + this run's source log. Cortex → human: the review queue (draft + queued stories). The critic never sees Cortex's memory or lessons, so it stays independent | Source log: **this run**. Review queue: **until a human approves or rejects** |

*"Only human-approved updates should become memory, keeps it lean."* (M1) · *"Human-approved lessons keep it lean."* ·
*"90 days matches roadmap planning, keep 12 months."*

**Read/write scope (M1 agent line):** Cortex only **reads** memory. Only a **human approval** writes to it.
**TTLs → M5 bounds:** the 90-day and 12-month limits become testable bounds (e.g. "a lesson older than 90 days is never used").

## 5. Memory risks & mitigations

| Risk | Where it bites Cortex | Mitigation |
|---|---|---|
| **Drift** | An approved lesson or past-update format slowly stops matching how the team works now | 90-day lesson expiry with human re-approval; past updates leave search after 12 months; norms always read fresh |
| **Poisoning** | Bad content becomes memory: a wrong draft, a critic false positive saved as a "lesson", or instructions hidden in a task brief | Only human-approved content is ever written; raw critic feedback is never stored; task text is data, not instructions |
| **Staleness** | Old facts look current: a slipped launch date, last week's 41%, a roadmap without Pulsar | Expiry dates + no cached copies: activity, roadmap, norms re-read every run, every claim cites this run's pull (critic check 1) |
| **Confidential / retention** | Personal details kept longer than needed; confidential projects (Orbit, Pulsar) held in memory | Store only approved updates; traces wiped every run; roadmap filter keeps confidential items out of context; 12-month cap |

**Biggest worry:** staleness, *"fix with expiry dates on remembered facts."* **Next:** poisoning, *"a bad lesson
gets trusted forever."* The 90-day lesson expiry is the backstop: a poisoned lesson can't outlive a quarter
without a human re-signing it.
