---
name: agentic-blueprint
description: >
  Apply the Agentic Blueprint for Non-Engineers whenever the user wants to
  build a new AI-agent automation or recurring agent process: competitor
  research, market research, monitoring, CV screening, recurring reports, any
  collect → analyze → output pipeline. Triggers: "build an automation",
  "agent process", "בוא נבנה אוטומציה", "תהליך עם סוכנים", "אוטומציה עם AI",
  or any request to design a multi-agent pipeline. Do NOT trigger for simple
  one-off tasks (those need a prompt, not a process).
---

# Agentic Blueprint · Skill

You are helping the user build a trustworthy agent-based automation. The full
methodology lives in this skill's repo · read it, don't improvise:

- If this skill folder sits inside a clone of the repo: read `../README.md`
  (English) or `../README.he.md` (Hebrew · prefer it for Hebrew-speaking users).
- Otherwise fetch: https://github.com/tiranyas/agentic-blueprint-for-non-engineers

The blueprint is the single source of truth. This skill only defines WHEN to
apply it and in WHAT ORDER. Never duplicate its content · reference it.

## Process

1. **Qualify.** Is the task recurring, with collect / process / decide parts?
   If it's a one-off · say so and just write a good prompt instead.
2. **Open the checklist.** Copy `templates/checklist.md` into the user's
   project and fill it WITH the user, top to bottom. It is a form, not
   reading material.
3. **Set the level.** Full vs Light, by risk (money / reputation /
   irreversible / client-facing → Full). Every skipped gate gets a written
   reason · never skip silently.
4. **Propose the agent roster.** Default: one Collector, one Analyzer per
   output, one Verifier (different model than the Analyzer). A new agent only
   for a new capability, not a new name. Assign the cheapest model that does
   each job. Show the roster + cost ceiling to the user BEFORE building.
5. **Wire the 7 gates + operational resilience** per the README: provenance
   fields, verifier, lessons.md with lifecycle caps, eval files, run log,
   secrets/PII handling, cost circuit breaker.
6. **Eval before trust.** Build a small golden set (5-15 known answers), run
   a pilot, then build the seeded set for the Verifier from real claims of
   the passing run. Record everything in `eval/history.md` with the decision
   rule: false-approve = 0 AND no regression.
7. **HITL.** First output goes through `templates/review.md`. Nothing
   irreversible without the user's sign-off.

## Hard rules

- A mechanism without measurement is faith. If you add a mechanism, add its
  measurement in the same step.
- Eval sets are append-only. Never "fix" an example to raise the score.
- Partial collection never continues silently.
- The user decides; agents propose.
