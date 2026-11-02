# SWE AI Builder Workflow

**Build LLM features that are worth building, behave predictably, and are owned.**

Version 1.0 · October 2026 · Companion to [`SWE AI Builder Copilot.md`](./SWE%20AI%20Builder%20Copilot.md) and [`System Review Copilot.md`](./System%20Review%20Copilot.md)

---

## Contents
- [What this is](#what-this-is)
- [The one principle](#the-one-principle)
- [The seven phases](#the-seven-phases)
- [Take it on or not](#take-it-on-or-not)
- [What to build](#what-to-build)
- [How to start in each tool](#how-to-start-in-each-tool)
- [Rules of the road](#rules-of-the-road)
- [What this teaches at each level](#what-this-teaches-at-each-level)
- [System Review](#system-review)
- [Files](#files)

---

## What this is

A seven-phase workflow your AI coding agent runs with you (**Claude Code**, **GitHub Copilot** agent mode, or **OpenAI Codex**), from *"should I build this?"* to *"it's running and someone owns it."* The agent interviews you, drafts, writes code and tests, and stops at a **gate** after each phase for your approval.

Everything it produces lives **in the repo**: docs, rules module, tests, evals, and README. Nothing important stays only in chat history.

It adapts a business-focused workshop exercise into an engineering workflow:

| Workshop version | This version |
|---|---|
| Chat ledger and a PDF spec | Repo files and code |
| Business case first | A take-it-on-or-not gate that checks data access and build vs. buy |
| A governance-heavy UI | The UI leads with the user's answer; the trace sits behind "Why?" |
| Tests only | Unit tests for rules, **evals** for LLM steps |
| Security implied | Prompt injection, secrets, and external actions handled explicitly |

---

## The one principle

> **AI proposes, code decides, people own the high-stakes calls, and every miss becomes a test.**

**Why:** LLMs are probabilistic. 20 chained steps at 98% reliability each succeed only about **67%** of the time. Rules are deterministic, testable, and explainable. So:

- **LLM at the edges:** messy input → structured fields, and results → prose
- **Rules in the middle:** pure functions make every decision
- **Human in the loop:** wherever a wrong answer is costly
- **Every miss becomes a test:** each bug or complaint turns into a test or an eval case

```
 untrusted input ──► LLM reads ──► validate ──► RULES decide ──► LLM drafts ──► check draft ──► user
                        │             │              │                              │
                        └── retry ◄───┘              └──► HUMAN (high stakes)       └──► retry → human
```

---

## The seven phases

| # | Phase | You do | Agent writes |
|---|---|---|---|
| 0 | **Kickoff** | Pick a mode (Spike or Full) | Capabilities check, repo scan |
| 1 | **Discover** | Answer 6 questions | `docs/brief.md` |
| 2 | **Decide** | Approve GO / RESCOPE / NO-GO | `docs/decision.md` |
| 3 | **Design** | Approve rules, tests, and contracts | `docs/rules.md`, `tests/`, `docs/llm-contracts.md`, `evals/` |
| 4 | **Build** | Review diffs, run it | Rules module, LLM adapters, UI, `README.md` |
| 5 | **Verify** | Put it in front of a real user | Self-review, feedback log, new tests |
| 6 | **Ship & Own** | Decide rollout, pitch it | `docs/ownership.md`, `docs/pitch.md` |

### Modes

| | Spike (a few hours) | Full (days to two weeks) |
|---|---|---|
| **Use for** | Learning or a proof of concept | Something people will rely on |
| **Interview** | 6 questions | Up to 15 |
| **Tests** | 5+ golden, plus malformed input | Every threshold boundary, plus malformed input |
| **Evals** | 5–10 examples | 20–50 examples with a pass threshold, in CI |
| **Ship** | README + ownership note | Feature flag, ownership plan, pilot, pitch |

---

## Take it on or not

The Phase 2 gate. Score each criterion **1–5**:

| Criterion | Question |
|---|---|
| Value | Is the time saved or risk avoided worth the build and upkeep? |
| Data access | Can you actually get the inputs, legally, at the freshness you need? |
| Decidability | What share of the decisions can be explicit rules? |
| Evaluability | Can you label right answers? *Could you teach it to an intern?* |
| Blast radius | How bad is a wrong output? (5 = small) |
| Scope | Does a useful slice fit in one spike or sprint? |
| Demand | Is a named person asking for it? |
| Build vs. buy | Does a tool you already have do this? (5 = nothing does) |

### Hard stops
- **Data access ≤ 2 or Evaluability ≤ 2** → NO-GO or rescope. *A design with no reachable data source is a demo, not a product.*
- **Blast radius ≤ 2** → the core decision stays human; the tool only prepares it.
- **Build vs. buy ≤ 2** → configure the tool you have instead of building one.
- **Value ≤ 2** → say so, and continue only as an explicit learning build.

### Pick the product shape
The shape decides what the UI shows first:

| Shape | UI leads with |
|---|---|
| Workflow decision | The decision and the next owner |
| Information product | The answer at a glance: score, color, one-line reason |
| Generation assistant | The draft, with its checks |
| Action agent | The proposed action and its diff, for approval |

---

## What to build

### Top pick for a first project: flaky test triage or a PR risk labeler
- You already have the data (CI logs, diffs).
- Most decisions are clean rules: failure counts, paths touched, lines changed, tests present.
- The LLM has a clear, checkable job: cluster logs, summarize a diff.
- The human step is natural: approve a quarantine, merge the PR.
- The value shows up in your team's week, and it makes a strong portfolio and interview story.

### Other good builds
| Idea | Rules decide | LLM does | Human |
|---|---|---|---|
| Alert routing and summary | Owner and severity from the service catalog and thresholds | Summarizes the payload and recent deploys | Acks Sev1 |
| Dependency upgrade triage | Patch + green tests → auto-PR; major or CVE → human | Summarizes the changelog | Approves majors |
| Bug report intake | Required fields present → triage queue | Extracts fields, asks the reporter for what's missing | Sets priority |
| Release notes drafter | PRs selected by label and window | Drafts the notes | Edits and publishes |

**Personal and learning builds:** a transit station safety score (only with a real data source), a job application tracker, a bill and subscription checker.

### Don't take on (or rescope first)
- ❌ The LLM approves, merges, deploys, pays, or messages customers on its own
- ❌ It ranks or judges people (hiring, performance, eligibility)
- ❌ There's no reachable data, or the only source violates its terms
- ❌ Dependabot, Renovate, CODEOWNERS, or alert rules already do it
- ❌ It needs untrusted input + private data + an external action, with no control in between

---

## How to start in each tool

**Start line (all tools):**
```text
Read docs/swe-ai-builder.md and act as the SWE AI Builder. Start Phase 0 in Spike mode.
```

<details>
<summary><b>Claude Code</b></summary>

- **Option A (skill):** save the agent file as `.claude/skills/swe-ai-builder/SKILL.md`, then run `/swe-ai-builder`.
- **Option B:** save it as `docs/swe-ai-builder.md` and send the start line.
- Use **plan mode** (Shift+Tab) for Phases 1–3, so nothing is written until you approve the design.
</details>

<details>
<summary><b>GitHub Copilot (VS Code)</b></summary>

- Open Chat in **Agent** mode (needed to edit files and run tests).
- Attach the agent file with `#file` and send the start line, or save it as `.github/prompts/swe-ai-builder.prompt.md` to reuse it.
- Put the principles block (Appendix G of the agent file) in `.github/copilot-instructions.md`.
</details>

<details>
<summary><b>OpenAI Codex</b></summary>

- Put the principles block in `AGENTS.md` at the repo root (Codex reads it automatically).
- Save the agent file as `docs/swe-ai-builder.md` and send the start line.
- In cloud tasks, run one phase per task and review each diff.
</details>

---

## Rules of the road

- Use **synthetic or public data** unless the tool is approved for real data.
- **No secrets** in prompts, code, or logs. Model IDs and keys come from config or environment variables.
- **You review every diff.** The agent doesn't commit, push, or deploy without asking, and it reports only tests it actually ran.
- **Label every number:** `MEASURED`, `ESTIMATED`, or `ASSUMED`.
- **"No" at the Decide gate is a win:** it saves building the wrong thing.

---

## What this teaches at each level

### Junior: build LLM features you can trust
- LLM at the edges, deterministic code in the middle.
- Validate model output like untrusted input (schema, evidence, forbidden words, length). Retry once, then route to a person.
- Parse each input in one place, and test malformed input (`"1e3"`, `"12.5"`).
- Write golden tests before the UI; every bug becomes a test.

### Mid / Senior: decide what gets built
- Match each decision to the right decider: rules, LLM, or human.
- Measure before building: volume × time × cost.
- Label numbers, and separate what's measured from what's assumed.
- Check that the process fits the problem, and adapt it when it doesn't.

### Staff / Lead: get it adopted and owned
- A prototype is **step 1 of 7**. Then come auditability, a human in the loop, proven value, a pilot, champions, and quality monitoring.
- Name who uses it, who owns it, who can block it, and who it affects.
- Pitch the conservative case first, and say what has to be true.
- **You build it, you own it:** logs, metrics, runbook, versioned rules.

---

## System Review

**A companion workflow for an app you already have.** The builder takes one idea from "should I build this?" to
"shipped and owned." The System Review looks across everything already built and decides what comes next: new
slices, rescoped parked slices, one-off fixes, data corrections, or nothing. It's a separate skill
([`System Review Copilot.md`](./System%20Review%20Copilot.md)), run before a builder Phase 1, never inside a
builder's Verify (Verify checks one slice's diff; a review checks the whole system).

> **Look before deciding, evidence before opinion, one decision at a time, and the owner decides.**

| # | Phase | You do | Agent writes |
|---|---|---|---|
| R0 | **Scope** | Pick an area and a mode (Quick or Full) | Capabilities check, repo scan |
| R1 | **Inventory** | Add anything it should look at | Schema, data profile, code paths, all MEASURED (read-only) |
| R2 | **Findings** | Approve the ranked list | Findings with evidence (query result or `file:line`) and options |
| R3 | **Decisions** | Answer one question per finding, picking from suggested answers | Your decision per finding |
| R4 | **Triage** | Confirm data fixes | Change-list entries, builder Phase 1 briefs, decision log |

**Each finding gets one decision:** one-off (a single prompt, about a day) · slice (schema change, new LLM step, new
view, or several steps; goes to builder Phase 1) · fold into an existing slice · data fix (confirmed row by row) ·
park · drop.

**When to run it:** after every 2 released slices or at each keep-or-stop date (builder Phase 6 schedules it), before
resuming a parked slice (Quick mode on its area), and whenever something surprises you in daily use.

**Start line:**
```text
Read docs/system-review.md and act as the System Reviewer. Start R0 in Quick mode.
```
In Claude Code, save it as `.claude/skills/system-review/SKILL.md` and run `/system-review`.

---

## Files

| File | Purpose |
|---|---|
| [`SWE AI Builder Copilot.md`](./SWE%20AI%20Builder%20Copilot.md) | The builder agent's full instructions |
| [`System Review Copilot.md`](./System%20Review%20Copilot.md) | The System Review agent's full instructions (separate skill) |
| `README.md` | This guide |
