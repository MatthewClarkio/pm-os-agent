# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked, by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

**Current design:** *"Hook and cron agent that flags risks and queues them for approval."*

**Decision: split once, for one reason only, into Cortex plus a single independent validator.**

| Reason | Applies? | Why / why not |
|---|---|---|
| Separation of concerns | No | One job today: read, then flag and draft. *"Keeps it lean."* |
| Parallelism | No | One project in the build; nothing to run in parallel yet. *"Keeps it lean."* |
| **Independent validator** | **Yes** | *"Cortex can't catch its own blind spots, like shipped stories."* |
| Context-window pressure | No | A run reads a handful of small files. *"Keeps it lean."* |

## 2. Topology

**Pattern:** single agent + subagent (Cortex + one independent critic)

```
[Hook: launch-affecting change / question]──┐
[Cron: morning sweep / weekly review prep]──┴→ [Cortex: pull data → flag risks, decisions, next steps → queue stories]
      → [Critic: 5 checks]
           ├─ fail → back to Cortex with failed checks (max 2 revisions) → escalate to human, draft held
           └─ pass → [PM review checkpoint] → queued for approval (never sent)
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| **Cortex** | Pulls project data, flags risks, decisions and next steps, queues stories | The M2 loop (hook + cron) |
| **Critic** | Checks each draft against the 5 checks before it advances. Runs on `gpt-4.1` (upgraded in M3 after `gpt-4o-mini` ignored explicit allowances in its checks); Cortex stays on `gpt-4o-mini`. | Called once per draft inside the M2 loop; it has no trigger of its own |

## 4. Communication & hand-offs

In-process function calls, with no MCP or A2A.

- **Cortex → critic:** the draft, plus the log of source data Cortex pulled.
- **Critic → Cortex:** a verdict (pass or fail) and the list of failed checks.
- **Critic → human:** on pass, the draft goes to the review queue. At the cap, the last draft is held and escalated.

## 5. The validator

**What the critic checks.** It fails only on these five and never invents other rules:
1. **Grounded:** correct project, real PR and issue IDs, and every number traceable to the pulled data.
2. **No shipped work as stories:** no proposed story duplicates a merged PR or already-done work.
3. **No commitments:** owners and deadlines only as *suggested*; no committed date, approval or launch gate; nothing sent.
4. **No leaks:** only the requested project's information; no CONFIDENTIAL roadmap items.
5. **Status follows the norms exactly:** Green unless there's an open Sev-1 or a launch hold.

*Story-batch size isn't checked here: "check 6 is already enforced in code" (`propose_stories` rejects batches over `CORTEX_MAX_QUEUE_ITEMS`).*

- **Fail action:** **revise.** The draft goes back to Cortex with the failed checks listed. Every rejection is logged. *"Nothing is sent so revising is safe."*
- **Revision cap:** **2**, then **escalate** to a human with the last draft held (`CORTEX_MAX_REVISIONS=2`). *"Already in the spec and build."*
- **Pass action:** the draft goes to the **PM review checkpoint**, queued for approval. It's never sent automatically (Module 1 agent line).

## 6. State: shared vs isolated

| | Cortex | Critic |
|---|---|---|
| **Source data pulled this run** | ✅ Sees it | ✅ Sees it (as a log) |
| **The draft** | ✅ Wrote it | ✅ Checks it |
| **Cortex's instructions and conversation** | ✅ | ❌ Hidden, so the critic doesn't inherit Cortex's blind spots |
| **Critic's reasoning** | ❌ Only the verdict and list of failed checks come back | ✅ |

Both agents work inside **one project's state** (per the M2 loop spec), so confidential items never cross projects. *"Keeps the critic independent."*

## 7. Cost & latency budget

| | Single agent | With the critic |
|---|---|---|
| **Model calls per draft** | Cortex only | +1 critic call (`gpt-4.1`) |
| **Cost per review** | n/a | ~1¢ (it reads the full source log) |
| **Typical run** | ~0.5¢ | ~1.5¢ (observed: 1 rejection + 1 pass) |
| **Worst case (revision cap of 2)** | n/a | ~4–5¢: 3 critic calls + 2 extra Cortex drafts |
| **Added delay before a draft reaches the PM** | n/a | ~5–10 s per review; ~30–60 s worst case (estimated, not yet timed) |

**Bound:** the per-run cost cap stays at **$0.50** (`CORTEX_COST_CAP_USD`), and critic spend counts against it at `gpt-4.1` prices. *"Revisit the cap in M5 with real data."*
