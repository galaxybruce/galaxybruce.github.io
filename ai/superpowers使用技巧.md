# Superpowers 使用技巧

## 自动使用

- **任务匹配即强制使用**：只要任务明显匹配某技能的描述，Agent 必须自动调用，没有选择余地。
- **1% 可能匹配也必须调用**：规则是"只要有 1% 可能适用，就必须先读技能再行动"。
- **对话开始自动检查**：每次对话从 `using-superpowers` 开始，在任何响应、澄清提问、探索代码之前先做技能检查。
- **常见自动触发**：遇到 bug → `systematic-debugging`；创造性工作（建功能、改行为）→ `brainstorming`；声称完成/提交前 → `verification-before-completion`；收到评审意见 → `receiving-code-review`。

## 手动使用

- **用户点名技能**：用户提到某技能名（如 `$test-driven-development` 或"用 TDD"）时，必须按该技能执行。
- **要求特定流程**：用户明确要求写计划、建 worktree、并行派发子代理等，手动指定对应技能。
- **多技能叠加**：一次任务可同时触发多个技能，按顺序执行；用户也可主动叠加，如"先写计划再执行"。
- **多个技能适用时的优先级**：过程技能优先（`brainstorming`、`systematic-debugging`），再执行实施技能。
  - "构建 X" → `brainstorming` → 实施技能
  - "修 bug" → `systematic-debugging` → 领域技能

更多手动使用场景：

- **要求固定流程**：即使改动很小，也明确要求走 TDD、先写计划或先建 worktree，例如"这个改动也按 TDD 来"。
- **项目规范约定**：团队规范要求所有功能必须评审、所有提交前必须验证，即使单次任务不会自动触发对应技能。
- **追求并行加速**：改动虽小但想加快，手动指定并行派发，例如"这几个文件分别让子代理改"。
- **主动要求评审**：合并前手动指定"先评审我的改动"，触发 `requesting-code-review`。
- **先讨论再动手**：功能还不明确时手动说"先头脑风暴一下方案"，提前触发 `brainstorming`。
- **交接/复跑已有计划**：拿到他人写好的实施计划，手动指定"按这个计划执行"，触发 `executing-plans`。
- **创建或更新技能**：手动说"帮我创建一个 XX 技能"，触发 `writing-skills`。

## 各技能适用时机

| 技能 | 何时使用 |
| --- | --- |
| `using-superpowers` | 每次对话开始；确定是否适用技能，先检查后行动 |
| `brainstorming` | 任何创造性工作前（创建功能、构建组件、添加功能、修改行为）；探索意图、需求与设计 |
| `writing-plans` | 已有规格/需求的多步任务，写代码前 |
| `executing-plans` | 在独立会话中执行已写好的实施计划（带评审检查点） |
| `subagent-driven-development` | 在当前会话执行含独立任务的实施计划，用子代理推进 |
| `dispatching-parallel-agents` | 有 2+ 个独立任务，且无共享状态、无顺序依赖 |
| `test-driven-development` | 实现任何功能或 bugfix 前，先写测试 |
| `systematic-debugging` | 遇到任何 bug、测试失败或意外行为时，先定位原因再提修复 |
| `receiving-code-review` | 收到代码评审反馈后、实施建议前；尤其反馈不清晰或技术可疑时 |
| `requesting-code-review` | 完成任务、实现大功能或合并前，验证工作是否满足要求 |
| `verification-before-completion` | 声称完成/修复/通过、提交或建 PR 前；先跑验证命令再下结论 |
| `using-git-worktrees` | 开始需要隔离的功能开发，或执行实施计划前 |
| `finishing-a-development-branch` | 实现完成且测试通过后，决定如何集成（提交/推送/PR） |
| `writing-skills` | 创建新技能、编辑既有技能或验证技能可用性时 |

## 常用使用流程

**完整开发流程**（新功能 → 合并）

1. `using-superpowers`：对话开始，做技能检查
2. `brainstorming`：明确需求与设计
3. `writing-plans`：多步任务先写实施计划
4. `using-git-worktrees`：隔离开发环境
5. `test-driven-development`：先写测试再实现
6. `verification-before-completion`：全部通过后再声称完成
7. `requesting-code-review`：合并前验证工作
8. `finishing-a-development-branch`：提交、推送、建 PR

示例：为项目新增"导出 CSV"功能

1. 对话开始：检查技能，判定为创造性工作
2. `brainstorming`：明确导出字段、文件编码、空数据处理
3. `writing-plans`：拆成后端生成、前端下载、测试三步
4. `using-git-worktrees`：建 `feature/csv-export` 隔离开发
5. `test-driven-development`：先写导出函数的单测
6. `verification-before-completion`：跑全部测试与构建
7. `requesting-code-review`：重点评审空数据与中文编码边界
8. `finishing-a-development-branch`：提交、推送、建 PR

**修 bug 流程**

1. `systematic-debugging`：定位根因，不直接猜改
2. `test-driven-development`：写复现失败的测试
3. 修复实现 → 验证通过
4. 收到评审意见时走 `receiving-code-review`

示例：登录页白屏

1. `systematic-debugging`：复现并查日志，定位为登录接口字段重命名导致前端解析报错
2. `test-driven-development`：写一条复现白屏的失败测试
3. 修复解析逻辑 → 测试通过、页面正常
4. 评审质疑"兼容旧接口"→ 走 `receiving-code-review` 核实需求再改

**执行已有计划**（独立会话）

1. `using-git-worktrees`：创建隔离工作区
2. `executing-plans`：按检查点分阶段执行，每阶段验证
3. `finishing-a-development-branch`：集成收尾

示例：将构建工具从 Vite 4 升级到 5

1. `using-git-worktrees`：建升级专用工作区，不动主分支
2. `executing-plans`：按计划检查点执行——升级依赖 → 跑构建 → 修复破坏性变更 → 比对产物
3. `finishing-a-development-branch`：测试全过后集成

**当前会话并行执行计划**

1. `subagent-driven-development`：拆分独立任务并派发
2. 并行任务用 `dispatching-parallel-agents`
3. 逐任务验证、汇总、集成

示例：一次性实现三个无依赖的接口 A、B、C

1. `subagent-driven-development`：把 A、B、C 拆成三个独立子任务
2. `dispatching-parallel-agents`：三个子代理并行实现
3. 逐接口跑测试 → 汇总代码 → 跑集成测试 → 合并

**维护技能**

1. `writing-skills`：创建或编辑技能
2. 验证可用性后再使用

示例：新建"定时任务检查"技能

1. `writing-skills`：编写 SKILL.md 与说明
2. 用真实场景调用一次，确认触发规则与步骤可用

## 注意点

- 技能检查优先于一切：包括澄清提问、读代码、查文件。
- 用户指令（CLAUDE.md、AGENTS.md、直接请求）优先级高于技能；技能仅覆盖默认行为。
- `dispatching-parallel-agents` 与 `subagent-driven-development` 依赖多代理支持：需在 Codex 配置 `[features] multi_agent = true`。
- 技能会演进，每次使用前读取当前版本，不要凭记忆执行。
