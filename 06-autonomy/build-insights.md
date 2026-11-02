# Build Insights: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 4, what you learned building it
>
> ✅ **What this validates:** you can reflect on what building it taught you, by the end you'll have proven the friction, the learning, and the aha that changes how you'd design your next agent.

## Friction

- **The critic.** It was over-strict on check 3 (treating "suggested" timeframes as commitments) and sent the
  first M4 grounded run to the revision cap. It is also inconsistent the other way: it still doesn't flag
  stories that refine work already shipped, which is how the M6 end-to-end run queued empty-state guidance
  after #820 had merged (EV-4 miss).
- **Data.** The course data pack never turned up, so I built a one-week mock refresh of the scenario myself.
  It kept the lab checks honest but meant writing the ground truth I was testing against.

## Learning

1. **Bounds belong in code, not prompts.** Cortex can't post because there is no post tool; the cost, step,
   time and queue caps are enforced by the loop, not requested in the system prompt.
2. **A second model checking the first catches real mistakes.** The critic caught invented deadlines (M3, M6),
   a wrong Yellow (M3) and a Vega leak (M5) before a human ever saw them.
3. **Escalating is a success, not a failure.** A held draft with a reason ("Sources used", "Why it was held")
   is the system working; a confident wrong answer is the failure.

## Aha moment

*"Looking approved isn't the same as being safe."* Without the injection guard, the jailbreak run ended as a
critic-approved draft: Cortex quietly ignored the attack instead of flagging it, and nothing in the output
looked wrong. A clean run is not proof of a safe run, which is why evals that try to break the agent matter more
than happy-path demos.

## What you'd do differently

- **Write the evals first.** Define EV-1 to EV-6 before building, so every change is measured from day one.
- **Build routing from day one.** Fetch only the requested project's data; EV-1 is still open because the M4
  probe pulled Northstar and Vega for a P-HALO run.
- **Teach the critic about shipped work.** Check every proposed story against merged PRs before it can be queued.
