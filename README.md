中文介绍在后面

# Spec-Kit Coding - OpenClaw Skill Orchestrator

> **Orchestrate the full [github/spec-kit](https://github.com/github/spec-kit) Spec-Driven Development workflow inside OpenClaw.**

This is an **OpenClaw agent skill** that acts as the conductor for 9 sub-skills (speckit-specify, speckit-plan, speckit-implement, etc.). When you say "let's build a new feature with spec-kit-coding", this skill takes over: it checks prerequisites, installs toolchain, initializes the project, and walks through the entire SDD pipeline - from constitution to implementation.

---

## How It Works

```
User: "Start a new project with spec-kit-coding"
                │
                ▼
┌────────────────────────────────────────────────┐
│         spec-kit-coding (Orchestrator)         │
│  • Checks prerequisites                        │
│  • Installs speckit skills (one-time)          │
│  • Initializes project (.specify/ + docs)      │
│  • Routes to the correct sub-skill             │
└───────┬───────────────────────────┬────────────┘
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
  speckit-implement
```

### Two Workflow Paths

| Path                          | Steps                                                                                    | When to Use                               |
| ----------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------- |
| **Production** (8-step) | constitution → specify → clarify → checklist → plan → tasks → analyze → implement | Complex, ambiguous, or high-risk features |
| **Lean** (4-step)       | specify → plan → tasks → implement                                                    | Simple, well-understood features          |

The skill auto-selects the path based on requirement clarity. Each feature tracks its own phase independently in `DEVLOG.md`, so interleaving work across features causes no conflicts.

---

## Usage

## Install

```bash
# Place the spec-kit-coding skill directory into OpenClaw's skills folder
cp -r spec-kit-coding/ ~/.openclaw/workspace/skills/
```

Restart OpenClaw (or reload skills). Before first use, you'll be guided to install the [github/spec-kit](https://github.com/github/spec-kit) CLI if not present.

## Usage

Once installed, just tell the agent:

> "Start a new project at ~/my-project with spec-kit-coding"

The orchestrator checks prerequisites, installs missing sub-skills (one-time), initializes the project, and creates `README.md` + `DEVLOG.md`.

Then:

> "Add a user login feature"

It routes to `speckit-specify`, which produces `spec.md`. After review, continue through the pipeline — the agent handles routing automatically.

---

## Related

- **[github/spec-kit](https://github.com/github/spec-kit)** - The official Spec-Kit toolchain
- **[Spec-Kit Quickstart](https://github.github.io/spec-kit/quickstart.html)** - Workflow docs
- **[OpenClaw](https://docs.openclaw.ai)** - How skills work

> This is an **OpenClaw agent skill**, not a standalone CLI. It requires OpenClaw to run.

## License

MIT

---

# Spec-Kit Coding - OpenClaw 技能编排器

> **在 OpenClaw 中编排 [github/spec-kit](https://github.com/github/spec-kit) 的完整 Spec-Driven Development 工作流。**

这是一个 **OpenClaw 代理技能**,作为 9 个子技能的指挥者(speckit-specify、speckit-plan、speckit-implement 等)。当你对 AI 说"用 spec-kit-coding 开始一个新特性"时,这个技能就会接管--检查依赖、安装工具链、初始化项目、然后走完从 constitution 到 implement 的整个 SDD 流水线。

---

## 工作原理

```
用户: "用 spec-kit-coding 开始一个新项目"
                │
                ▼
┌────────────────────────────────────────────────┐
│         spec-kit-coding (编排器)                │
│  • 检查依赖                                     │
│  • 安装 speckit 子技能(一次性)                   │
│  • 初始化项目(.specify/ + 文档)                  │
│  • 路由到正确的子技能                            │
└───────┬───────────────────────────┬────────────┘
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
  speckit-implement
```

### 两条工作流路径

| 路径                     | 步骤                                                                                     | 适用场景               |
| ------------------------ | ---------------------------------------------------------------------------------------- | ---------------------- |
| **完整路径**(8 步) | constitution → specify → clarify → checklist → plan → tasks → analyze → implement | 复杂、模糊、高风险特性 |
| **精简路径**(4 步) | specify → plan → tasks → implement                                                    | 简单、清晰明确的特性   |

技能会自动根据需求清晰度选择路径。每个特性独立追踪自己的阶段状态,穿插工作也不会冲突。

---

## 使用方式

## 安装

```bash
# 将 spec-kit-coding 技能目录放至 OpenClaw 的 skills 文件夹
cp -r spec-kit-coding/ ~/.openclaw/workspace/skills/
```

重启 OpenClaw（或重载 skills）。首次使用时会引导你安装 [github/spec-kit](https://github.com/github/spec-kit) CLI（如未安装）。

## 使用方式

安装后，直接告诉代理：

> "用 spec-kit-coding 在 ~/my-project 启动一个新项目"

编排器检查依赖、安装缺失子技能（仅一次）、初始化项目、创建 `README.md` + `DEVLOG.md`。

然后：

> "新增一个用户登录特性"

它会路由到 `speckit-specify`，生成 `spec.md`。审阅后继续走后续流程——代理自动处理路由。

---

## 相关资源

- **[github/spec-kit](https://github.com/github/spec-kit)** - 官方 Spec-Kit 工具链
- **[Spec-Kit 快速入门](https://github.github.io/spec-kit/quickstart.html)** - 工作流文档
- **[OpenClaw](https://docs.openclaw.ai)** - 技能工作原理

> 这是一个 **OpenClaw 代理技能**,不是独立 CLI,需要 OpenClaw 才能运行。

## 许可证

MIT
