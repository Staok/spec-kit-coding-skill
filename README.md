[中文文档](https://github.com/Staok/spec-kit-coding-skill/blob/main/README_zh-cn.md)

------

Skill on ClawHub: [Spec-kit Coding — ClawHub](https://clawhub.ai/staok/spec-kit-coding).

------

# Spec-Kit Coding - OpenClaw Skill Orchestrator

> **Orchestrator for [github/spec-kit](https://github.com/github/spec-kit) SDD workflow inside OpenClaw.**

This skill acts as the conductor for 8 speckit sub-skills (speckit-constitution, speckit-specify, speckit-implement, etc.). It checks prerequisites, installs the toolchain, initializes projects, routes features to the correct sub-skill, manages context isolation, and enforces documentation and git discipline.

**Key principle**: don't tell the user to manage context — the skill handles it automatically via sub-agent spawning.

---

## How It Works

```
User: "Add a new feature with spec-kit-coding"
                │
                ▼
┌──────────────────────────────────────────────────────┐
│              spec-kit-coding (Orchestrator)           │
│  • Checks prerequisites / installs skills (one-time)  │
│  • Initializes project (.specify/ + docs)             │
│  • Routes to correct sub-skill (specify vs clarify)   │
│  • Manages context (spawns sub-agents for implement)  │
│  • Enforces documentation-first + git discipline      │
└───────┬───────────────────────────┬──────────────────┘
        │                           │
        ▼                           ▼
  New Feature                  Modify Existing
  ──────────────────           ──────────────────
  speckit-specify               speckit-clarify
  speckit-clarify  ──(optional)─┘
  speckit-checklist(optional)   (then same path)
  speckit-plan
  speckit-tasks
  speckit-analyze  (optional)
  speckit-implement ── spawns isolated sub-agents
```

### Two Workflow Paths

| Path | Steps | When |
|------|-------|------|
| **Production** (8-step) | constitution → specify → clarify → checklist → plan → tasks → analyze → implement | Complex, ambiguous, high-risk features |
| **Lean** (6-step) | constitution → specify → clarify → plan → tasks → implement | Simple, well-understood features |

Each feature tracks its own phase independently in `DEVLOG.md`.

---

## Usage

### Install

```bash
cp -r spec-kit-coding/ ~/.openclaw/workspace/skills/
```

Restart OpenClaw or reload skills. First use will guide you through installing the `specify` CLI.

### Use

```
"Start a new project at ~/my-project with spec-kit-coding"
```

The orchestrator checks prerequisites, installs sub-skills (one-time), initializes the project, creates `README.md` + `DEVLOG.md`.

```
"Add a user login feature"
```

Routes to `speckit-specify` → generates `spec.md`. Review → continue through the pipeline.

```
"Modify the user login to add 2FA"
```

Routes based on change type: parameter tweak → edit spec.md; ambiguity → `speckit-clarify`; substantive → re-run `speckit-specify`. See SKILL.md §4 for triage.

---

## Related

- **[github/spec-kit](https://github.com/github/spec-kit)** — Official Spec-Kit toolchain
- **[Spec-Kit Quickstart](https://github.github.io/spec-kit/quickstart.html)** — Workflow docs
- **[OpenClaw](https://docs.openclaw.ai)** — How skills work

> This is an OpenClaw agent skill, not a standalone CLI.

## License

MIT

