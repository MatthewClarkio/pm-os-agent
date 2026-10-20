# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop, by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** **Hook** (primary) + **cron** (backup)

- **Hook:** fires when a project change could affect a launch (a changed requirement, slipping deadline, blocker, resource conflict or missing approval), or when someone asks Cortex a question.
- **Cron:** a morning sweep across all projects, plus a weekly run to prepare for the initiative review.

**Why:** *"Hook catches changes immediately, cron catches anything missed."*

**Ruled out:**
- **Heartbeat:** *"Heartbeat is overkill and too expensive. I want to run Lean."*
- **Cron only:** *"A cron is not enough. It will be ineffective and fail. There could be misses that could have prevented work from being committed or action being taken by members of the team."*
- **Goal loop:** *"Goal loop could keep running and get expensive. Again I want this to run as Lean as possible but still be effective."*

**Duplicate check:** Cortex records the ID of every change it handles and skips any ID it has already seen, whether a hook or the cron delivered it. *"Keeps it lean."*

## 2. Goal / definition of done

Cortex has checked the relevant information, logged what it handled, and identified risks, decisions and next steps with **suggested** owners and deadlines and supporting evidence, all **queued for human approval**. Nothing is changed in project tools or sent. *"Keeps it consistent with my agent line."*

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | The output is queued, passes the critic's check, and every project in scope was checked (any project that hit a problem is listed as escalated). *"One bad project shouldn't block the rest."* | Stop. Leave the package for human approval and record the handled change IDs. |
| **Stuck / give up** | A tool fails 3 times in a row · the critic rejects the output twice (`CORTEX_MAX_REVISIONS=2`) · 8 steps without finishing (`CORTEX_MAX_ITERATIONS=8`) · required data is missing or sources disagree. *"Already in the build so it's testable."* | Stop the affected project (or the whole run, at the step cap), log why, hold any draft, tell a human, and carry on with other projects. |
| **Escalate to human** | **Commitments** (requirements, scope, budget, launch date, money, contracts, vendor promises) · **Approvals & launch gates** (approvals, test acceptance, launch authorisation, skipped checks, change-freeze exceptions) · **Priority conflicts** between projects · **Communications** needing approval · **Sensitive info** · **Out of scope** or conflicting instructions · **Production impact** · **No owner** identified. *"They all trace back to my agent line."* | HITL checkpoint: stop that action, hold the draft, and send it to a human with the evidence. Other projects carry on. |
| **Halt** | A person asks Cortex to pause or stop. | Stop the whole run immediately. |

## 4. State

**Scope:** per project, plus a small shared portfolio layer (who is committed where, and which dates depend on which). Confidential items never leave their own project. *"Keeps confidential info inside each project."*

| | What | Kept? |
|---|---|---|
| **During a run** | Pulled data, current draft, critic feedback, step and revision counts, each project's status (done / escalated) | Purged when the run ends |
| **Across runs** | Log of handled change IDs and actions (what changed and why); human-approved outputs | Kept |
| **Not stored** | Reference data (roadmap, requirements, resources) | Read fresh each run; approved records are authoritative |

## 5. The five things a loop can lean on

_`state` is always-on. `connectors` only if you already have one wired (e.g. a Jira key or Google MCP), otherwise just note it as a plan. `skills`, `subagents`, `work tree` scale with autonomy; "not needed yet, because…" is a valid answer._

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Not needed yet: Cortex only reads, and writes only to its own log and the approval queue. *"Not needed yet keeps it lean."* |
| **Skills** (reusable capabilities) | Not needed yet; planned: scheduling/dependency calculators and a launch-checklist check, once portfolio data is real. *"Not needed yet keeps it lean."* |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | None wired yet (tools read sample files in `00-build/fixtures/`). Plan: first connect the project tool (Jira or Smartsheet), **read-only**. Later, only if needed: requirements, resources, calendars, test results, vendor updates, meeting notes, owner directory. *"Keeps it lean and read-only."* |
| **Subagents** (independent check when the loop can't grade itself) | The critic independently checks Cortex's output, so Cortex isn't grading itself. More detail in M3 `orchestration-map.md`. |
| **State tracking** | See section 4: per-project state, a shared portfolio layer, and a log of handled change IDs. |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules, you'll add them to their own deliverables then, not here.

## Link to live loop

[`00-build/agent.py`](../00-build/agent.py)
