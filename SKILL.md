---
name: spec-kit-coding
description: "Orchestrator for GitHub Spec-Kit SDD workflow in OpenClaw. Use when starting a new project with spec-driven development, setting up spec-kit toolchain, or running through the full SDD pipeline."
---
# Spec-Kit Coding Skill — OpenClaw Orchestrator

This skill orchestrates the complete Spec-Driven Development workflow via
[github/spec-kit](https://github.com/github/spec-kit). It handles toolchain
checks, one-time skill installation, per-project initialization, and generates
project docs (README.md, DEVLOG.md).

---

## 0. Communication & Safety Rules (Hard Constraints)

These rules apply to ALL interactions during spec-kit workflows:

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

### Documentation-First Workflow (Hard Constraint)

- **For any new requirement or change: update documentation FIRST, then modify code.**
- Documentation (README.md, DEVLOG.md, spec.md, plan.md, etc.) must always reflect the latest state.
- Workflow order (do NOT reverse):
  1. Update relevant docs
  2. Implement code changes
  3. Test and stabilize
- Do NOT require the user to repeat this rule every time.

---

## 1. Check Prerequisites

**Check if `specify` CLI is available:**

```bash
which specify && specify version
```

### If NOT installed

Tell the user to install the `specify` CLI. Provide the exact installation command:

```
## specify CLI not found

You need the official GitHub Spec-Kit CLI. Choose one:

### Option A: uv (recommended — cross-platform, self-contained)
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@latest

### Option B: pipx
pipx install --force git+https://github.com/github/spec-kit.git@latest

### Verify:
specify version

### Requirements:
- Python >= 3.11
- uv (https://docs.astral.sh/uv/) or pipx (https://pipx.pypa.io/)

More details: https://github.com/github/spec-kit
```

**Do NOT proceed** until `specify version` succeeds. Ask the user to confirm after installation, then re-run the check.

### If installed

Check whether the speckit skills already exist in the OpenClaw workspace:

```bash
test -d ~/.openclaw/workspace/skills/speckit-specify && echo "skills-present" || echo "skills-missing"
```

---

## 2. Install Speckit Skills to OpenClaw (one-time, only if missing)

This step generates the 9 spec-kit SKILL.md files and copies them to the
OpenClaw workspace. Run only once — subsequent projects skip this.

```bash
cd /tmp && rm -rf _spec && mkdir _spec && cd _spec
specify init --here --integration claude --force --ignore-agent-tools
cp -r .claude/skills/speckit-* ~/.openclaw/workspace/skills/
rm -rf /tmp/_spec
```

**Verify installation:**

```bash
ls ~/.openclaw/workspace/skills/speckit-specify/SKILL.md && echo "OK: skills installed"
```

If the copy fails (e.g., `No such file or directory`), re-run the generation
from scratch — the `specify init` step may have failed silently.

---

## 3. Initialize Project

### 3.1 Ask the user for the project path

Ask: *"Which project directory should I initialize with spec-kit?"*

Default: current working directory (`.`).

### 3.2 Run `specify init` in the project

```bash
cd <project-dir>
specify init --here --integration claude --force --ignore-agent-tools --script sh --no-git
```

Note: `--no-git` may produce a deprecation warning. This is harmless.

### 3.3 Clean up Claude-specific artifacts

```bash
cd <project-dir>
rm -rf .claude CLAUDE.md
```

The `.specify/` directory IS the spec-kit scaffolding and MUST be kept
(it contains scripts, templates, and project configuration).

### 3.4 Verify

```bash
test -d <project-dir>/.specify && echo "OK: .specify/ created"
test -d ~/.openclaw/workspace/skills/speckit-specify && echo "OK: skills present"
```

---

## 4. Create Project Documentation

### 4.1 Create `README.md`

Ask the user for the project's one-line description. If not provided, use a
reasonable placeholder.

Create `<project-dir>/README.md` with the following structure:

````markdown
# <Project Name>

## Project Introduction

<One-paragraph overview of the project, its purpose, and target users.>

## Key Features

- <Feature 1>
- <Feature 2>
- ...

## Local Build

```bash
# Prerequisites
# ...

# Build
...

# Run
...
```

## Usage Examples

### Example 1: <description>
```bash
...
```

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

### 4.2 Create `DEVLOG.md`

Create `<project-dir>/DEVLOG.md` with **per-feature progress tracking**.

Each feature tracks its own phase independently. Features do NOT share a flat
progress table — this ensures two `speckit-specify` runs never fight each other.

```markdown
# Development Log — <Project Name>

## Feature Progress Summary

| Feature | Spec | Clarify | Checklist | Plan | Tasks | Analyze | Implement | Updated |
|---------|------|---------|-----------|------|-------|---------|-----------|---------|
| — | — | — | — | — | — | — | — | — |

Legend: ⬜ pending | 🔄 in-progress | ✅ complete | ⏭️ skipped | ❌ blocked

## Feature Details

<!-- Each feature gets its own detail block. Add a new block for each spec. -->

<!-- Template (copy for each feature):
### <NNN>-<short-name>

- **Description**: <one-line summary>
- **Current Phase**: <phase>
- **Last Updated**: <YYYY-MM-DD HH:MM>

**Phase History**:

| Phase | Date | Status | Notes |
|-------|------|--------|-------|
| speckit-specify | | pending | |
| speckit-clarify | | pending | |
| speckit-checklist | | pending | |
| speckit-plan | | pending | |
| speckit-tasks | | pending | |
| speckit-analyze | | pending | |
| speckit-implement | | pending | |
-->

## Global Notes

- Constitution: <date completed or pending>
- Project init: <YYYY-MM-DD HH:MM>
- <Any cross-feature decisions, blockers, or context>
```

**Date/Time Format (Hard Rule)**: ALL date-bearing fields in DEVLOG.md MUST use
`YYYY-MM-DD HH:MM` format (e.g., `2026-05-17 16:30`). This applies to:

- Summary table `Updated` column
- Feature detail `Last Updated` field
- Phase History `Date` column
- Global Notes date fields (Constitution, Project init, etc.)

**When to update DEVLOG**:

- After EACH spec-kit phase completes for any feature, update both the Summary table AND the Feature Detail block.
- When a feature is re-entered (e.g., modifying an existing spec), add a new row to its Phase History and update the Summary table accordingly.
- If the user asks to start a new feature before finishing a previous one, that's fine — the per-feature tracking keeps them independent.

### 4.3 Report completion

After everything is done, summarize:

```
## ✅ Spec-Kit project initialized

- `.specify/` — scaffolding (scripts, templates, constitution template)
- `README.md` — project overview + SDD workflow guide
- `DEVLOG.md` — development progress tracker

### Next Steps

Start with: "run speckit-constitution to establish project principles"
Or ask me: "begin speckit-constitution"
```

---

## 5. SDD Workflow — Two Modes

Per the [official spec-kit quickstart](https://github.github.io/spec-kit/quickstart.html),
there are two workflow modes:

### Production Path (full 8-step, recommended for complex/ambiguous features)

```
constitution → specify → clarify → checklist → plan → tasks → analyze → implement
```

- `clarify` and `checklist` are quality gates that validate requirements before planning
- `analyze` validates cross-artifact consistency before implementation

### Lean Path (4-step, for simple/well-understood features)

```
specify → plan → tasks → implement
```

- Skip `clarify`, `checklist`, and `analyze` when requirements are unambiguous
- `constitution` is only run once at project start (not per-feature)

**IMPORTANT**: `speckit-specify` may generate an internal validation checklist
as part of its own flow, but this is NOT the same as the standalone
`speckit-checklist` step. The standalone `speckit-checklist` generates
domain-specific "Unit Tests for English" — requirement quality checklists —
and must be invoked separately.

### Feature Independence & Per-Feature Tracking

- **Each feature carries its own phase state.** Starting feature 002 does NOT
  affect feature 001's progress. They are tracked independently in DEVLOG.md.
- **You can interleave features**: finish specify for 001 → start specify for
  002 → go back to clarify 001 → implement 002. DEVLOG's per-feature structure
  keeps this manageable.
- **Add new features**: re-enters its workflow at `speckit-specify`.
  The subsequent path depends on requirement clarity:
  - **Ambiguous requirements**: production path — `specify → clarify → checklist → plan → tasks → analyze → implement`.
  - **Clear, well-understood requirements**: lean path — `specify → plan → tasks → implement`.
  DEVLOG records a new phase cycle for that feature.
- **Modifying an existing feature** re-enters its workflow at `speckit-clarify`.
  The subsequent path depends on requirement clarity:
  - **Ambiguous changes**: production path — `clarify → checklist → plan → tasks → analyze → implement`.
  - **Clear, well-understood changes**: lean path — `clarify → plan → tasks → implement`.
    DEVLOG records a new phase cycle for that feature.

### How to Prompt: New Feature vs. Modify Existing

**CRITICAL — Semantic routing, not keyword matching.**
The decision between "new feature" and "modify existing" is based on the
user's INTENT, not on specific English trigger phrases. Natural language
in any language works. Judge by meaning:

**New feature indicators** (any language — these are examples, not an
  exhaustive list):

- "新增一个特性" / "加个功能" / "新需求" / "再加一个"
- "Add new feature" / "add a feature" / "new requirement" / "new capability"
- "I want to also..." (adding something not in any existing spec)
- Any description of a capability that does NOT already exist in any
  spec directory
- The user says "new" or "add" in ANY language → default to new feature

→ Route to: `speckit-specify`. Then continue with the workflow (production or lean based on ambiguity).

**Modify indicators** (any language):

- "修改某个特性" / "改一下" / "调整" / "在已有基础上加"
- "Modify specs/001-xxx" / "change" / "update" / "adjust"
- The user explicitly references an existing spec directory or feature name
- The description clearly extends/replaces behavior already defined in a spec
- The user says "改" or "修改" in ANY language → default to modify

→ Route to: `speckit-clarify` on the existing spec. Then continue with the workflow (production or lean based on ambiguity).

**HARD RULE — when uncertain, ASK:**
If the user's wording could reasonably be interpreted EITHER way, do NOT guess.
Present both options concisely and ask:

> 这是新增一个独立特性（走 `/speckit-specify`），还是在已有特性基础上修改（走 `/speckit-clarify`）？

Do not proceed until the user confirms.

**Tie-breaking defaults** (use ONLY when user is unavailable and the intent is 80% clear from context):

- "新增" / "add" / "new" / "加个" → new feature
- "修改" / "改" / "change" / "adjust" → modify existing
- User describes behavior that no existing spec covers → new feature
- User describes changing an existing behavior → modify

**Important**: if the user's description mixes new features and modifications,
separate them first. Ask the user to confirm the split, then handle each
independently. Never silently merge a modification into a new spec or
vice-versa.

## 6. Troubleshooting

| Problem                          | Fix                                       |
| -------------------------------- | ----------------------------------------- |
| `specify: command not found`   | Install via uv or pipx (see Section 1)    |
| Skills not in workspace          | Re-run Section 2                          |
| `.specify/` missing in project | Re-run Section 3 in the project directory |
| Scripts not executable           | `chmod +x .specify/scripts/bash/*.sh`   |
| Task references stale spec       | Re-run the relevant speckit-* phase       |

## Reference: Speckit Skill List

These are the skills that get installed to the workspace (one-time):

| Skill Name                | Purpose                                                                     | When to Use                                                      |
| ------------------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `speckit-constitution`  | Project principles & governance                                             | Start of project (once)                                          |
| `speckit-specify`       | Feature specification (what & why)                                          | Every new feature                                                |
| `speckit-clarify`       | Clarify ambiguous requirements; entry point for modifying existing features | After specify (new), or to modify an existing spec (before plan) |
| `speckit-checklist`     | Requirement quality checklist ("Unit Tests for English")                    | After specify/clarify, before plan (production path)             |
| `speckit-plan`          | Technical implementation plan                                               | After specify/clarify/checklist                                  |
| `speckit-tasks`         | Actionable, dependency-ordered tasks                                        | After plan                                                       |
| `speckit-analyze`       | Cross-artifact consistency analysis                                         | After tasks, before implement (production path)                  |
| `speckit-implement`     | Execute all tasks                                                           | After analyze (or after tasks on lean path)                      |
| `speckit-taskstoissues` | Convert tasks to GitHub Issues                                              | After tasks (optional)                                           |
