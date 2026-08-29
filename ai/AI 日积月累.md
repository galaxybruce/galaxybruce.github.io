# AI 实践日积月累

> 持续沉淀 AI 开发实践中的经验、踩坑与最佳实践。采用「目录 + 专题」结构，点击目录条目可跳转到对应章节，后续新专题在对应分类末尾追加。

### 目录（持续更新）

- [AI 实践日积月累](#ai-实践日积月累)
    - [目录（持续更新）](#目录持续更新)
  - [一、架构与选型](#一架构与选型)
    - [Codex / app-server](#codex--app-server)
      - [专题 1 · 自定义应用调用 app-server：到底要不要装 Codex 桌面 App？](#专题-1--自定义应用调用-app-server到底要不要装-codex-桌面-app)
        - [结论](#结论)
        - [核心原则](#核心原则)
        - [组件对照](#组件对照)
        - [实践要点](#实践要点)
      - [专题 2 · codex-rs/app-server 与 codex cli 的关联与区别](#专题-2--codex-rsapp-server-与-codex-cli-的关联与区别)
        - [1. 核心关联（本质关系）](#1-核心关联本质关系)
        - [2. 主要区别（功能侧重）](#2-主要区别功能侧重)
        - [3. 自定义应用开发选型结论](#3-自定义应用开发选型结论)
  - [二、提示词工程](#二提示词工程)
    - [专题 1 · 提示词写作要点：复杂任务的四要素](#专题-1--提示词写作要点复杂任务的四要素)
  - [三、工具与 Skill](#三工具与-skill)
    - [专题 1 · AI Skill 与插件收集记录](#专题-1--ai-skill-与插件收集记录)
      - [claude-vision-skill](#claude-vision-skill)
      - [Superpowers 插件](#superpowers-插件)
    - [专题 2 · CodeGraph 使用](#专题-2--codegraph-使用)
    - [专题 3 · pre-commit 使用](#专题-3--pre-commit-使用)
  - [四、模型部署](#四模型部署)
    - [专题 1 · 大模型本地部署的显存计算：精度与量化](#专题-1--大模型本地部署的显存计算精度与量化)

## 一、架构与选型

### Codex / app-server

#### 专题 1 · 自定义应用调用 app-server：到底要不要装 Codex 桌面 App？

##### 结论

- **不需要**：Codex 桌面 App（GUI 客户端）
- **必须**：Codex CLI，或直接下载 app-server 的独立二进制文件 / 源码

##### 核心原则

你的自定义应用需要一个**后台进程（运行时 / runtime）**，它能通过 **JSON-RPC 协议**与 app-server 通信。这个后台进程由 **CLI** 或 **独立二进制文件** 提供；桌面 App 只是这个运行时之上的一层图形前端壳（GUI shell），并非运行时本体。

##### 组件对照

| 组件 | 角色 | 是否必需 |
| --- | --- | --- |
| Codex 桌面 App | 图形前端壳（GUI shell） | 否 |
| Codex CLI | 提供 JSON-RPC 运行时的后台进程 | 是（或下方替代项） |
| app-server 独立二进制 / 源码 | 同上，运行时的另一种提供方式 | 是（或使用 CLI） |

##### 实践要点

1. 启动一个后台进程作为运行时；
2. 通过 JSON-RPC 与 app-server 通信；
3. 桌面 App 仅用于本地可视化 / 调试，不参与自定义应用的生产链路。

#### 专题 2 · codex-rs/app-server 与 codex cli 的关联与区别

> 结构精简，适合直接复制。

##### 1. 核心关联（本质关系）

- **大脑与外壳**：codex-rs/app-server 是底层的「AI 核心引擎（大脑）」；codex cli 是基于这个引擎封装的「命令行客户端（外壳）」。
- **包含关系**：当你运行 codex cli 的核心指令（如 `codex app-server`）时，它在后台拉起并运行的正是 codex-rs/app-server。
- **共享状态**：两者底层使用完全相同的本地配置、Model Context Protocol (MCP) 插件以及全局身份认证 Session（`~/.codex/auth.json`）。

##### 2. 主要区别（功能侧重）

| 特性 / 维度 | codex-rs/app-server (Rust 引擎) | codex cli (命令行客户端) |
| --- | --- | --- |
| 产品定位 | 纯粹的无头服务（Headless Service），面向开发者。 | 完整的终端应用（TUI），面向终端最终用户。 |
| 交互界面 | 无人类交互界面。只通过标准输入输出（stdio）或 WebSocket 读写 JSON-RPC 报文。 | 有交互界面。内置精美的终端聊天窗口、实时流式文本和代码差异（Diff）高亮显示。 |
| 核心职责 | 负责 AI 深度思考、代码沙箱执行、文件读写补丁（Patch）及长会话线程管理。 | 负责接收人类键盘输入、渲染排版 AI 回复、以及提供 `codex auth` 等便捷管理命令。 |
| 开发集成 | 推荐集成。适合被第三方应用（如自定义软件、IDE 插件）通过进程管道直接拉起调用。 | 适合开发调试阶段，手动用来做本地验证或日常终端编程。 |

##### 3. 自定义应用开发选型结论

如果你要独立开发一款应用，你的产品只需要通过 **JSON-RPC 协议** 去驱动 codex-rs/app-server 即可。

你不需要重复实现它的智能体循环和沙箱逻辑，直接向它发送会话指令，再将返回的 JSON 数据渲染到你自己的 UI 界面中。

## 二、提示词工程

### 专题 1 · 提示词写作要点：复杂任务的四要素

简单的任务，一个简短的提示通常就足够了；对于较为复杂或重要的任务，提示词需要包含以下至关重要的部分。

| 要素 | 英文 | 含义 |
| --- | --- | --- |
| 目标 | Goal: What should ChatGPT do? | ChatGPT 应该做什么？ |
| 上下文 | Context: What information or sources will help? | 哪些信息或来源能够提供帮助？ |
| 输出 | Output: What format, length, or level of detail do you need? | 需要什么格式、长度或详细程度？ |
| 边界 | Boundaries: What must stay unchanged? What should it avoid or check with you before it acts? | 哪些事情必须保持不变？行动前应避免什么、需要与你确认什么？ |

## 三、工具与 Skill

### 专题 1 · AI Skill 与插件收集记录

#### claude-vision-skill

- 作用：开源的视觉 Skill，为 AI 编程助手增加视觉理解能力，可处理图片识别与分析类任务
- GitHub 地址：https://github.com/asuojun/claude-vision-skill
- 安装提示词：
- 请帮我全局安装开源的 Vision Skill。项目的 GitHub 地址是：https://github.com/asuojun/claude-vision-skill。请按照项目的配置要求，将其中的视觉模型指定为智普的GLM-4.6V-Flash，并将我的 API Key 配置为 \\\<在此填入你的 API Key，勿写入文档\>。安装好后请告知我。

#### Superpowers 插件

- 作用：开源的 Claude Code 工程化开发插件（作者 Jesse Vincent / obra，GitHub 14 万+ stars），通过一套可组合的 skill 模块（20+ 个）强制 AI 走结构化工作流——先讨论方案、再写计划、再 TDD 实现、再自检与验证，避免「上来就写代码、写完就说 done」。
- GitHub 地址：https://github.com/obra/superpowers
- 用途与场景：核心模块包括
  - **brainstorming**：需求讨论，先拉着 AI 探索不同路径、把关键决策点梳理清楚再动手（最有用的模块之一）
- **writing-plans**：把需求拆解成实现计划
- **test-driven-development**：TDD 红绿重构循环——先写失败的测试，再写最小实现让其通过，再重构，测试不过不改完状态
- **systematic-debugging**：四阶段调试法，要求根因分析后才能动手修；连续 3 次修复失败会触发架构审查
- **code-reviewer**：实现完成后按计划、编码规范、架构原则自我审查
- **verification-before-completion**：声称「完成」前必须通过的验证门
- **dispatching-parallel-agents**：当有 2 个以上无依赖的独立任务时自动并行调度
- 安装提示词：
- 在 Claude Code 会话中执行 `claude plugin install superpowers`（或 `/plugin install superpowers@claude-plugins-official`）。下次启动会话时看到 "You have Superpowers" 提示即表示安装成功。建议只在 CLAUDE.md 中显式开启常用 2\~3 个模块（如 brainstorming、test-driven-development、verification-before-completion），其余按项目需要再启用，避免 20+ 模块全开导致上下文压力过大。

### 专题 2 · CodeGraph 使用

- 作用：开源的本地代码知识图谱工具，为 AI 编程 Agent（Codex CLI、Claude Code、Cursor、opencode 等）预建项目结构地图，一次工具调用即可返回入口点、关联符号与调用链，减少反复 grep / 逐文件读取的开销
- 项目地址：https://github.com/colbymchenry/codegraph
- 特点：
  - **100% 本地**：索引保存在本地 SQLite，数据不出机器，无需 API Key
  - **Rust 内核**：支持 20+ 种语言（TypeScript、JavaScript、Python、Go、Rust、Java、C/C++、Swift、Kotlin 等）
  - **影响分析**：可追踪任意符号的调用方 / 被调用方与影响面，改动前先评估风险
  - **自动同步**：监听文件变化，编辑后自动增量更新索引，无需手动重建
- 安装方式（任选其一）：
  - 官方脚本（macOS / Linux）：

    ```bash
    curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh
    ```
  - npm（已有 Node 环境）：

    ```bash
    npm i -g @colbymchenry/codegraph
    ```
- 使用步骤：
  1. 接入 Agent：执行 `codegraph install`，自动检测并配置 Codex CLI、Claude Code、Cursor、opencode 等，通过 MCP 将 CodeGraph 接入
  2. 初始化项目：在项目根目录执行 `codegraph init`，创建本地 `.codegraph/` 目录并构建全量图谱
  3. 之后无需手动同步，保存文件时图谱自动增量更新
- 常用命令：
  - `codegraph upgrade`：升级（自动识别安装方式，也可指定版本）
  - `codegraph uninstall`：从已配置的 Agent 中移除并卸载（`--keep-cli` 仅移除 Agent 配置）
  - `codegraph uninit`：删除某个项目的本地索引
- 使用建议：CodeGraph 只在被 Agent 直接查询时生效，建议让 Agent 优先直接查询图谱，而不是退回逐文件探索，否则图索引会成为额外开销

### 专题 3 · pre-commit 使用

- 独立文档：[pre-commit使用](pre-commit使用.md)
- 归类说明：`pre-commit` 属于开发工具链与自动化校验能力，统一收拢在「工具与 Skill」分类下，避免在仓库总目录重复展开。

## 四、模型部署

### 专题 1 · 大模型本地部署的显存计算：精度与量化

模型显存占用按出厂全精度（FP16 / 16 位浮点数）计算：每个参数占 2 字节（Byte），所以 1B（10 亿）参数对应 2GB 显存。但实际本地部署时极少直接跑全精度模型，而是使用量化技术对模型进行压缩。

| 精度 | 每参数占用 | 1B 参数显存 | 32B 模型显存 |
| --- | --- | --- | --- |
| 全精度 FP16 | 2 字节 | 2 GB | 64 GB（48G 内存确实不够） |
| 主流本地量化 INT4 / Q4 | ≈ 0.5 字节 | ≈ 0.5 GB | 16 \~ 20 GB |
| INT8 / Q8 | 1 字节 | 1 GB | \~32 GB |

**INT4 / Q4（4 位量化）**：每个参数用 4 个比特存储（0.5 字节），压缩率最高，显存约为全精度的 1/4。代价是精度损失较大，适合显存有限的消费级显卡本地部署。Q4 是 GGUF 格式中的命名（如 Q4\\\_K\\\_M），INT4 是通用术语，两者本质相同。

**INT8 / Q8（8 位量化）**：每个参数用 8 个比特存储（1 字节），压缩率为全精度的 1/2。相比 INT4 质量更接近原版，推理精度损失很小，但显存占用翻倍。适合显存较充裕（40GB+）或对输出质量要求较高的任务。Q8 同理是 GGUF 格式中的命名。

结论：32B 模型全精度需要 64GB，48G 内存确实不够；INT4 量化后只需 16\~20GB，主流消费级显卡即可本地部署；INT8 量化约需 32GB，适合 40GB+ 显存的中高端显卡，质量更接近原版。
