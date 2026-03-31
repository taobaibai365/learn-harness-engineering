# Learn Harness Engineering

> 这是一门项目制课程：系统学习如何通过环境、状态、验证与控制机制，让 Codex 和 Claude Code 更可靠地工作。

本课程仍在持续建设中，内容后续可能会调整。

[英文版](./README.md)

---

## 模型很强，但 Harness 让它靠谱

有一个很多人交过学费才明白的事实：**世界上最强的模型，如果没有一个合适的工作环境，依然会在真实工程任务中翻车。**

你多半见过这种情况。你给 Claude 或 GPT 一个任务，它看起来干得不错——读文件、写代码、很努力。然后出问题了。它跳过了一个步骤。它搞坏了一个测试。它说"完成了"但实际上什么都没跑通。你花在收拾烂摊子上的时间比自己做还多。

这不是模型的问题，这是 harness 的问题。

证据很明确。Anthropic 做过一组对照实验：同一个模型（Opus 4.5），同一段提示词（"做一个 2D 复古游戏编辑器"）。没有 harness 的情况下，20 分钟花了 $9，结果游戏核心功能跑不起来。加上完整 harness（planner + generator + evaluator 三 agent 架构），6 小时花了 $200，做出来的游戏可以正常游玩。模型没换，换的是 harness。

OpenAI 用 Codex 也得出了同样结论：在一个 harness 搭得好的仓库里，同一个模型从"不可靠"变成"可靠"——不是"好了一点"，是质变。

**这门课教你怎么搭建那个环境。**

```text
                    THE HARNESS PATTERN
                    ====================

    You --> give task --> Agent reads harness files --> Agent executes
                                                        |
                                              harness governs every step:
                                              |
                                              +--> Instructions: what to do, in what order
                                              +--> Scope:       one feature at a time, no overreach
                                              +--> State:       progress log, feature list, git history
                                              +--> Verification: tests, lint, type-check, smoke runs
                                              +--> Boundaries:  what counts as "done," what counts as "broken"
                                              |
                                              v
                                         Agent stops only when
                                         verification passes
```

---

## Harness Engineering 到底是什么

Harness engineering 是围绕模型搭建一整套工作环境，让它产出可靠的结果。不只是写更好的提示词，而是设计模型运行所在的系统。

一个 harness 包含五个子系统：

```text
    ┌─────────────────────────────────────────────────────────────────┐
    │                        THE HARNESS                              │
    │                                                                 │
    │   ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
    │   │ Instructions  │  │    State     │  │   Verification       │ │
    │   │              │  │              │  │                      │ │
    │   │ AGENTS.md    │  │ progress.md  │  │ tests + lint         │ │
    │   │ CLAUDE.md    │  │ feature_list │  │ type-check           │ │
    │   │ feature_list │  │ git log      │  │ smoke runs           │ │
    │   │ docs/        │  │ session hand │  │ e2e pipeline         │ │
    │   └──────────────┘  └──────────────┘  └──────────────────────┘ │
    │                                                                 │
    │   ┌──────────────┐  ┌──────────────────────────────────────┐   │
    │   │    Scope     │  │         Session Lifecycle             │   │
    │   │              │  │                                      │   │
    │   │ one feature  │  │ init.sh at start                     │   │
    │   │ at a time   │  │ clean-state checklist at end          │   │
    │   │ definition   │  │ handoff note for next session        │   │
    │   │ of done      │  │ commit only when safe to resume      │   │
    │   └──────────────┘  └──────────────────────────────────────┘   │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘

    The MODEL decides what code to write.
    The HARNESS governs when, where, and how it writes it.
    The harness doesn't make the model smarter.
    It makes the model's output reliable.
```

每个子系统各司其职：

- **Instructions（指令）** — 告诉 agent 做什么、按什么顺序、开工前先读什么。不是一个巨大的文件，而是渐进式展开的结构，agent 按需导航。
- **State（状态）** — 跟踪做了什么、正在做什么、下一步是什么。持久化到磁盘，下次会话从上次停下的地方继续。
- **Verification（验证）** — 只有通过的测试套件才算数。agent 不能没有可运行的证据就说"做完了"。
- **Scope（范围）** — 约束 agent 一次只做一个功能。不多做，不少做，不偷偷改功能清单掩盖未完成的工作。
- **Session Lifecycle（会话生命周期）** — 开始时初始化，结束时清理，给下次会话留一条干净的重启路径。

---

## 为什么要有这门课

问题不是"模型能不能写代码"。能写。问题是：**能不能在真实的仓库里，跨越多次会话，不需要人一直盯着，就可靠地完成真实的工程任务？**

现在的答案是：没有 harness 就不行。

```text
    WITHOUT HARNESS                          WITH HARNESS
    ==============                          ============

    Session 1: agent writes code            Session 1: agent reads instructions
              agent breaks tests                      agent runs init.sh
              agent says "done"                       agent works on one feature
              you fix it manually                     agent verifies before claiming done
                                                       agent updates progress log
    Session 2: agent starts fresh                    agent commits clean state
              agent has no memory
              of what happened before         Session 2: agent reads progress log
              agent re-does work                       agent picks up exactly where it left off
              or does something else entirely          agent continues the unfinished feature
              you fix it again                         you review, not rescue

    Result: you spend more time                  Result: agent does the work,
            cleaning up than if you                      you verify the result
            did it yourself
```

课程真正关心的问题：

- 哪些 harness 设计会提升任务完成率？
- 哪些设计会减少返工和错误完成？
- 哪些机制能让长时任务更稳定地持续推进？
- 哪些结构能让系统在多轮 agent 运行后仍然可维护？

---

## 快速开始：今天就能改善你的 agent

不用读完 12 个讲义再动手。如果你已经在用 coding agent 做项目，下面这些能立刻改善效果。

思路很简单：不是光写提示词，而是在仓库里放一组结构化的文件——告诉 agent 该做什么、做完了什么、怎么验证。这些文件就放在项目里，每次会话都从同一个状态开始。

```text
    YOUR PROJECT ROOT
    ├── AGENTS.md              <-- the agent's operating manual
    ├── CLAUDE.md              <-- (alternative, if using Claude Code)
    ├── init.sh                <-- runs install + verify + start
    ├── feature_list.json      <-- what features exist, which are done
    ├── claude-progress.md     <-- what happened each session
    └── src/                   <-- your actual code
```

**第一步.** 把根指令文件复制到项目根目录：

- [`AGENTS.md`](./docs/resources/zh/templates/AGENTS.md) 通用版，或者 [`CLAUDE.md`](./docs/resources/zh/templates/CLAUDE.md) 如果你用 Claude Code
- 把里面的命令、路径和规则换成你自己项目的

**第二步.** 把启动脚本复制进去：

- `docs/resources/zh/templates/init.sh` —— 一条命令完成依赖安装、验证和启动
- 把 `INSTALL_CMD`、`VERIFY_CMD`、`START_CMD` 换成你自己的

**第三步.** 把进度日志复制进去：

- [`claude-progress.md`](./docs/resources/zh/templates/claude-progress.md) —— 每轮会话记录做了什么、验证了什么、下一步是什么
- agent 每次开工会先读这个文件，从上次停下的地方继续

**第四步.** 把功能清单复制进去：

- [`feature_list.json`](./docs/resources/zh/templates/feature_list.json) —— 机器可读的功能列表，每个功能有状态、验证步骤和证据
- 把示例功能换成你自己的

四个文件，最小起步就够了。比光靠一段提示词稳定得多。

等项目变复杂了，再补这些：

- [`session-handoff.md`](./docs/resources/zh/templates/session-handoff.md) —— 会话之间的交接摘要
- [`clean-state-checklist.md`](./docs/resources/zh/templates/clean-state-checklist.md) —— 每次会话结束前的清理清单
- [`evaluator-rubric.md`](./docs/resources/zh/templates/evaluator-rubric.md) —— 评审 agent 产出质量的评分表

每个文件的详细用法写在 [中文模板指南](./docs/resources/zh/templates/index.md)。英文版见 [英文模板指南](./docs/resources/en/templates/index.md)。

如果你想直接用 OpenAI 那篇 harness engineering 文章里的更完整高级结构，可以继续看 [`docs/resources/zh/openai-advanced/`](./docs/resources/zh/openai-advanced/index.md) 和 [`docs/resources/en/openai-advanced/`](./docs/resources/en/openai-advanced/index.md)。

---

## 贯穿项目：一个真实的应用

全部 6 个课程项目都围绕同一个产品：**一个基于 Electron 的个人知识库桌面应用**。

```text
    ┌─────────────────────────────────────────────────────┐
    │               Knowledge Base Desktop App            │
    │                                                     │
    │  ┌──────────────┐  ┌──────────────────────────────┐│
    │  │ Document List │  │       Q&A Panel              ││
    │  │              │  │                              ││
    │  │ doc-001.md   │  │  Q: What is harness eng?    ││
    │  │ doc-002.md   │  │  A: The environment built    ││
    │  │ doc-003.md   │  │     around an agent model... ││
    │  │ ...          │  │     [citation: doc-002.md]   ││
    │  └──────────────┘  └──────────────────────────────┘│
    │                                                     │
    │  ┌─────────────────────────────────────────────────┐│
    │  │ Status Bar: 42 docs | 38 indexed | last sync 3m ││
    │  └─────────────────────────────────────────────────┘│
    └─────────────────────────────────────────────────────┘

    Core features:
    ├── Import local documents
    ├── Manage a document library
    ├── Process and index documents
    ├── Run AI-powered Q&A over imported content
    └── Return grounded answers with citations
```

选择这个项目是因为它同时具备：很强的实际价值感、足够真实的产品复杂度、以及很适合观察 harness 优化前后的效果差异。

每个课程项目的 starter/solution 是这个 Electron 应用在对应演化阶段的完整副本。P(N+1) 的 starter 从 P(N) 的 solution 衍生而来——应用跟着你的 harness 技能一起进化。

---

## 学习路径

这门课按顺序来效果最好，每一阶段都建立在前一个的基础上。

```text
    Phase 1: SEE THE PROBLEM              Phase 2: STRUCTURE THE REPO
    ========================              ==========================

    L01  Strong models ≠ reliable         L03  Repository as single
         execution                              source of truth
    L02  What harness actually means
                                       L04  Split instructions across
         |                                   files, not one giant file
         v
    P01  Prompt-only vs.                       |
         rules-first comparison                v
                                               P02  Agent-readable workspace


    Phase 3: CONNECT SESSIONS             Phase 4: FEEDBACK & SCOPE
    ==========================           =========================

    L05  Keep context alive               L07  Draw clear task boundaries
         across sessions
                                       L08  Feature lists as harness
    L06  Initialize before every               primitives
         agent session
                                               |
         |                                     v
         v                                     P04  Runtime feedback to
    P03  Multi-session continuity                   correct agent behavior


    Phase 5: VERIFICATION                 Phase 6: PUT IT ALL TOGETHER
    =====================                 ============================

    L09  Stop agents from                 L11  Make agent's runtime
         declaring victory early               observable

    L10  Full-pipeline run =              L12  Clean handoff at end of
         real verification                      every session

         |                                     |
         v                                     v
    P05  Agent verifies its own work       P06  Build a complete harness
                                               (capstone project)
```

每个阶段大约一周（兼职节奏）。想快的话，前三个阶段一个周末就能跑完。

---

## 课程大纲

### 讲义 — 12 个概念单元，每个只回答一个核心问题

| Session | 核心问题 | 关键概念 |
| ------- | -------- | -------- |
| [L01](./docs/lectures/lecture-01-why-capable-agents-still-fail/index.md) | 模型能力强，不等于执行可靠 | 基准测试与真实工程之间的能力鸿沟 |
| [L02](./docs/lectures/lecture-02-what-a-harness-actually-is/index.md) | Harness 到底是什么 | 五个子系统：指令、状态、验证、范围、生命周期 |
| [L03](./docs/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md) | 让仓库成为唯一事实来源 | agent 看不到的东西，对它来说就不存在 |
| [L04](./docs/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md) | 为什么一个巨大的指令文件会失败 | 渐进式展开：给地图，不给百科全书 |
| [L05](./docs/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md) | 为什么长时任务会丢失上下文 | 把进度持久化到磁盘，从停下的地方继续 |
| [L06](./docs/lectures/lecture-06-why-initialization-needs-its-own-phase/index.md) | 为什么初始化需要单独一个阶段 | agent 开始工作前先验证环境是否健康 |
| [L07](./docs/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md) | 为什么 agent 会多做或少做 | 一次一个功能，显式的完成定义 |
| [L08](./docs/lectures/lecture-08-why-feature-lists-are-harness-primitives/index.md) | 为什么功能清单是 harness 原语 | 机器可读的范围边界，agent 无法忽略 |
| [L09](./docs/lectures/lecture-09-why-agents-declare-victory-too-early/index.md) | 为什么 agent 会提前宣告完成 | 验证缺口：自信 ≠ 正确 |
| [L10](./docs/lectures/lecture-10-why-end-to-end-testing-changes-results/index.md) | 为什么端到端测试会改变结果 | 只有跑通完整流程才算真正验证 |
| [L11](./docs/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md) | 为什么可观测性属于 harness | 看不到 agent 做了什么，就修不了它搞坏的东西 |
| [L12](./docs/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md) | 为什么每次会话都要留干净状态 | 下次会话的成功，取决于这次会话的清理 |

### 项目 — 6 个实践项目，把讲义方法落实到同一个 Electron 应用上

| Project | 你要做什么 | Harness 机制 |
| ------- | ---------- | ------------ |
| [P01](./docs/projects/project-01-baseline-vs-minimal-harness/index.md) | 跑两次同样的任务：只写提示词 vs 定好规则 | 最小 harness：AGENTS.md + init.sh + feature_list.json |
| [P02](./docs/projects/project-02-agent-readable-workspace/index.md) | 重组项目结构，让 agent 能读懂 | Agent 可读的工作空间 + 持久化状态文件 |
| [P03](./docs/projects/project-03-multi-session-continuity/index.md) | 让 agent 关掉再打开还能接着干 | 进度日志 + 会话交接 + 多会话连续性 |
| [P04](./docs/projects/project-04-incremental-indexing/index.md) | 防止 agent 做多了或做少了 | 运行反馈 + 范围控制 + 增量索引 |
| [P05](./docs/projects/project-05-grounded-qa-verification/index.md) | 让 agent 自己验证自己的工作 | 自验证 + 带引用的问答 + 基于证据的完成判定 |
| [P06](./docs/projects/project-06-runtime-observability-and-debugging/index.md) | 从零搭建一套完整的 harness（综合项目） | 完整 harness：全部机制 + 可观测性 + 消融实验 |

```text
    PROJECT EVOLUTION
    =================

    P01  Prompt-only vs. rules-first       You see the problem
     |
     v
    P02  Agent-readable workspace           You restructure the repo
     |
     v
    P03  Multi-session continuity           You connect sessions
     |
     v
    P04  Runtime feedback & scope           You add feedback loops
     |
     v
    P05  Self-verification                  You make the agent check itself
     |
     v
    P06  Complete harness (capstone)        You build the full system

    Each project's solution becomes the next project's starter.
    The app evolves. Your harness skills grow with it.
```

### 资料库

- [资料库总览](./docs/resources/index.md)
- [中文资料库](./docs/resources/zh/index.md) — 中文模板、清单和方法参考
- [英文资料库](./docs/resources/en/index.md) — English templates, checklists, and method references

---

## Agent 会话生命周期

这门课的一个核心理念：**agent 的会话应该遵循结构化的生命周期，而不是自由发挥。** 它长这样：

```text
    AGENT SESSION LIFECYCLE
    ======================

    ┌──────────────────────────────────────────────────────────────────┐
    │  START                                                          │
    │                                                                  │
    │  1. Agent reads AGENTS.md / CLAUDE.md                           │
    │  2. Agent runs init.sh (install, verify, health check)          │
    │  3. Agent reads claude-progress.md (what happened last time)    │
    │  4. Agent reads feature_list.json (what's done, what's next)    │
    │  5. Agent checks git log (recent changes)                       │
    │                                                                  │
    │  SELECT                                                          │
    │                                                                  │
    │  6. Agent picks exactly ONE unfinished feature                   │
    │  7. Agent works only on that feature                             │
    │                                                                  │
    │  EXECUTE                                                         │
    │                                                                  │
    │  8. Agent implements the feature                                 │
    │  9. Agent runs verification (tests, lint, type-check)           │
    │  10. If verification fails: fix and re-run                      │
    │  11. If verification passes: record evidence                    │
    │                                                                  │
    │  WRAP UP                                                         │
    │                                                                  │
    │  12. Agent updates claude-progress.md                           │
    │  13. Agent updates feature_list.json                            │
    │  14. Agent records what's still broken or unverified            │
    │  15. Agent commits (only when safe to resume)                   │
    │  16. Agent leaves clean restart path for next session           │
    │                                                                  │
    └──────────────────────────────────────────────────────────────────┘

    The harness governs every transition in this lifecycle.
    The model decides what code to write at each step.
    Without the harness, step 9 becomes "agent says it looks fine."
    With the harness, step 9 is "tests pass, lint is clean, types check."
```

---

## 适合谁

这门课适合：

- 已经在使用 coding agent、希望提升稳定性和质量的工程师
- 想系统理解 harness 设计的研究者或构建者
- 需要理解"环境设计如何影响 agent 表现"的技术负责人

这门课不适合：

- 只想要一个零代码 AI 入门的人
- 只关心 prompt，而不打算做真实实现的人
- 不准备让 agent 在真实仓库里工作的学习者

---

## 环境要求

这是一门真正需要动手跑 coding agent 的课程。

你至少需要具备一个这类工具：

- Claude Code
- Codex
- 其他支持文件编辑、命令执行、多步任务的 IDE / CLI coding agent

课程默认你可以：

- 打开本地仓库
- 允许 agent 编辑文件
- 允许 agent 运行命令
- 检查输出并重复执行任务

如果你没有这类工具，仍然可以阅读课程内容，但无法按预期完成课程项目。

---

## 本地预览

本仓库使用 VitePress 作为文档查看器。

```sh
npm install
npm run docs:dev        # Dev server with hot reload
npm run docs:build      # Production build
npm run docs:preview    # Preview built site
```

然后在浏览器里打开 VitePress 输出的本地地址即可。

---

## 先修要求

必需：

- 熟悉终端、git 和本地开发环境
- 至少会读写一种常见应用栈中的代码
- 有基本的软件调试经验，知道如何看日志、测试和运行行为
- 能投入足够时间完成偏实现型的课程任务

有帮助但非强制：

- 用过 Electron、桌面应用或本地优先工具
- 有测试、日志、软件架构方面的经验
- 已经接触过 Codex、Claude Code 或类似 coding agent

---

## 核心参考资料

主参考：

- [OpenAI: Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/)
- [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps)

辅助参考：

- [LangChain: The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness/)
- [Thoughtworks: Harness Engineering](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)
- [HumanLayer: Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents)

---

## 仓库结构

```text
learn-harness-engineering/
├── docs/                          # VitePress documentation site
│   ├── lectures/                  # 12 lectures (index.md + code/ examples)
│   │   ├── lecture-01-*/
│   │   ├── lecture-02-*/
│   │   └── ... (12 total)
│   ├── projects/                  # 6 project descriptions
│   │   ├── project-01-*/
│   │   └── ... (6 total)
│   └── resources/                 # Bilingual templates & references
│       ├── en/                    # English templates, checklists, guides
│       └── zh/                    # 中文模板、清单和指南
├── projects/
│   ├── shared/                    # Shared Electron + TypeScript + React foundation
│   └── project-NN/               # Per-project starter/ and solution/ directories
├── package.json                   # VitePress + dev tooling
└── CLAUDE.md                      # Claude Code instructions for this repo
```

---

## 课程组织方式

- 每个讲义聚焦一个问题
- 整门课配套 6 个项目
- 每个项目都要求 agent 真正干活
- 每个项目都要做弱 harness / 强 harness 对照
- 我们关心的是效果变化，而不是"写了多少说明文档"

---

## 致谢

本课程的灵感和部分思路来自 [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) —— 一个从零开始搭建 agent harness 的渐进式教程，从单个循环到隔离的自主执行。
