# Spec-Kit Coding - OpenClaw Skill Orchestrator

> **Orchestrator for [github/spec-kit](https://github.com/github/spec-kit) SDD workflow inside OpenClaw.**

This skill acts as the conductor for 9 sub-skills (speckit-specify, speckit-plan, speckit-implement, etc.). It checks prerequisites, installs the toolchain, initializes projects, routes features to the correct sub-skill, manages context isolation, and enforces documentation and git discipline.

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
| **Lean** (4-step) | specify → plan → tasks → implement | Simple, well-understood features |

Each feature tracks its own phase independently in `DEVLOG.md`. Interleaving features is safe.

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

Routes to `speckit-clarify` on the existing spec.

---

## Related

- **[github/spec-kit](https://github.com/github/spec-kit)** — Official Spec-Kit toolchain
- **[Spec-Kit Quickstart](https://github.github.io/spec-kit/quickstart.html)** — Workflow docs
- **[OpenClaw](https://docs.openclaw.ai)** — How skills work

> This is an OpenClaw agent skill, not a standalone CLI.

## License

MIT

---

# Spec-Kit Coding - OpenClaw 技能编排器

> **在 OpenClaw 中编排 [github/spec-kit](https://github.com/github/spec-kit) 的完整 SDD 工作流。**

本技能是 9 个子技能（speckit-specify、speckit-plan、speckit-implement 等）的指挥者。负责检查依赖、安装工具链、初始化项目、路由特性到正确子技能、管理上下文隔离、执行文档和 git 规范。

**核心原则**：不要求用户手动管理上下文——技能通过子会话 spawn 自动处理。

---

## 工作原理

```
用户: "用 spec-kit-coding 新增一个特性"
                │
                ▼
┌──────────────────────────────────────────────────────┐
│              spec-kit-coding (编排器)                 │
│  • 检查依赖 / 安装子技能（一次性）                     │
│  • 初始化项目（.specify/ + 文档）                     │
│  • 路由到正确子技能（specify vs clarify）             │
│  • 管理上下文（implement 阶段 spawn 子会话）          │
│  • 执行文档优先 + git 规范                            │
└───────┬───────────────────────────┬──────────────────┘
        │                           │
        ▼                           ▼
  新特性                        修改已有特性
  ──────────────────           ──────────────────
  speckit-specify               speckit-clarify
  speckit-clarify  ──(可选)─────┘
  speckit-checklist(可选)       (后续路径相同)
  speckit-plan
  speckit-tasks
  speckit-analyze  (可选)
  speckit-implement ── spawn 独立子会话
```

### 两条工作流路径

| 路径 | 步骤 | 适用场景 |
|------|------|----------|
| **完整路径**（8 步） | constitution → specify → clarify → checklist → plan → tasks → analyze → implement | 复杂、模糊、高风险特性 |
| **精简路径**（4 步） | specify → plan → tasks → implement | 简单、清晰明确的特性 |

每个特性在 `DEVLOG.md` 中独立追踪阶段状态，穿插工作不会冲突。

---

## 使用方式

### 安装

```bash
cp -r spec-kit-coding/ ~/.openclaw/workspace/skills/
```

重启 OpenClaw 或重载 skills。首次使用会引导安装 `specify` CLI。

### 使用

```
"用 spec-kit-coding 在 ~/my-project 启动一个新项目"
```

编排器检查依赖、安装子技能（一次性）、初始化项目、创建 `README.md` + `DEVLOG.md`。

```
"新增一个用户登录特性"
```

路由到 `speckit-specify` → 生成 `spec.md`。审阅后继续走后续流程。

```
"修改用户登录，加 2FA"
```

路由到 `speckit-clarify`，对已有 spec 执行。

---

## 相关资源

- **[github/spec-kit](https://github.com/github/spec-kit)** — 官方 Spec-Kit 工具链
- **[Spec-Kit 快速入门](https://github.github.io/spec-kit/quickstart.html)** — 工作流文档
- **[OpenClaw](https://docs.openclaw.ai)** — 技能工作原理

> 这是 OpenClaw 代理技能，不是独立 CLI。

## 许可证

MIT
