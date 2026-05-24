---
name: spec-kit-coding
description: "Orchestrator for GitHub Spec-Kit SDD workflow in OpenClaw. Use when starting a new project with spec-driven development, setting up spec-kit toolchain, or running through the full SDD pipeline."
---
## ⚡ Quick Reference Card

### Paths

→ §4 (Production 8-step vs Lean 5-step)

### Hard Constraints

→ §0 (feature splitting, docs-first, context isolation, git policy)

### Checkpoints

→ §Checkpoint A (coding standards, before constitution)

→ §Checkpoint B (UI skill, before plan)

### Post-Implement Gates

→ §6 (code review → testing → final review → delivery, each in fresh session)

### Key Ref

spawn template → §5.2 | troubleshoot → §7 | skill list → §8 | aux skills → §9

---

# Spec-Kit Coding Skill — OpenClaw Orchestrator

This skill orchestrates the complete Spec-Driven Development workflow via
[github/spec-kit](https://github.com/github/spec-kit). It handles toolchain
checks, one-time skill installation, per-project initialization, and generates
project docs (README.md, DEVLOG.md).

## Scope

spec-kit-coding covers **forward coding only** — the pipeline from
constitution through implement, plus post-implement quality gates
(code review → testing → debugging → final review, see Section 6).
It does NOT cover:

- Requirements discovery
- Operations / Deployment
- Cross-domain coverage (SRE, security, data analytics, product)

---

## 0. Communication & Safety Rules (Hard Constraints)

These rules apply to ALL interactions during spec-kit workflows.
**These are global constraints — do NOT require the user to repeat them.**

### Security & Privacy

- **Do NOT transmit sensitive information to the network.** Generally forbidden.
- Before any sensitive action (external API calls, sending data outside local machine, etc.), explain and ask for approval first.

### Feature Splitting

- **Each `/speckit-specify` invocation creates exactly ONE feature.** If the user describes a large, messy, or multi-concern requirement, do NOT silently treat it as one spec.
- Analyze the user's description, identify natural boundaries, and present a structured breakdown:
  - List each proposed feature with a short name and one-line summary
  - Note any dependencies between features
  - Ask user to confirm the split before proceeding
- Remind the user: splitting features well saves rework, keeps specs focused, and prevents specs from fighting each other.

### Communication Style

- **Collect all unclear points first, then ask once.** Avoid back-and-forth fragmented questioning.
- Be efficient and concise. Output only necessary information.
- When the user's description is ambiguous or disorganized, present your organized analysis and wait for confirmation before proceeding.
- Remind the user how to think about the problem better, and help them improve prompt quality over time.

### Tool Usage (Hard Constraint)

- **Do NOT abuse system modifications or install third-party libraries without asking.**
  If a new dependency is needed, explain why and ask for approval first.
- **Prefer reusing existing, proven, popular third-party solutions.**
  Avoid reinventing the wheel — it wastes tokens and introduces bugs.
  Look for well-maintained libraries before writing custom implementations.
- Keep tool usage simple and lean. Minimize dependency footprint.

### Skill References

Skills referenced by name throughout this document (e.g., `speckit-specify`,
`superpowers-requesting-code-review`, `ecc-cpp-coding-standards`) are listed
with full paths and sources in **§8** (speckit skills) and **§9** (auxiliary skills).

### Documentation-First Workflow (Hard Constraint)

- **For spec/plan phase docs (spec.md, plan.md, tasks.md): update BEFORE coding.**
  These are created via the speckit-* phases prior to implementation.
- **For implementation-result docs (DEVLOG.md): update per batch
  during implementation** (see §5.1), to track incremental progress.
  README.md Architecture is seeded during the plan phase with the intended
  design, and updated by the orchestrator after all implementation completes
  to reflect what was actually built (see §5.3).
- Documentation must always reflect the latest state of the project.
- Workflow is split into two levels:

  **Feature-level** (once per feature, before any code):

  1. Generate/update spec/plan docs via speckit-* phases
     → If git management is enabled: git commit

  **Per-change** (every code change within a feature):

  1. Implement code changes
  2. Test and stabilize
  3. Update DEVLOG.md to reflect what was actually built
     → If git management is enabled: git commit
- Do NOT reverse either level's order.

### Git Management (Hard Constraint)

- **Default: ASK the user during project initialization** (see §2.1) whether
  to enable git management. The orchestrator must NOT silently skip this step.
- If the user chooses "no": do not create or manage a git repository.
- If the user chooses "yes": run `git init`, create `.gitignore`, and an
  initial commit; from then on, all phase completions get a git commit.
- The user may also request git management at any later point (mid-feature,
  etc.). Once enabled, keep it enabled for the remainder of the project.
- When git management is enabled, ALL git commits throughout the spec-kit
  workflow follow this pattern: "if git management is enabled, git commit".
  Never assume a commit step without checking this flag.

---

## 1. Check Prerequisites

### ⚠️ Before First Run

The setup script will:

- Install `specify` CLI via `uv` or `pipx` (system-wide Python tool)
- Clone multiple third-party git repos and download ~40 auxiliary skills from GitHub

These are **one-time setup operations** and will not be repeated after initial installation.

**Security note**: The auxiliary skills downloaded from external GitHub repos
(see §8 and §9 for full list and sources) are **not pre-vetted**. Review them
with the `skill-vetter` protocol before trusting in production workflows.

**Ask the user for confirmation before running `setup.sh` for the first time.**

### Run Setup

```bash
bash ~/.openclaw/workspace/skills/spec-kit-coding/setup.sh
```

Options:

- `bash setup.sh --check-only` — only check, don't install
- `bash setup.sh --force` — force reinstall everything
- `bash setup.sh --help` — show usage

This covers: `specify` CLI, speckit-* skills, and all external auxiliary skills.

**Do NOT proceed** until the script reports all dependencies ready.

---

## 2. Initialize Project

### 2.1 Ask the user for the project path

Ask: *"Which project directory should I initialize with spec-kit?"*

Default: current working directory (`.`).

### 2.2 Ask whether to enable git management

Ask: *"Should I enable git management? (Recommended for version control.
I'll init the repo, create .gitignore, and commit after each phase.)"*

Record the answer as a boolean flag. If "yes": after `specify init` completes,
run `git init` and create a sensible `.gitignore`. If "no": skip git setup.

### 2.3 Run `specify init` in the project

```bash
cd <project-dir>
specify init --here --integration claude --force --ignore-agent-tools --script sh --no-git
```

Note: `--no-git` may produce a deprecation warning. This is harmless.

### 2.4 Clean up Claude-specific artifacts

```bash
cd <project-dir>
rm -rf .claude CLAUDE.md
```

The `.specify/` directory IS the spec-kit scaffolding and MUST be kept
(it contains scripts, templates, and project configuration).

### 2.5 Verify

```bash
test -d <project-dir>/.specify && echo "OK: .specify/ created"
test -d ~/.openclaw/workspace/skills/spec-kit-coding/external-skills/speckit-specify && echo "OK: skills present"
```

---

## 3. Create Project Documentation

### 3.1 Create `README.md`

Ask the user for the project's one-line description. If not provided, use a
reasonable placeholder.

Create `<project-dir>/README.md` with the following structure:

````markdown
# <Project Name>

## Project Introduction

<One-paragraph overview of the project, its purpose, and target users.>

## Key Features

<!-- Completed features (use `- ` list, NOT checkboxes).
     This section describes what the project DOES today. -->
- <Feature 1>
- <Feature 2>
- ...

## Local Build

### Prerequisites

- <Prerequisite 1>
- ...

### Build Commands

```bash
# Debug
...

# Release
...
```

## Usage Examples

```bash
# Basic usage
...

# With options
...
```

## SPEC Overview

- **Type**: <CLI tool / TUI / library / web service / ...>
- **Language(s) / Version(s)**: <e.g. C++20, Python 3.11; for mixed projects: C++20 (backend) + Python 3.11 (tooling)>
- **Build**: <CMake, cargo, pip, ...>
- **Dependencies**: <key deps only>
- **License**: <MIT, Apache-2.0, ...>

## Architecture

**This section is a living document.** Seed it during the plan phase with
high-level module layout and key design decisions. Sub-agents do NOT modify
Architecture during implementation — it is updated by the orchestrator after
all implementation completes (per Section 5.4).

<Architecture diagram (ASCII art preferred) and description.
Include: high-level component layout, platform abstraction (if cross-platform),
data model summary, and key design decisions.>

### Platform / Component Details

<Break down key subsystems with enough detail that a new developer
can understand the layout without reading all source code.>

## Known Limitations / Issues

- <Limitation 1: what it is and why>
- <Limitation 2>

## Features Plan / TODOs

<!-- Planned/upcoming features (use `- [ ]` checkboxes).
     This section describes what the project WILL DO in the future.
     Move items to Key Features (as `- ` bullets) when implemented. -->
- [ ] <Planned feature or pending task>
- [ ] ...

## Spec-Driven Development Workflow

This project uses [github/spec-kit](https://github.com/github/spec-kit) for
spec-driven development, orchestrated via the **spec-kit-coding** OpenClaw skill.
Refer to that skill for the full workflow — this README documents the project,
not the process.

### Tracking Progress

Development progress is tracked in `DEVLOG.md` — each feature's phase
completion and milestones are logged there, per feature.
````

Fill in sections marked with `<>` based on conversation with the user.

All project-level technical information (overview, architecture, data sources,
known limitations, future plans) belongs in README under the sections shown
above (`SPEC Overview` through `Features Plan / TODOs`). This keeps README
the single source of truth for:

- what the project is and does
- how to build and use it
- high-level architecture (for new contributors)
- known limitations/issues and future plans

This keeps the README the go-to file for both users and developers.

### 3.2 Create `DEVLOG.md`

Create `<project-dir>/DEVLOG.md` with **per-feature progress tracking**.

Each feature tracks its own phase independently. Features do NOT share a flat
progress table — this ensures two `speckit-specify` runs never fight each other.

All dates in DEVLOG.md MUST use `YYYY-MM-DD HH:MM` format.

`````markdown
# Development Log — <Project Name>

## Feature Progress Summary

| Feature | Spec | Clarify | Checklist | Plan | Tasks | Analyze | Implement | Updated |
|---------|------|---------|-----------|------|-------|---------|-----------|---------|
| — | — | — | — | — | — | — | — | — |

Legend: ⬜ pending | 🔄 in-progress | ✅ complete | ⏭️ skipped | ❌ blocked

## Feature Details

[TEMPLATE — copy this block for each new feature. When copying:
  - All phases start as `pending`.
  - After choosing Lean path: mark `speckit-checklist` and `speckit-analyze`
    as `⏭️ skipped` in the copy (clarify is always run, even on lean path).]
<!-- FEATURE BLOCK START -->
### <NNN>-<short-name>

- **Description**: <one-line summary>
- **Current Phase**: <phase>
- **Last Updated**: <date>

**Phase History**:

<!-- Status legend (same as Summary table above): ⬜ pending | 🔄 in-progress | ✅ complete | ⏭️ skipped | ❌ blocked -->
| Phase | Date | Status | Notes |
|-------|------|--------|-------|
| speckit-specify | | ⬜ | |
| speckit-clarify | | ⬜ | |
| speckit-checklist | | ⬜ | |
| speckit-plan | | ⬜ | |
| speckit-tasks | | ⬜ | |
| speckit-analyze | | ⬜ | |
| speckit-implement | | ⬜ | |
<!-- FEATURE BLOCK END -->

## Global Notes

- Constitution: <date completed or pending>
- Project init: <date>
- <Any cross-feature decisions, blockers, or context>
`````

**When to update DEVLOG**:

- After EACH spec-kit phase completes for any feature, update the Feature Detail block (Phase History table + Current Phase + Last Updated).
- **The Summary table is generated from Feature Detail blocks.** After updating any Feature Detail, regenerate the Summary table by reading all Feature Detail blocks and aggregating their current status into the summary. Do NOT manually maintain the Summary table — it is always derived from Feature Details.
- When a feature is re-entered (e.g., modifying an existing spec), add a new row to its Phase History.
- If the user asks to start a new feature before finishing a previous one, that's fine — the per-feature tracking keeps them independent.

### 3.3 Report completion

After everything is done, summarize:

```
## ✅ Spec-Kit project initialized

- `.specify/` — scaffolding (scripts, templates, constitution template)
- `README.md` — project overview + SDD workflow guide
- `DEVLOG.md` — development progress tracker

### Next Steps

> ⚠️ **Tip**: Avoid working on multiple unrelated features simultaneously.
> Each feature touches shared files (README Architecture, source modules),
> and concurrent edits can cause conflicts or confusion. Finish or pause
> one feature before starting another.

Start with: "run speckit-constitution to establish project principles"
Or ask me: "begin speckit-constitution"

**If you already have a feature in mind**, just describe it directly —
constitution will run automatically on first use. No need to explicitly
start speckit-constitution before describing your first feature.
```

---

## 4. SDD Workflow — Two Modes

Per the [official spec-kit quickstart](https://github.github.io/spec-kit/quickstart.html),
there are two workflow modes:

### Production Path (full 8-step, recommended for complex/ambiguous features)

```
constitution → specify → clarify → checklist → plan → tasks → analyze → implement
```

- `clarify` and `checklist` are quality gates that validate requirements before planning
- `analyze` validates cross-artifact consistency before implementation

### Lean Path (5-step, for simple/well-understood features)

```
specify → clarify → plan → tasks → implement
```

- `clarify` is ALWAYS run after specify (production and lean alike) to catch ambiguities
- Skip `checklist` and `analyze` when requirements are straightforward
- `constitution` is only run once at project start (not per-feature)

**IMPORTANT**: `speckit-specify` may generate an internal validation checklist
as part of its own flow, but this is NOT the same as the standalone
`speckit-checklist` step. The standalone `speckit-checklist` generates
domain-specific "Unit Tests for English" — requirement quality checklists —
and must be invoked separately.

### Feature Independence & Per-Feature Tracking

- **Each feature carries its own phase state.** Starting feature 002 does NOT
  affect feature 001's progress. They are tracked independently in DEVLOG.md.
- **Add new features** when the user describes a capability that does NOT
  already exist in any spec directory. Natural language in any language works
  — judge by meaning, not by keyword matching. Indicators include "新增" /
  "add" / "new" / "加个" or any equivalent in any language.
  → Route to: `speckit-specify`. Then continue based on requirement clarity:

  - **Ambiguous requirements**: production path — `specify → clarify → checklist → plan → tasks → analyze → implement`.
  - **Clear, well-understood requirements**: lean path — `specify → clarify → plan → tasks → implement`.
    DEVLOG records a new phase cycle for that feature.
- **Modify an existing feature** when the user explicitly references an
  existing spec or feature name, or describes changing/extending behavior
  already defined in a spec. Indicators include "修改" / "改" / "change" /
  "adjust" or any equivalent in any language. The entry point depends on the
  **type of change**, not a blanket "modify → clarify" rule.

  #### Change-Type Triage for Modifications

  When the user wants to modify an existing feature, assess the change type
  and route accordingly(if unclear, ask concisely):

  | Tier | Type               | Examples                                        | Route                                                                                                                                                      |
  | ---- | ------------------ | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | 1    | Parameter/Constant | timeout 30s→60s, max retries 3→5              | Edit `spec.md` → `clarify` (parameter changes can expose latent ambiguities — clarify always runs after any spec edit) → plan → tasks → implement |
  | 2    | Ambiguity/Gap      | "handle errors" unspecified, missing edge cases | `speckit-clarify` → plan → tasks → implement                                                                                                          |
  | 3    | Substantive        | new OAuth login, REST→WebSocket, new roles     | Re-run `speckit-specify` → full pipeline                                                                                                                |

  DEVLOG records a new phase cycle regardless of tier.
- **Fixing bug** please refer to the "Post-Implementation Workflow" section.

**HARD RULE — when uncertain, ASK:** If the user's wording could reasonably
be interpreted EITHER way (new feature OR modification), do NOT guess.
Present both options concisely and ask:

> Is this a new feature (via `/speckit-specify`) or a modification to an existing one?
> If modification, which type: parameter tweak / ambiguity fix / substantive change?

Do not proceed until the user confirms.

**Tie-breaking defaults** (use ONLY when user is unavailable and the intent
is 80% clear from context):

- "新增" / "add" / "new" / "加个" → new feature
- "修改" / "改" / "change" / "adjust" → modify existing
- User describes behavior that no existing spec covers → new feature
- User describes changing an existing behavior → modify

**Important**: if the user's description mixes new features and modifications,
separate them first. Ask the user to confirm the split, then handle each
independently. Never silently merge a modification into a new spec or
vice-versa.

### Coding Standards Integration (Hard Constraint)

At two checkpoints in the workflow, prompt the user for coding standards
documents. If the user already provided or attached these documents earlier in
the conversation, use them directly — do NOT re-ask. Skip the prompt only if
the document is already in hand.

#### Checkpoint A: Before constitution

**Timing**: BEFORE `speckit-constitution`. The orchestrator collects BOTH
top-level coding guidance AND coding style conventions from the user in
ONE combined prompt, and passes them into the constitution prompt so the
generated `constitution.md` includes both from birth.

Since `constitution.md` is project-wide (not per-feature), all coding
principles and conventions persist across all features.

**What to collect (both categories in ONE prompt)**:

1. **Top-level coding guidance** — architecture principles: layered
   architecture, modular design, parameterization, cross-platform strategy,
   performance/robustness requirements, design pattern principles, testability.
2. **Coding style / encoding format** — implementation-level conventions:
   naming, formatting, comment style (Doxygen/English), class/function
   templates, clang-format config.

**If user has provided documents/URLs/inline text earlier**: use them directly
in the constitution prompt (reference path for files, inline text otherwise).
Do NOT re-prompt. Do NOT copy file content — reference the path.

**If not provided**: FIRST use the language(s) already recorded in
`README.md` → `SPEC Overview` → `Language(s) / Version(s)` (set during project
initialization, §3.1), or detect from existing code if applicable.
**For multi-language projects** (e.g. C++ backend + Python tooling):
collect standards for EACH language in ONE combined prompt. THEN ask
ONE combined question that covers all relevant standards for all detected languages:

> Use built-in coding standards as constitution reference?
>
> Architecture & Design (all languages):
> `spec-kit-coding/CodingGuidance/TopLevelCodingGuidance.md`
>
> <If C++ is one of the project languages:>
> Coding Style:
> `spec-kit-coding/CodingGuidance/CppCodingStyle.md`
> `spec-kit-coding/CodingGuidance/CppEngineeringFrameworkReference/`
> `spec-kit-coding/CodingGuidance/DesignPattern/`
>
> Language-specific standards (`external-skills/`):
> <For each detected language, list relevant ecc-* skills, e.g.:>
> <C++: `ecc-cpp-coding-standards` + `ecc-cpp-testing`>
> <Python: `ecc-python-patterns` + `ecc-python-testing`>
> <Rust: `ecc-rust-patterns` + `ecc-rust-testing`>
> <... etc.>
>
> `ecc-coding-standards` (general conventions, all languages)

- **Yes**: include all selected reference paths in the prompt. Do NOT copy content.
- **No**: generate concise generic guidance + style rules inline in the prompt.
- **Partial**: user may pick specific items — respect the selection.

**If reply is ambiguous**: Ask one clarifying question — which documents to
reference, or provide informal remarks.

#### Checkpoint B: Before plan

**Timing**: BEFORE `speckit-plan`. After constitution.md is established, the
orchestrator checks whether UI/frontend skills should be loaded for planning.

Only prompt for UI skill — coding standards were already collected in
Checkpoint A and are in constitution.md.

##### UI Skill Check (All Project Types)

If the project involves UI/frontend, remind:

> This project involves UI. The following frontend skills in `external-skills/`
> are available:
>
> - `ui-ux-pro-max-skill` — design system (67 styles, 161 palettes, 57 fonts)
> - `ecc-frontend-patterns` — general frontend patterns
> - `ecc-dart-flutter-patterns` + `ecc-flutter-dart-code-review` (Flutter/Dart)
> - `ecc-swiftui-patterns` (SwiftUI)
> - `ecc-nuxt4-patterns` (Nuxt/Vue)
>   Load relevant ones for frontend implementation?

- **Yes**: the sub-agent reads ui-ux-pro-max during plan/implement.
- **No**: skip.

#### Rules for all checkpoints

- **One prompt each.** Do NOT re-ask if user declines.
- **Use what's given.** Reference files/URLs by path — do NOT copy content.
  Organize inline text before including.
- **Do NOT silently apply defaults.** Always ask before using built-in defaults.
  If declined, generate generic content in the target document.
- **Do NOT modify `CodingGuidance/`.** Read-only except during skill updates.
- **Do NOT fabricate.** No invented standards beyond user-provided or built-in defaults.
- **Cross-feature persistence.** Once user provides/confirms coding standards,
  reuse for all subsequent features. Do NOT re-prompt per-feature.

### 4.1 Session Interrupt & Recovery

If a session is interrupted mid-phase (crash, timeout, user abort):

1. **Do NOT assume** which phase to restart from.
2. **Ask the user**: "Session was interrupted during `<phase>`. Restart from
   `<phase>` or from the previous completed phase (`<prev-phase>`)?"
3. If the user is unsure, **default to re-running the interrupted phase**
   from the start — this is safer than carrying forward a partial result.

---

## 5. Context Management & Execution Strategy

### 5.1 Context Isolation (Hard Constraint)

**LLMs degrade when context usage exceeds 50-60%.** To maintain quality:

- **Lightweight phases** (specify, clarify, checklist, plan, tasks, analyze):
  can run in the orchestrator's current session. Each phase produces artifacts
  on disk that the next phase reads fresh.
- **Heavy phase (implement)**: MUST spawn a fresh isolated sub-agent. The sub-agent starts with empty context and reads only what
  it needs. Do NOT run implement in a session that has accumulated multiple
  prior phases — the degraded output will produce bugs.

**Implementation batching**: Split implementation into batches when a single
session would accumulate too much context. The primary split signal is
**estimated code-generation tool calls**: if a batch would exceed ~12 heavy
code-generation calls, split before it saturates.

Task count is a rough fallback heuristic only — task size varies widely
(a single checkbox item could be 5 lines or 200 lines). If tasks.md contains
more than ~15 task items, the orchestrator should inspect task complexity and
group them into batches of roughly equal code-generation load:

- Each batch = a fresh isolated sub-agent
- After each batch: the sub-agent updates DEVLOG.md; if git management is
  enabled, git commit
- The orchestrator tracks remaining tasks, and spawns the next batch sub-agent

Better to over-split than to produce garbage from context saturation.

### 5.2 Unified Spawn Template

When spawning ANY sub-agent (implement, review, test), the orchestrator
provides file paths only — never pre-digested content. The sub-agent
reads everything fresh.

Use this template, filling in placeholders from the table below:

```
You are <ROLE> for feature <NNN>-<name> in project at <project-dir>.

⚠️ CONTEXT BOUNDARY: You are a fresh isolated session. Focus EXCLUSIVELY on
feature <NNN>-<name>. Disregard any prior memory, context, or assumptions
about this project — the documents below are your sole source of truth.
Do NOT mix in details from other features, other projects, or earlier
implementation batches unless explicitly referenced in these documents.

Before <ACTION>, read these documents in order:
1. <project-dir>/.specify/memory/constitution.md — project principles, must not violate
2. <project-dir>/specs/<NNN>-<name>/spec.md — feature specification
3. <project-dir>/specs/<NNN>-<name>/plan.md — technical implementation plan
4. <project-dir>/specs/<NNN>-<name>/tasks.md — actionable tasks
5. <project-dir>/specs/<NNN>-<name>/analysis.md — cross-artifact consistency findings (if exists)
6. <project-dir>/README.md (Architecture section) — current codebase structure

<ROLE_SPECIFIC_INSTRUCTIONS>

After completing your work:
- Update <project-dir>/DEVLOG.md
- If git management is enabled: git commit all changes
- Report back to orchestrator using the structured format below
```

#### Sub-Agent Report Format

Every sub-agent MUST end its session with a structured report so the
orchestrator can parse the outcome deterministically:

```
## SUB-AGENT REPORT
- **Role**: <implement | code-review | test>
- **Feature**: <NNN>-<name>
- **Status**: <SUCCESS | PARTIAL | BLOCKED | FAILED>
- **Tasks Completed**: <list of task IDs or "all">
- **Tasks Remaining**: <list of task IDs or "none">
- **Issues Found**: <count, with severity breakdown if review/test>
- **Blockers** (if any): <description of what prevents progress>
- **Files Modified**: <list of key files changed>
- **Summary**: <1-2 sentence description of what was accomplished>
```

The orchestrator uses `Status` to decide next action:

- `SUCCESS` → proceed to next gate/batch
- `PARTIAL` → spawn continuation batch for remaining tasks
- `BLOCKED` → escalate to user with blocker details
- `FAILED` → diagnose and decide: retry, rollback, or escalate

| Placeholder                      | Implement                                                                                                                                                                                          | Code Review (§6.1)                 | Test (§6.2)                     |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- | -------------------------------- |
| `<ROLE>`                       | implementing                                                                                                                                                                                       | performing a CODE REVIEW for        | performing TEST DEVELOPMENT for  |
| `<ACTION>`                     | writing any code                                                                                                                                                                                   | reviewing                           | writing any test code            |
| `<ROLE_SPECIFIC_INSTRUCTIONS>` | Implement tasks `<M>`–`<N>` from tasks.md per speckit-implement skill. If the plan is infeasible or conflicts with spec.md, STOP and report to orchestrator with specifics — do NOT proceed. | See review checklist below (§6.1). | See test strategy below (§6.2). |

**Key rules**:

- Provide **file paths only** — sub-agents read fresh
- Documents may be updated between batches — always re-read
- Only difference between implementation batches: task range `<M>`–`<N>`

**What to skip**: The clarify and checklist outputs are NOT re-read for
implementation — `clarify` output is already encoded into spec.md, and
`checklist` is a requirement quality meta-tool not directly actionable for coding.

### 5.3 Architecture Updates During Planning & Implementation

During planning, the Architecture section of `README.md`
is a **living document** — not a one-time artifact.

**During plan phase**: After `speckit-plan` completes, seed `README.md` →
`## Architecture` with:

- High-level component/layer diagram (ASCII art preferred)
- Module responsibility boundaries
- Key design decisions and trade-offs
- Platform abstraction strategy (if cross-platform)

**During implementation**: Sub-agents READ Architecture as context but do NOT
write to it — this prevents isolated sessions from making conflicting edits.
The orchestrator updates Architecture AFTER all implementation completes,
reflecting what was actually built.

If a significant architectural deviation is discovered during implementation,
flag it to the orchestrator: the plan-phase docs may need to be re-run to
realign the design with reality.

---

## 6. Post-Implementation Workflow

After all implementation batches complete (all tasks in tasks.md are ✅),
proceed through the following quality gates. Each gate runs in a **fresh
isolated session** to ensure unbiased review and full context capacity.

### Workflow Overview

```
    implement (all batches done)
                │
                ▼
  ┌─────────────────────────────┐
  │ 6.1 Code Review             │  ← fresh session
  │ requesting-code-review      │
  └─────────────┬───────────────┘
                │
                ▼
  ┌─────────────────────────────┐
  │ 6.2 Testing                 │  ← fresh session
  │ systematic-debugging        │
  └─────────────┬───────────────┘
                │
                ▼
  ┌─────────────────────────────┐
  │ 6.3 Final Review            │
  │ optimization + TODO check   │
  └─────────────┬───────────────┘
                │
                ▼
  ┌─────────────────────────────┐
  │ 6.4 Delivery                │
  │ industrial-grade handoff    │
  └─────────────────────────────┘
```

### 6.1 Code Review (Hard Constraint)

**After all implementation completes**, spawn a fresh isolated sub-agent
session. Use spawn template from §5.2. The reviewer starts clean —
no implementation context.

**Document reading**: Use §5.2 spawn template. The review evaluates the code
against the specification (spec.md), plan (plan.md), and principles
(constitution.md).

The `<ROLE_SPECIFIC_INSTRUCTIONS>` are:

**Review Checklist** (apply `superpowers-requesting-code-review` skill):

1. **Practicality & Generality** — Reusable? Hidden assumptions?
2. **Risk Assessment** — Memory leaks, thread safety, unhandled errors, UB, security?
3. **Optimization** — Algorithmic inefficiencies, redundant allocations, excess copies, unnecessary deps?
4. **Architecture Alignment** — Code matches plan? Architecture decisions documented in README?
5. **Coding Standards** — Follows encoding format spec in constitution.md?

**Output**: Structured review report with:

- Severity: 🔴 Critical / 🟠 Important / 🟡 Minor / 🔵 Suggestion
- Each issue: file:line, description, recommendation
- Overall: Ready / Needs Fixes / Major Rework

After review, report findings back to the orchestrator.

#### After Review

1. Present the review findings to the user.
2. Ask: "Which review findings should be addressed?"
3. Apply ONLY the user-approved fixes.
4. Re-run the review on changed files. If the changes may have cross-file impact (e.g., shared headers, interface changes, common utilities), also review related files that could be affected.
5. If new issues are found in step 4, repeat from step 1 — but **limit to 3 review–fix cycles total**. If issues persist after 3 cycles, log remaining items to `README.md → Features Plan / TODOs` and proceed.
6. If git management is enabled: git commit the review fixes.

### 6.2 Testing (Hard Constraint)

**After code review fixes are applied**, spawn a fresh isolated sub-agent
session for test development and debugging.

**Document reading**: Use §5.2 spawn template (which includes plan.md and
tasks.md for general scope understanding). However, **test case design must
derive from spec.md behavioral requirements** — plan.md and tasks.md are
read only to understand implementation scope and boundaries, NOT to derive
test assertions. This prevents tests from mirroring implementation bugs.

**Additionally**, read ALL source files under the feature's implementation
directory. The sub-agent needs to know actual API signatures, module names,
and function declarations to write concrete tests. Use spec.md for WHAT to
test (expected behaviors), source code for HOW to invoke (actual signatures).

#### Pre-Test: Ask for Test Plan Approval

Before spawning the test session, ask the user to confirm:

```
Please confirm the test plan:

Module-Level: cover all public APIs with normal/boundary/error inputs,
  key API call sequences, and multi-threading safety (if applicable).

Integration: inter-module call testing, full app functional flow.

Coverage (optional): use language-appropriate tools (gcov/C++, pytest-cov,
  cargo-tarpaulin/Rust, Jest --coverage/JS-TS, etc.)
```

**Do NOT proceed** until user confirms or adjusts the plan.

#### Spawn Message Template (Test Session)

Use spawn template from §5.2. The `<ROLE_SPECIFIC_INSTRUCTIONS>` for
test sessions are:

```
### Test Environment
- ALL temporary files, build artifacts, test outputs → /tmp/<project-name>-test/
- Do NOT pollute the source tree

### Test Framework
Choose based on project language:
- **C++**: Google Test (gtest), compile via cmake --build
- **Python**: pytest (`python -m pytest`)
- **Rust**: `#[cfg(test)]` + `cargo test`
- **JavaScript/TypeScript**: Jest (`npx jest`)
- **Go**: `go test ./...`
- **Other**: project's standard test infrastructure

Place test code under standard test dir (`test/`, `tests/`, `__tests__/`).

### Test Strategy
[INSERT USER-APPROVED TEST PLAN HERE]

Encoding format specification / coding conventions are in
constitution.md — see §4 Checkpoint A.

### Module-Level Tests
For EACH module:
1. Cover all public API functions
2. Cover normal, boundary (min/max/empty/zero-length), and error inputs
3. Cover key API call sequences
4. Multi-threading safety (if module supports it): construction/destruction
   in thread pool, concurrent API calls from multiple threads

### Integration Tests
1. Module-to-module interaction tests
2. Full application functional flow tests
3. Specify control/communication method for external dependencies

### Optional: Code Coverage
If user requested: gcov+lcov (C++), pytest-cov, cargo-tarpaulin (Rust),
Jest --coverage (JS/TS), or language-equivalent.

### Debugging
When tests fail, apply `superpowers-systematic-debugging`:
1. Root Cause → 2. Pattern Analysis → 3. Hypothesis → 4. Fix + Verify.
Also reference `mattpocock-diagnose` for feedback loops.

### Iteration
- Run tests → fix → re-run until ALL pass
- If 3+ fix attempts fail → question architecture, report to orchestrator

### After All Tests Pass
1. Update DEVLOG.md with test results
2. Report explicit status: total tests, passed, failed, skipped
3. Report: tests written, coverage, issues found & fixed

**If any test cannot be fixed within 3 attempts**, report to orchestrator with
the failing test details and your diagnosis — do NOT silently skip.
```

#### After Testing

1. Verify the sub-agent's test status report (total/passed/failed/skipped).
2. If all tests pass: proceed to Final Review.
3. If issues remain: iterate with systematic debugging. If 3+ fix attempts
   on the same issue fail, report to user for architectural discussion.

### 6.3 Final Review — Optimization & Code Quality

After testing is complete, perform a final pass:

1. **Re-read all modified source files** in the feature.
2. **Check for optimization opportunities**:

   - Algorithmic improvements (better complexity)
   - Redundant allocations or copies
   - Unnecessary dependencies
   - Dead code or unreachable branches
3. **Record optimization opportunities in `README.md` → `## Features Plan / TODOs`**.
   Do NOT add TODO comments to source code — keep the source tree clean.
   Instead, append each finding as a checkbox item under `Features Plan / TODOs`
   with category and location:

   Format: `- [ ] [<category>] <description> (file: <path>:<line-range>)`

   Common categories: `optimization`, `robustness`, `clarity`, `security`, `perf`.

   Examples:

   ```markdown
   - [ ] [optimization] Consider using unordered_map for O(1) lookup (file: src/cache.cpp:42-55)
   - [ ] [robustness] Add timeout handling for network operations (file: src/client.py:110)
   ```
4. **Present optimization candidates to the user**: From the items added in
   step 3, identify which ones are low-risk / high-impact enough to implement
   immediately (vs. defer). Present a concise list:

   > Optimization candidates found during final review:
   >
   > **Implement now (low risk, high impact):**
   >
   > - [ ]
   >   <item 1>
   > - [ ]
   >   <item 2>
   >
   > **Defer (tracked in README Features Plan / TODOs):**
   >
   > - <item 3>
   >
   > Which "implement now" items should I apply?
   >

   Apply ONLY user-approved items. Deferred items remain in README as
   tracked TODOs. If no candidate is approved, skip to step 6.
5. **Regression Gate (Hard Constraint)**:
   If actual code changes were made during final review:

   - Full clean rebuild (language-appropriate: `cmake --build build --clean-first`
     for C++, `cargo clean && cargo build` for Rust,
     `make clean && make` or equivalent)
   - Run ALL tests using the project's test runner (module-level + integration)
   - If any test fails → return to **§6.2 Testing** for debugging
   - If all tests pass → proceed to **§6.4 Delivery**
   - If 3 cycles of regression → test → debug fail to converge,
     STOP and escalate to user for architectural discussion.
6. **Verify Architecture documentation** is up-to-date in README.md.
7. **If git management is enabled**: git commit.

### 6.4 Industrial-Grade Delivery Checklist

Before declaring the feature complete, verify:

- [ ] All required speckit-* phases completed or skipped (lean path skips
  checklist, analyze — this is expected and OK; clarify is always run)
- [ ] Code review completed and approved fixes applied
- [ ] All tests pass (module-level + integration)
- [ ] Code coverage meets target (if enabled; gcov/C++ or language-equivalent)
- [ ] Source tree is clean (no temp files, no build artifacts in source dirs)
- [ ] README.md Architecture section is up-to-date
- [ ] DEVLOG.md reflects all completed phases
- [ ] Optimization findings tracked in README.md → Features Plan / TODOs
- [ ] All hard constraints from Section 0 respected
- [ ] If git management enabled: all changes committed

---

## 7. Troubleshooting

| Problem                          | Fix                                       |
| -------------------------------- | ----------------------------------------- |
| `specify: command not found`   | Install via uv or pipx (see Section 1)    |
| Skills not in workspace          | Run `bash setup.sh` (see Section 1)     |
| `.specify/` missing in project | Re-run Section 2 in the project directory |
| Scripts not executable           | `chmod +x .specify/scripts/bash/*.sh`   |
| Task references stale spec       | Re-run the relevant speckit-* phase       |

## 8. Reference: Speckit Skill List

These are the skills that get installed to the workspace (one-time):

| Skill Name               | Purpose                                                                    | When to Use                                           |
| ------------------------ | -------------------------------------------------------------------------- | ----------------------------------------------------- |
| `speckit-constitution` | Project principles & governance                                            | §4 — once per project                               |
| `speckit-specify`      | Feature specification (what & why)                                         | §4 — every new feature                              |
| `speckit-clarify`      | Quality gate — catch spec ambiguities. NOT a general-purpose spec editor. | §4 — after specify                                  |
| `speckit-checklist`    | Requirement quality checklist ("Unit Tests for English")                   | §4 — production path, after specify/clarify         |
| `speckit-plan`         | Technical implementation plan                                              | §4 — after specify/clarify/checklist                |
| `speckit-tasks`        | Actionable, dependency-ordered tasks                                       | §4 — after plan                                     |
| `speckit-analyze`      | Cross-artifact consistency analysis                                        | §4 — production path, after tasks, before implement |
| `speckit-implement`    | Execute tasks (batched)                                                    | §5 — after analyze (or after tasks on lean path)    |

---

## 9. Auxiliary Skills Reference

All auxiliary skills live under `spec-kit-coding/external-skills/`.
They are NOT part of spec-kit-coding core but are invoked as needed
in plan phase (language/UI reminders) and post-implement workflow.

| Skill                                        | Source                                                                                       | Purpose                                                                                     |
| -------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `superpowers-requesting-code-review`       | [obra/superpowers](https://github.com/obra/superpowers)                                         | Code review: practicality, risks, optimization                                              |
| `superpowers-systematic-debugging`         | [obra/superpowers](https://github.com/obra/superpowers)                                         | 4-phase debugging: root cause → pattern → hypothesis → fix                               |
| `mattpocock-diagnose`                      | [mattpocock/skills](https://github.com/mattpocock/skills)                                       | 6-phase diagnosis: feedback loop → reproduce → hypothesis → instrument → fix → cleanup |
| `mattpocock-improve-codebase-architecture` | [mattpocock/skills](https://github.com/mattpocock/skills)                                       | Architectural deepening: explore → candidates → grilling                                  |
| `ecc-cpp-coding-standards`                 | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | C++ Core Guidelines enforcement                                                             |
| `ecc-cpp-testing`                          | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | C++ testing patterns (gtest, gmock, coverage)                                               |
| `ecc-python-patterns`                      | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | Pythonic idioms, PEP 8, type hints                                                          |
| `ecc-python-testing`                       | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | Python testing patterns (pytest, coverage)                                                  |
| `ecc-pytorch-patterns`                     | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | PyTorch DL/RL patterns                                                                      |
| `ecc-dart-flutter-patterns`                | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | Dart/Flutter idioms and patterns                                                            |
| `ecc-flutter-dart-code-review`             | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | Flutter/Dart code review standards                                                          |
| `ecc-coding-standards`                     | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | General coding conventions (KISS/DRY/YAGNI)                                                 |
| `ecc-security-scan`                        | [affaan-m/ECC](https://github.com/affaan-m/ECC)                                                 | AgentShield security scan                                                                   |
| `ui-ux-pro-max-skill`                      | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | Full design system: 67 styles, 161 palettes, 57 fonts, 99 UX guidelines                     |

Additional ECC language skills available in `external-skills/`:
`ecc-java-coding-standards`, `ecc-kotlin-patterns`, `ecc-kotlin-testing`,
`ecc-golang-patterns`, `ecc-golang-testing`, `ecc-rust-patterns`,
`ecc-rust-testing`, `ecc-swiftui-patterns`, `ecc-swift-protocol-di-testing`, `ecc-dotnet-patterns`,
`ecc-csharp-testing`, `ecc-fsharp-testing`, `ecc-fastapi-patterns`,
`ecc-nestjs-patterns`, `ecc-nuxt4-patterns`, `ecc-frontend-patterns`,
`ecc-docker-patterns`, `ecc-deployment-patterns`,
`ecc-springboot-patterns`, `ecc-springboot-security`, `ecc-springboot-tdd`,
`ecc-springboot-verification`,
`ecc-api-design`, `ecc-error-handling`, `ecc-git-workflow`, `ecc-accessibility`,
`ecc-postgres-patterns`, `ecc-redis-patterns`.
