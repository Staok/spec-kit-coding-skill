# 本 Skill 与 另外几个编程 Agent 的 对比分析



## 参考的编程 Agent



- 主要编码流程，参考：[github/spec-kit: 💫 Toolkit to help you get started with Spec-Driven Development](https://github.com/github/spec-kit)、[EnzeD/vibe-coding](https://github.com/EnzeD/vibe-coding) 等。以及加入了自己的具体编码惯例和规范。



主力参考，在下面 `分析` 章节里面主要叙述：

- [GanyuanRan/Aegis: Make AI coding agents architecture-aware: baseline-first, evidence-verified, drift-checked, and safe across long tasks.](https://github.com/GanyuanRan/Aegis)。
- [mattpocock/skills: Skills for Real Engineers. Straight from my .claude directory.](https://github.com/mattpocock/skills)。



集各家之长，经过 AI 分析，本 spec-kit-coding 集成了以下各家所长：

- [obra/superpowers: An agentic skills framework & software development methodology that works.](https://github.com/obra/superpowers)。

  [jnMetaCode/superpowers-zh: 🦸 AI 编程超能力 · 中文增强版 — superpowers 完整汉化 + 中国原创 skills，让 Claude Code / Copilot CLI / Hermes Agent / Cursor / Windsurf / Kiro / Gemini CLI 等 AI 编程工具真正会干活](https://github.com/jnMetaCode/superpowers-zh).

- [shanraisshan/claude-code-best-practice: from vibe coding to agentic engineering - practice makes claude perfect](https://github.com/shanraisshan/claude-code-best-practice)。

- [addxai/enterprise-harness-engineering: Enterprise-grade AI Agent Skills for software development, DevOps, SRE, security, and product teams. Compatible with Claude Code, Cursor, Windsurf, Gemini CLI, GitHub Copilot, and 30+ AI coding agents.](https://github.com/addxai/enterprise-harness-engineering)。

- [humanlayer/12-factor-agents: What are the principles we can use to build LLM-powered software that is actually good enough to put in the hands of production customers?](https://github.com/humanlayer/12-factor-agents)。

- [affaan-m/ECC: The agent harness performance optimization system. Skills, instincts, memory, security, and research-first development for Claude Code, Codex, Opencode, Cursor and beyond.](https://github.com/affaan-m/ECC)。

比如：

> - spec-kit 流程。
> - Context Isolation（步骤 6 子 agent 隔离） — 这是 spec-kit-coding 最强的设计决策，Aegis 和 mattpocock 都没有这么严格的实现隔离
> - 文档优先（Documentation-First） — DEVLOG.md + README.md 模板已很完善
>
> - Git 管理 — 每阶段 commit 的纪律
>
> - Coding Standards 检查点 — 语言特定的编码规范集成
>
> - 子 Agent 报告格式 — 结构化的 SUCCESS/PARTIAL/BLOCKED/FAILED 状态机
>
> - HARD CONSTRAINTS — 安全、通信纪律
> - 等等



## 我们应保持的，我们应避免的



应保持的：

- 结构化流水线作业（spec-kit），把日常工作定为几种流水管线进行作业：
  - 开新工程（前面有产品经理、市场等角色 的 充分调研）：就是 本 spec-kit-coding 的 SETP 从头到尾。
  - 修 BUG：就是 本 spec-kit-coding 的 Bug Fix Entry Point。
  - 添加新 feature：就是 本 spec-kit-coding 的 STEP 5: Spec-Kit Phases / New Feature。
  - 修改 feature：就是 本 spec-kit-coding 的 Feature Modification Entry Point

- skill 简短有力，有效。



> 应避免的：
>
> - Aegis 的过度工程化（7 层诊断对多数项目过度）
>
> - mattpocock 的完全去流程化（我偏好 structured workflow，不完全是独立小技能）
>
> - 任何需要额外 daemon/后台进程的方案
>
> - 破坏现有 spec-kit 兼容性的改动



## 分析 GanyuanRan/Aegis 和 mattpocock/skills



> 两个工程在 OpenClaw 的安装方式
>
> ### Aegis 安装（在 OpenClaw 中）
>
> Aegis 有专门的 [OpenClaw 安装文档](https://github.com/GanyuanRan/Aegis/blob/main/docs/README.openclaw.md)。核心：Aegis 是 multi-skill method-pack，每个 skill 单独安装：
>
> 复制
>
> ```bash
> git clone https://github.com/GanyuanRan/Aegis.git ~/.openclaw/aegis
> 
> # 按需安装各个 skill
> openclaw skills install ~/.openclaw/aegis/skills/using-aegis --as using-aegis --global
> openclaw skills install ~/.openclaw/aegis/skills/systematic-debugging --as systematic-debugging --global
> openclaw skills install ~/.openclaw/aegis/skills/writing-plans --as writing-plans --global
> openclaw skills install ~/.openclaw/aegis/skills/verification-before-completion --as verification-before-completion --global
> openclaw skills install ~/.openclaw/aegis/skills/brainstorming --as brainstorming --global
> # ... 按需加更多
> ```
>
> 验证：`openclaw skills list` 能看到这些 skill。Aegis 状态说明 OpenClaw 是 "Structural Agent Skills / SKILL.md skill-host adaptations; release-level fresh host smoke is still pending"。
>
> ### mattpocock/skills 安装
>
> 官方方式通过 `skills.sh`：
>
> 复制
>
> ```bash
> npx skills@latest add mattpocock/skills
> ```
>
> 这会启动交互式选择器，让你勾选需要的 skill + 选择要安装到哪些 coding agent。然后在新 session 里运行 `/setup-matt-pocock-skills` 做 per-repo 配置。



### 引入的优势



STEP 4



> **用 mattpocock 的 `grill-with-docs` skill**
>
> **它做什么**（来自 SKILL.md）：
>
> - Agent **一次一个问题**采访你，沿着设计树每个分支深入
> - 每个问题给出推荐答案，等你反馈
> - 能通过读代码库回答的问题直接读代码，不问人
> - 挑战你对模糊术语的使用（"你说 'account' — 是 Customer 还是 User？两者不同"）
> - 用具体场景探边界条件
>
> **生成产物**：
>
> 1. **CONTEXT.md**（领域词汇表/术语表）— 每次术语确定后当场写入，格式见 `CONTEXT-FORMAT.md`
> 2. **ADRs**（架构决策记录，节制使用）— 只在同时满足三个条件时创建：难以逆转 + 没有上下文会很惊讶 + 是真实权衡的结果
>
> **在 spec-kit-coding 工作流中的位置**：STEP 4，clarify 之前。做完 grill 后，clarify 要检查的 spec 文档就有了更精确的语言基础。
>
> 
>
> mattpocock/skills 的 CONTEXT.md 管"长期项目理解"。
>
> **CONTEXT.md 的核心优势**（Matt Pocock 原话："this might be the single coolest technique in this repo"）：
>
> 1. **Token 大幅节省**：agent 不需要每次用 20 个词描述同一个概念，1 个术语就够了。session 复 session 积累节省
> 2. **命名一致性**：变量、函数、文件名都用同一种语言，代码库对 agent 更易导航
> 3. **agent 思考更高效**：有了精确术语，推理链更短。例如 `materialization cascade` 代替 "when a lesson inside a section of a course is made real (i.e. given a spot in the file system)"
> 4. **新人（新 agent session）入坑快**：读完 CONTEXT.md 就懂项目语言



STEP 6.2 Testing



> **mattpocock-diagnose** 它的 6 阶段循环就是我报告中描述的：
>
> > Phase 1: 构建反馈循环 → Phase 2: 复现 → Phase 3: 假设（3-5 个可证伪假设）→ Phase 4: 定位 → Phase 5: 修复+回归测试 → Phase 6: 清理+复盘
>
> 当前 spec-kit-coding 的 STEP 6.2 Testing 里引用了 `superpowers-systematic-debugging`，mattpocock 的 diagnose 更实战导向（"Build feedback loop FIRST — be aggressive, be creative"）。**两者可以共存**：Aegis 的 systematic-debugging 偏架构诊断（7 层模型），mattpocock 的 diagnose 偏实战反馈循环。



（这个暂不用，太细）

> **TDD 垂直切片指导**
>
> 用 mattpocock 的 `tdd` skill
>
> **它做什么**（来自 SKILL.md）：
>
> 核心哲学：
>
> - 测试应通过**公共接口**验证行为，而非实现细节
> - **好测试** = 集成风格，读起来像规格说明书
> - **坏测试** = 耦合实现，mock 内部协作者，测试私有方法
>
> **反模式警告**：水平切片 = 先写所有测试再写所有代码。这产生垃圾测试。
>
> 正确做法（垂直切片 / tracer bullet）：
>
> 复制
>
> ```
> WRONG: RED: test1-5  → GREEN: impl1-5
> RIGHT: RED→GREEN: test1→impl1, test2→impl2, ...
> ```
>
> **工作流**：
>
> 1. Planning — 确认公共接口、行为优先级
> 2. Tracer Bullet — 先写一个测试验证端到端通路
> 3. Incremental Loop — 逐个行为：红→绿→红→绿
> 4. Refactor — 全绿后重构，每次重构后跑测试
>
> **在 spec-kit-coding 中的位置**：STEP 6.2 Testing。用它替代/增强我们当前的测试策略描述。



Step 6.3 Final Review

> **Repair/Retirement 报告**
>
> Aegis 把它内嵌在 **`verification-before-completion`** skill 的 **Complexity Delta** 段落里，不是独立 skill。
>
> **它生成什么**（来自 Aegis verification-before-completion SKILL.md）：
>
> 复制
>
> ```text
> Complexity Delta:
> - Files over 800 lines:
> - Files newly crossing 800 lines:
> - Largest touched file delta:
> - Largest touched function/block:
> - New branches/fallbacks/adapters:
> - Retired branches/fallbacks/adapters:    ← 这就是 Retirement Track
> - Net entropy: decreased | stable | increased-with-justification
> - Required follow-up:
> 
> Complexity Governance Suggestion:
> - Recommendation: none | monitor | schedule-refactor | extract helper | split owner | open follow-up
> - Why:
> - Suggested scope:
> - Timing:
> ```
>
> **我们的做法**：在 spec-kit-coding 的**子 Agent 报告格式**和 **Step 6.3 Final Review** 中内嵌这个检查逻辑：
>
> 子 agent 报告增加 "Retired Code / Retained Paths" 字段
>
> Step 6.3 增加 Complexity Delta 检查（800 行文件 / 80 行块 / 新旧分支比）
>
> **Complexity Delta**
>
> Aegis 的 **`verification-before-completion`** skill。不是独立 skill，是它的一个内嵌检查段落。
>
> **生成产物**（同上第 4 点）：Complexity Delta + Complexity Governance Suggestion。
>
> **阈值规则**：
>
> - 文件超 800 行 → review signal
> - 块超 80 行 → review signal
> - 新分支/fallback/adapter 增长但无退役 → 警告
> - Net entropy 判定：decreased / stable / increased-with-justification
>
> **推荐等级**：none → monitor → schedule-refactor → extract helper → split owner → open follow-up
>
> **我们的做法**：在 Step 6.3 Final Review 中加入这个检查逻辑，作为代码审查后的熵审计。
>
> ！！！
>
> 不过，按照我的规范进行上面实际嵌入时候的修改：
>
> - 单文件最多尽量控制在 1200 行以内 或 左右。
> - 函数最多尽量控制在 100 行以内。一个函数执行功能保持单一。
> - 每一行最好不要超过100个字符在，超过部分选择在合适位置换行。



Step 7 Delivery Check

> **Evidence Card**
>
> 来自 Aegis 的 **`verification-before-completion`** skill。这是它的核心输出。
>
> **生成产物**：
>
> ```text
> Evidence Card:
> - Command / Check:         ← 精确验证命令
> - Exit Status:             ← 退出码
> - Covered:                 ← 已验证范围
> - Not Covered:             ← 未验证范围
> - Residual Risk:           ← 残余风险
> - Confidence: A | B | C    ← 置信等级
> ```
>
> **置信等级**：
>
> - **A**：直接验证 + 回归验证，无未知因素
> - **B**：直接验证，有边界风险
> - **C**：仅部分验证，未闭合
>
> 另外还触发 **Red Flags**：agent 用了 "should", "probably", "seems to", "Great!", "Done!" 等词但没跑验证 → STOP。
>
> **我们的做法**：在 Step 7 Delivery Check 中加入 Evidence Card 要求。



> 关于 `verification-before-completion` 完整 skill 有 12 个要点，我们已覆盖关键项：
>
> | 要点                                             | 状态       | 位置                                                  |
> | ------------------------------------------------ | ---------- | ----------------------------------------------------- |
> | **Complexity Delta**                             | ✅ 已用     | STEP 6.3                                              |
> | **Evidence Card + 置信等级 A/B/C**               | ✅ 已用     | STEP 7                                                |
> | **Red Flags**（should/probably/seems to = STOP） | ✅ 已用     | STEP 7 Evidence Card 段                               |
> | QA Closure（副作用清理、evidence bundle）        | ✅ 间接覆盖 | STEP 7 checklist（source tree clean, all tests pass） |
> | Readiness Summary                                | ✅ 间接覆盖 | STEP 7 checklist 已覆盖 tests/docs/version 等         |
> | Goal Closure                                     | ❌ 不适用   | 没用 Aegis goal-framing                               |
> | Workspace Integrity                              | ❌ 不适用   | 没用 Aegis workspace                                  |
> | Long-Task checkpoint                             | ❌ 不适用   | 我们的 batch 模型不需要                               |
> | Natural Aegis closeout                           | ❌ 跳过     | Aegis 特有的方法论                                    |
>
> **结论**：Complexity Delta + Evidence Card + Red Flags 三件套已经覆盖了 verification-before-completion 最核心的价值。其余要么已被我们现有机制覆盖，要么是 Aegis 特有的 infra 概念（workspace、goal-framing），不需要引入。