# Agentic Blueprint for Non-Engineers

**A framework for building AI-agent automations you can actually trust · for people who are not necessarily programmers.**

עברית: [README.he.md](README.he.md)

The problem: it's easy to build an agent that produces beautiful output. It's hard to know whether that output is true. This framework builds the checking into the architecture from day one · so you know within one run whether the process you built actually works, before a client discovers a hallucination for you.

**Not a programmer? This is for you.** Wherever this document says "in code", you don't write that code. You define what to check, and ask your AI tool (Claude or similar) to write and run the script. Your job is the decisions, not the typing.

Born from real projects (competitor research, collect-and-analyze pipelines). Deliberately generic: fits competitor research, market research, monitoring, recurring reports, CV screening · any collect → analyze → output process.

> v0.2 · July 2026. Templates are currently in Hebrew · English templates planned.

---

## When to use this

When the task repeats, involves collecting / processing / deciding, and can be split into roles.
A simple one-off task doesn't need a process · it needs a prompt.

## The core principle

**Quality is part of the architecture, not an afterthought.** And the rule that drives everything:

> **A mechanism without measurement is faith. A mechanism with measurement is engineering.**

Every gate in this document answers four questions: what · how · how you know it works · when to skip it.

Seven gates, in two groups:
- **Quality core (3):** Verifier, Learning, Eval · the mechanisms that catch and fix mistakes.
- **Supporting infrastructure (4):** Provenance, Determinism, HITL, Context hygiene · the conditions that let the core work.

## Two levels · by risk, not "always"

Gates are a default with conscious exceptions, not dogma. Skipping a gate · state why explicitly, never skip silently.

| Level | When | Required |
|---|---|---|
| **Full** | Money, reputation, irreversible, client-facing | All 7 gates + full operational resilience |
| **Light** | Internal, reversible, low risk | Provenance + manual spot-check + cost cap |

## Standard anatomy

```
Context/Input → Collect → Analyze → Verify → Output
                                       ↑        ↑
                              HITL at risk points  Eval (cross-cutting measurement)

Output ─(lessons)→ Context of the next run
```

Verify stops claims before output. HITL (Human-in-the-loop · a human approves at risk points) enters at risk points. Learning is a loop **between runs**. Eval measures the whole process.

---

## The 7 gates

### 1 · Provenance / Citations (infrastructure)
**What:** every fact carries a source link. A fact without a source doesn't enter the output.
**How:** a mandatory source field on every record (URL + retrieval date).
**Be precise:** a link reduces hallucination, it doesn't eliminate it. Models invent links and attach real sources to claims they don't support. That's why the Verifier checks that the source actually supports the claim.
**How you know it works:** zero records without a source field (code check).
**When to skip:** purely creative pipelines with no factual claims.

### 2 · Verifier (core)
**What:** an agent whose job is to refute, not approve. It checks the claims that are most expensive to get wrong, against the source itself.
**How:** a different model than the one that generated · this reduces error correlation, it doesn't eliminate it (different models share training data). So whatever can be checked in code · check in code: numbers, dates, quotes = string match against the source. A code check always beats LLM judgment.
**How you know it works:** the Verifier is itself an LLM, so it gets measured · with a seeded set (see Eval in depth). Two numbers: false-approve (approved a corrupted claim · the dangerous failure) and false-reject (rejected a true claim · the silent failure). Log what was rejected, not only what passed.
**When to skip:** Light + output a human reviews line by line anyway.

### 3 · Learning loop (core)
**What:** a `lessons.md` file every agent reads at the start of a run. A caught mistake becomes a rule.
**How:** every lesson has an origin (which run, date). **Entry gate:** a new lesson takes effect only after human approval · a wrong rule spreads to all agents and becomes permanent.
**Without polluting context** (the tension with gate 7): a notebook that only grows drowns the agent and degrades performance (context rot · see gate 7). So lessons have a lifecycle:
- **Scoped by role:** each lesson is tagged to an agent, and each agent reads only its own.
- **Hard cap:** 10-15 active rules per agent. Want to add beyond that · retire one first.
- **Promote and retire:** a stable lesson moves into the agent's definition and leaves the notebook; a lesson that doesn't improve the eval score gets deleted. The notebook is a transit station, not an archive.
**How you know it works:** rules that don't improve the eval score get removed, and the notebook stays under its cap.
**When to skip:** never · it's the cheapest gate of all.

### 4 · Eval (core)
**What:** a measurement that runs on every change and answers with a number: the process works or it doesn't.
**How:** see "Eval in depth" below.
**How you know it works:** a pass rate recorded in history for every version.
**When to skip:** Light with manual checking. Even then: 5 fixed examples beat nothing.

### 5 · Determinism where possible (infrastructure)
**What:** comparisons and calculations in code, not in an LLM's memory. One source of truth (file / Sheet / DB).
**How:** diffs, sums, merges · in code. The LLM analyzes, the code counts.
**How you know it works:** same input twice = same output in the deterministic layers.
**When to skip:** never.

### 6 · HITL at risk points (infrastructure)
**What:** HITL (Human-in-the-loop). The agent proposes, the human approves anything expensive or irreversible. Proposals, not decisions.
**How:** a review file with checkboxes before any irreversible action.
**How you know it works:** zero irreversible actions without a sign-off.
**When to skip:** reversible, cheap actions only.

### 7 · Context hygiene (infrastructure)
**What:** two distinct failures:
- **Context rot** · model performance degrades as context grows and gets noisy. Fix: context split by role (scoped), regular pruning.
- **Staleness** · old data presented as current. Fix: live scan every run, every document dated.
**How you know it works:** each agent loads only what's on its list; no file without a date.
**When to skip:** never.

---

## Operational resilience · the second axis

Quality without resilience = a process that gives correct answers until it breaks silently. At Full level, all of these are mandatory:

- **Failure, retry, and partial collection.** A collection-completeness gate: the agent declares how many items it expected vs. received, and marks the run `success / partial / failed`. A partial run never continues silently. One partial collection poisons every analyzer downstream, and the Verifier ends up approving claims built on gaps.
- **Run log.** Per agent: input, sources, output, cost, time, status. Without it you can't diagnose a bad output, and the learning loop is blocked.
- **Secrets and PII.** API keys in a separate file, never in a prompt or a log. A list of fields that must never reach the LLM, or get masked first.
- **Cost cap as a circuit breaker.** A hard runtime cap (tokens / calls / time) + stop and alert. A stuck loop stops itself.
- **Rate limits.** Predefined pacing per source, rate-limit responses recognized as an explicit state (not "zero results"), backoff instead of silent failure.
- **Prompt versioning.** Every prompt change = a version, and every eval run is recorded against it. A prompt change is never adopted without an eval run.

---

## Eval in depth · the chapter that turns hope into knowledge

Eval is unit tests for a non-deterministic system: fixed input, expected output, automatic comparison, runs on every change.

### Decision tree by output type

- **Closed output** (extraction: price, name, date, classification) → **golden set**: 10-20 input→known-correct-output examples. Compared in code.
- **Open output** (summary, analysis, content) → no single correct answer, so check **properties**:
  - In code: does every claim have a source? All fields filled? Reasonable length?
  - LLM-as-judge with a rubric (a closed list of yes/no questions: "does the summary contradict the source?"), not "score 1-10". A judge model different from the generator.
- **The Verifier itself** → a **seeded set** (the key trick):
  1. Take ~20 real claims known to be true.
  2. Deliberately corrupt ~10: change a number, swap a name, attach an unrelated link, flip a direction.
  3. Run the Verifier on all of them without telling it which is which.
  4. Measure false-approve and false-reject.

### The decision rule · when a change is adopted

Two gates, both must pass:
1. **false-approve = 0** on the seeded set · approving a corrupted claim is the expensive failure, so this gate is hard.
2. **No regression:** the score doesn't drop vs. the previous version in history. Dropped · not adopted, period.

### The fixed set · an asset that compounds

- Lives in versioned files next to the process (`eval/golden-set.csv`, `eval/seeded-set.csv`).
- **Append-only:** examples are never deleted or "fixed" to make the score go up. A production mistake → joins the set. The set gets harder over time, exactly as it should.
- **History** (`eval/history.md`): date, prompt version, pass rate, false-approve, false-reject, decision.
- **Trigger:** every prompt / model / lesson change → run before adopting.

---

## Agent design

- **One role per agent.** If you need an "and" to describe it · that's two agents.
- **Collect once, analyze many.** One shared collection layer + completeness gate. Never scrape the same source twice.
- **Scoped context.** Each agent loads only what it needs.
- **Model routing.** The cheapest model that does the job; the expensive model only for hard reasoning. Agent cap and budget set upfront.
- **A new agent only for a new capability, not a new name.**

## Anti-patterns

- One agent that does everything · split it.
- "The agent will remember what changed" · no, diff in code.
- Output without sources · forbidden.
- Link = truth · no, the source must support the claim.
- A Verifier nobody measures · an oracle taken on faith.
- A lesson that enters without approval · a mistake that becomes law.
- Building without eval · you're hoping, not knowing.
- Fixing the set to raise the score · the set is append-only.
- Partial collection that continues silently · poisons the chain.
- Accumulating context without pruning · context rot.
- Running the expensive model on every subtask · waste.

## End-to-end example · competitor research

| Gate | Implementation | File/tool |
|---|---|---|
| Source of truth | One competitor table | competitors.csv |
| Collect | One collection agent: sites, prices, news | collector + run log |
| Completeness | "Expected 12 competitors, got 12" | run status |
| Analyze | One analyzer per output (report, table) | analyzer per output |
| Provenance | Every claim with URL + retrieval date | mandatory field |
| Verifier | Different model checks key claims; prices = string check in code | verifier agent |
| Eval | Golden set: 15 known Q&A; seeded set: 10 corrupted claims | eval/*.csv + history.md |
| HITL | Threat ranking and publishing · approval only | review.md |
| Learning | "Site X blocks scraping · use the API" | lessons.md |
| Cost | Per-run cap + model routing | config |

## Getting started

0. **Pick where it runs.** The tool you already use: Claude (Projects / Code), n8n, Make, or any agent tool. The gates don't depend on the tool · they're structure, not technology. Unsure? Claude + a folder of files is a great start.
1. Copy `templates/` into your project.
2. Fill in `templates/checklist.md` · it's a form, not reading material. As you fill it, ask your AI tool to build the agents per the roles you defined (collector, analyzer, verifier).
3. Build a small golden set (even 5 examples you know the answers to) and run a pilot against it.
4. From the real claims of the passing run · build the seeded set for the Verifier (pilot first, seeded set after).
5. From there: every change → eval → decision rule → history.

## License

MIT. Take it, use it, improve it. PRs welcome.
