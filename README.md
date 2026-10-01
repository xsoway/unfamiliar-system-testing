<p align="center">
  <img src="https://img.shields.io/badge/version-1.0-blue" alt="version 1.0">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="license MIT">
  <img src="https://img.shields.io/badge/status-stable-success" alt="status stable">
  <img src="https://img.shields.io/badge/language-EN%20%7C%20%E4%B8%AD-orange" alt="bilingual EN + 中文">
  <img src="https://img.shields.io/badge/runtime-Python%203%2C%20no%20deps-lightgrey" alt="runtime: Python 3, no third-party deps">
</p>

<h1 align="center">unfamiliar-system-testing</h1>

<p align="center">
  <strong>Start testing a system you don't understand yet — minimal model, guided test points, unknown management</strong>
</p>

<p align="center">
  <a href="./README.zh-CN.md">中文</a>
</p>

<p align="center">
  <b>Onboard · Surface unknowns · Generate test points · Estimate risk</b>
</p>

---

## Table of Contents

- [What is it](#what-is-it)
- [Why](#why)
- [Core Concepts](#core-concepts)
- [Structure](#structure)
- [Quick Start](#quick-start)
- [Features](#features)
- [Safety & Design Principles](#safety--design-principles)
- [Validating & Testing](#validating--testing)
- [FAQ](#faq)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Maintainer](#maintainer)

## What is it

`unfamiliar-system-testing` is a **model-agnostic skill** — a self-contained package (`SKILL.md` + prompts + references + evals + validator) that any agent runtime can load (Codex, Claude, and other model-agnostic hosts). It guides a tester who must start testing a system they do not yet understand. Instead of dumping a generic checklist, it builds a *minimal test model* (8 questions), separates **facts / assumptions / unknowns**, generates test points and directions per dimension, and gives cross-business-type fallbacks plus testing-theory techniques — so the tester knows **what to consider, in which direction, and who to ask next**.

It is an **onboarding + test-point heuristic companion**, not an execution engine and not a skill router. When the tester knows exactly which testing type they need (API contract, performance, security, UI automation), `discover-testing` routes them to more specialized `testing-types/*` skills; this skill provides the "understand + generate direction" layer *before* entering those.

## Why

Starting tests on an unfamiliar system usually ends with either a blank page or a wall of irrelevant generic test cases. Both come from the same root cause — **reasoning about a system you haven't verified**:

| Instinct (anti-pattern) | This skill |
|---|---|
| "Learn the whole architecture before testing" | Build a *minimal* model first, test as you learn |
| Copy-paste a generic test-point list | Cut dimensions to the current system; mark "N/A" explicitly |
| Treat oral explanations / old docs as truth | Label them "assumption" and design evidence to verify |
| Pretend certainty without enough info | Write unknowns explicitly, name who can confirm |
| "I think that's wrong" with no basis | Define the oracle (how to judge right/wrong) first |
| Outsource judgment to AI | AI is a challenger/reviewer, never the conclusion source |

## Core Concepts

- **Minimal test model**: answer only the 8 questions needed for the current task (purpose, users, I/O, rules, states, dependencies, failure behavior, max-loss point). You don't need to learn every module on day one.
- **Unknown management**: "I don't know yet" is a working state, not a conclusion. Each gap records *what's missing, who most likely knows, and what observation/experiment verifies it*. Writing the unknown is safer than filling it with an unverified answer.
- **Facts vs assumptions vs unknowns**: oral explanations and old docs are assumptions until turned into observable, reproducible evidence.
- **Oracle-first**: every test direction needs a standard for judging whether a result is wrong. An assertion without an oracle is "I feel it's wrong" and is untrustworthy.
- **Happy path + failure path**: tests must cover not only the correct-operation path but input / permission / dependency / network / state failure paths.
- **Risk priority**: the max-loss question (question 8) is quantified with an impact × likelihood matrix to decide what to test first.

## Structure

```text
unfamiliar-system-testing/
├── SKILL.md                        # activation entry (frontmatter + rules)
├── prompts/unfamiliar-system-testing.md   # full execution spec (8-question model, output structure)
├── agents/openai.yaml              # discovery metadata + implicit invocation
├── references/
│   ├── test-point-dimensions.md    # 10 dimensions of test points, cut to the current system
│   ├── business-type-use-cases.md  # cross-business-type fallbacks (e-commerce/payment/CRM/search/IAM/Agent/...)
│   ├── testing-theory-techniques.md# equivalence/boundary/decision table/state/exploratory/risk-based/...
│   ├── ai-usage.md                 # AI as reviewer/challenger, not conclusion source
│   └── heuristics-oracles.md       # test oracles, SFDPOT, FEW HICCUPPS, risk matrix, charter template
├── evals/
│   ├── eval.yaml                   # evaluation harness (schema v1alpha1, claude_code engine)
│   └── cases/                      # basic-success / edge-incomplete-input / edge-narrow-unknown
└── scripts/validate_skill_package.py  # contract validation + secret/absolute-path scan
```

| Path | Responsibility |
|---|---|
| `SKILL.md` | Activation entry: when to use, positioning, core constraints, execution flow, lazy-load refs, delivery self-check, gotchas. |
| `prompts/unfamiliar-system-testing.md` | Full execution spec: 8-question minimal model, unknown management, direction generation Steps 1–6, oracle-first, output structure, minimum-coverage checklist. |
| `references/test-point-dimensions.md` | 10 dimension categories (function, input validation, state, permission, dependencies, network, performance, compatibility, security, release) with "cut or mark N/A" guidance. |
| `references/business-type-use-cases.md` | Fallback test points per business type (A–H), ⭐ = highest rollover-risk points. Lazy-loaded only when the type matches. |
| `references/testing-theory-techniques.md` | 12 classic techniques; a "which first" table for unfamiliar-system rounds. |
| `references/ai-usage.md` | How to use AI as challenger/reviewer with a baseline; what AI cannot know. |
| `references/heuristics-oracles.md` | Test-oracle question, SFDPOT / FEW HICCUPPS heuristics, 2×2 risk matrix, exploratory charter template, API black-box contract check. |
| `agents/openai.yaml` | Discovery metadata (display name, description, implicit-invocation policy). |
| `evals/` | Rule-based evaluation harness: one success case + two edge cases (incomplete input, narrow unknown). |
| `evals/cases/*.yaml` | Per-case prompt, `must_not_contain`, and rule-based judge. |
| `scripts/validate_skill_package.py` | Validates package contract (required files, headings, lazy-load refs, case wiring) and scans for credential-like content and absolute local paths. |

## Quick Start

**Requirements:** Python 3 (standard library only, no third-party dependencies).

**Validate the package contract**

```bash
python3 scripts/validate_skill_package.py unfamiliar-system-testing
# → PASS: unfamiliar-system-testing package contract
```

**Run the evaluation cases** (with the skill-up runner against your engine):

```bash
skill-up run unfamiliar-system-testing   # or: skill-up run evals/eval.yaml
```

The eval harness has three rule-based cases — `basic-success`, `edge-incomplete-input`, `edge-narrow-unknown` — that lock in the expected "minimal model + unknown management" behavior.

## Features

- **8-question minimal model**: purpose, users, I/O, rules, states, dependencies, failure behavior, max-loss — build only what the current task needs; missing items are marked `Unknown`.
- **Facts / assumptions / unknowns triage**: oral explanations and old docs become assumptions with a designed way to verify them; information gaps are explicit with an owner and an experiment.
- **Failure-path coverage**: input / permission / dependency / network / state directions are expanded from a single happy path.
- **Cross-business-type fallbacks**: e-commerce, payment/accounting, CRM, search, IAM, AI/LLM/Agent, data integration, messaging — lazy-loaded, ⭐ high-risk points marked.
- **Testing-theory toolbox**: equivalence partitioning, boundary value, decision table, state transition, exploratory (charter), risk-based, pairwise, metamorphic, property-based.
- **Oracle-first discipline**: every direction defines how to judge right/wrong; risk is scored with an impact × likelihood matrix.
- **AI as challenger, not source**: prompts and rules to have AI review your plan instead of producing conclusions.
- **Validatable**: a contract validator plus three rule-based eval cases.
- **No secrets**: the validator scans for credential-like text and absolute local paths; examples never carry real keys.

## Safety & Design Principles

- **Never invent business rules.** Without understanding, the skill forbids writing unverified rules as conclusions — unknowns are written, not papered over.
- **Facts before points.** Forcing a model without test points, or points without a model, both fail.
- **Heuristics are fences, not scripts.** The dimension list and business-type fallbacks are triggers to *cut to the current system*, never wholesale copy-paste.
- **AI is a challenger.** Generated content is never adopted as a business rule or conclusion without human judgment.
- **No fabricated content.** The output structure, priorities, oracles, and next steps come from the current system and the tester's evidence, not from borrowed conventions.

## Validating & Testing

- Package contract + secret/absolute-path scan:

  ```bash
  python3 scripts/validate_skill_package.py unfamiliar-system-testing
  # expect: PASS
  ```

- Evaluation harness (rule-based, three cases):

  ```bash
  skill-up run unfamiliar-system-testing
  ```

## FAQ

**Is this a test-execution framework?**
No. It produces a minimal model, test directions, unknowns, and next steps. When a specific testing type is needed, `discover-testing` routes to the dedicated `testing-types/*` skill.

**What if the user gives almost no information?**
The "unknown management" discipline takes over: it outputs a minimal model with gaps, names who can confirm each gap, and proposes an experiment — it never fabricates business rules to look confident.

**Does it cover every business type?**
The fallback covers A–H (e-commerce, payment, CRM, search, IAM, AI/Agent, data integration, messaging). Types are lazy-loaded on match; the high-risk ⭐ points are the safest place to start.

**Why an oracle for every direction?**
On an unfamiliar system, "this looks wrong" without a standard is unreliable. Defining the oracle (consistency, documentation, domain model, history, etc.) first is what makes a test direction meaningful.

**Can AI generate the test plan for me?**
This skill explicitly warns against it when you lack a baseline. It positions AI as challenger/reviewer: you write the first plan, AI attacks it, you judge each suggestion.

## Roadmap

- [x] 8-question minimal test model with facts/assumptions/unknowns triage
- [x] 10-dimension test-point list with cut-or-mark-N/A guidance
- [x] A–H cross-business-type fallback references with ⭐ high-risk points
- [x] Testing-theory toolbox and selection table
- [x] Oracle-first discipline + impact × likelihood risk matrix
- [x] AI-as-challenger guidance and prompt template
- [x] Package contract validator + three rule-based eval cases
- [x] Bilingual README (EN + 中文) and gruvbox-material homepage
- [ ] English copy of the skill body (currently under `skills/en/testing-workflows/unfamiliar-system-testing/`)
- [ ] More business-type fallbacks (e.g. fintech, DevOps) and per-type eval cases

## Contributing

Pull requests welcome. Before opening one, keep the package contract green:

```bash
python3 scripts/validate_skill_package.py unfamiliar-system-testing
```

Follow the skill's own evidence rule: describe what changed, how it was verified, and any risks. Never place real secrets, personal data, or absolute local paths in any file.

## License

[MIT](./LICENSE)

## Maintainer

[xulanzhong](https://github.com/xulanzhong) — reach out via [GitHub Discussions](https://github.com/xsoway/unfamiliar-system-testing/discussions) or the issue tracker (public, no personal email exposed)