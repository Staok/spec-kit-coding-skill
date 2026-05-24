这个 Skill 在 ClawHub: [Spec-kit Coding — ClawHub](https://clawhub.ai/staok/spec-kit-coding)。

------

# Spec-Kit Coding - OpenClaw 技能编排器

> **在 OpenClaw 中编排 [github/spec-kit](https://github.com/github/spec-kit) 的完整 SDD 工作流。**

本技能是 8 个 speckit 子技能（speckit-constitution、speckit-specify、speckit-implement 等）的指挥者。负责检查依赖、安装工具链、初始化项目、路由特性到正确子技能、管理上下文隔离、执行文档和 git 规范。

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

| 路径                 | 步骤                                                         | 适用场景               |
| -------------------- | ------------------------------------------------------------ | ---------------------- |
| **完整路径**（8 步） | constitution → specify → clarify → checklist → plan → tasks → analyze → implement | 复杂、模糊、高风险特性 |
| **精简路径**（5 步） | specify → clarify → plan → tasks → implement                 | 简单、清晰明确的特性   |

每个特性在 `DEVLOG.md` 中独立追踪阶段状态。

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

根据修改类型路由：参数微调 → 编辑 spec.md；歧义补全 → `speckit-clarify`；实质性变更 → 重跑 `speckit-specify`。详见 SKILL.md §4 分级。

---

## 相关资源

- **[github/spec-kit](https://github.com/github/spec-kit)** — 官方 Spec-Kit 工具链
- **[Spec-Kit 快速入门](https://github.github.io/spec-kit/quickstart.html)** — 工作流文档
- **[OpenClaw](https://docs.openclaw.ai)** — 技能工作原理

> 这是 OpenClaw 代理技能，不是独立 CLI。

## 许可证

MIT