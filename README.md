# my-opencode-config

个人的 [opencode](https://opencode.ai) 配置仓库，包含全局指令、自定义命令、技能（skills）与模型 provider 配置。

## 文件结构

```
.
├── AGENTS.md            # 全局指令：默认使用简体中文交流
├── command/
│   ├── grill-with-plan.md  # /grill-with-plan 命令（当前引用了尚未安装的 skill）
│   ├── writing-plans.md    # /writing-plans 命令：生成待用户 review 的实现计划
│   └── executing-plans.md  # /executing-plans 命令：执行已批准的计划
├── skills/                # 28 个工作流与参考 skill
├── opencode.jsonc         # provider、model 与 ask-image MCP 配置
└── opencode-easy-vision.jsonc # 图片粘贴与视觉分析插件配置
```

## 自定义命令

### /grill-with-plan

命令文件仍然存在，但它引用的 `grill-with-plan` skill 当前不在仓库中，因此暂时不要把它作为稳定入口。可使用下面的等价流程：

```text
/grilling → /writing-plans → 用户明确批准 → /executing-plans
```

详见 [command/grill-with-plan.md](command/grill-with-plan.md)。

### /writing-plans

从 spec、ticket 或明确需求生成详细实现计划。计划必须包含文件、接口、代码示例、测试示例、检查命令和预期结果；保存后等待用户明确批准，不修改应用代码。

详见 [command/writing-plans.md](command/writing-plans.md) 和 [skills/writing-plans/SKILL.md](skills/writing-plans/SKILL.md)。

### /executing-plans

执行已经批准的计划：逐任务实现、运行检查、记录偏差，完成后进行完整 diff review；用户批准 batch 后才关闭 ticket 并提交 commit。

详见 [command/executing-plans.md](command/executing-plans.md) 和 [skills/executing-plans/SKILL.md](skills/executing-plans/SKILL.md)。

## 推荐工作流

### 从想法到交付

```text
首次配置
   ↓
/setup-matt-pocock-skills
   ↓
/grill-with-docs
   ↓
需要可运行验证？──是──> /handoff → /prototype → /handoff 回主流程
   ↓ 否
需要跨多个 session？──是──> /to-spec → /to-tickets
   ↓ 否                         ↓
                                     /implement（每个 ticket 一个新上下文）
   ↓
/implement
   ↓
/writing-plans → 用户批准 → /executing-plans
   ↓
/tdd → /code-review → 用户批准 → commit
```

`/grill-with-docs`、`/to-spec` 和 `/to-tickets` 应尽量在同一个上下文中完成；每个 `/implement` ticket 从干净上下文开始。

### 其他入口

| 情况 | 入口 | 后续流程 |
| --- | --- | --- |
| 收到尚未整理的 issue 或 feature request | `/triage` | 分类、验证、补充信息或生成 agent brief，再交给 `/implement` |
| 难以复现、间歇性或性能问题 | `/diagnosing-bugs` | 先建立能针对症状变红的反馈循环，再最小化、假设验证、回归测试 |
| 项目或大型功能过于庞大，路线尚不清晰 | `/wayfinder` | 建立决策地图，逐个解决 decision ticket，最后进入 `/to-spec` |
| 想改善代码结构 | `/improve-codebase-architecture` | 生成架构候选 HTML 报告，选择候选后进入 `grilling` 和 `domain-modeling` |
| 已处于 merge/rebase 冲突状态 | `/resolving-merge-conflicts` | 按双方意图解决每个 hunk，运行检查并完成 merge/rebase |

## Skills 使用指南

以下是当前 `skills/` 目录中的全部 28 个 skill。skill 名称就是调用名，例如 `/tdd`；带有完整说明的链接指向对应的 `SKILL.md`。

### 路由、澄清与上下文

| Skill | 使用方法与流程 |
| --- | --- |
| [ask-matt](skills/ask-matt/SKILL.md) | 不确定该选哪个 skill 时使用。根据任务选择主流程、bug 流程、triage、wayfinder 或独立 skill。 |
| [grilling](skills/grilling/SKILL.md) | 需求或设计不清晰时使用。按设计树的 frontier 分轮提问，每轮等待回答，直到没有未决策。 |
| [grill-me](skills/grill-me/SKILL.md) | 没有工作目录、只想澄清想法时使用。调用 `grilling`，不写本地文档。 |
| [grill-with-docs](skills/grill-with-docs/SKILL.md) | 在仓库中澄清需求时使用。调用 `grilling` 和 `domain-modeling`，同步维护 `CONTEXT.md` 与 ADR。 |
| [wait-what](skills/wait-what/SKILL.md) | 用户没有理解上一段说明时使用。用简化技术英语和 `CONTEXT.md` 中的术语重新解释。 |
| [handoff](skills/handoff/SKILL.md) | 换 harness、目录、同事或分叉任务时使用。把当前上下文写入操作系统临时目录的 handoff Markdown。 |
| [wayfinder](skills/wayfinder/SKILL.md) | 大型且路线不清晰的工作使用。先定义 destination，再建立决策地图和依赖关系，逐 ticket 解决，不直接实现最终目标。 |
| [setup-matt-pocock-skills](skills/setup-matt-pocock-skills/SKILL.md) | 第一次使用工程流程前运行。配置 issue tracker、triage labels、`CONTEXT.md` 和 ADR 布局。 |

### 需求、规格与计划

| Skill | 使用方法与流程 |
| --- | --- |
| [to-questionnaire](skills/to-questionnaire/SKILL.md) | 需要从其他人处获取业务事实或决策时使用。先确认收件人和需要的信息，再生成 `to-questionnaire-<slug>.md`。 |
| [to-spec](skills/to-spec/SKILL.md) | 对话已经形成共识时使用。不重新采访，综合现有上下文写 spec，确认测试 seam 后发布到 issue tracker。 |
| [to-tickets](skills/to-tickets/SKILL.md) | 将 spec、计划或对话拆为可独立验证的 tracer-bullet tickets，声明每个 ticket 的 blocking edges；先让用户确认拆分再发布。 |
| [writing-plans](skills/writing-plans/SKILL.md) | 只写实现计划，不改应用代码。读取 ticket、spec、领域文档和 ADR，列出具体文件、接口、步骤、测试和检查命令，最后等待批准。 |
| [implement](skills/implement/SKILL.md) | 按 spec 或 ticket 开发。先调用 `writing-plans` 并等待批准，再调用 `executing-plans`；实现过程中使用 TDD，最后做 code review。 |
| [executing-plans](skills/executing-plans/SKILL.md) | 执行已批准的计划。记录 `base-sha`，逐任务验证，记录偏差，完成 batch review，等用户批准后关闭 ticket 并一次性提交。 |

### 测试、调试与评审

| Skill | 使用方法与流程 |
| --- | --- |
| [tdd](skills/tdd/SKILL.md) | 需要 test-first 或红绿重构时使用。先约定测试 seam，再按“红测试 → 最小实现 → 绿测试”的垂直切片循环推进。 |
| [diagnosing-bugs](skills/diagnosing-bugs/SKILL.md) | 难 bug、间歇性 bug 或性能回归使用。先建立一个已实际运行且能针对该症状变红的紧反馈循环，再提出可证伪假设并写回归测试。 |
| [code-review](skills/code-review/SKILL.md) | Review 分支、PR 或工作区变更时使用。要求 fixed point，并行执行 Standards 与 Spec 两个维度的 review。 |
| [resolving-merge-conflicts](skills/resolving-merge-conflicts/SKILL.md) | 处理进行中的 merge/rebase 冲突。查找双方 primary source，按意图解决 hunk，运行自动检查并完成操作。 |
| [prototype](skills/prototype/SKILL.md) | 逻辑、状态模型或 UI 难以纸面判断时使用。逻辑生成可双击的单 HTML；UI 在同一路由生成多个结构不同的 variant，验证后再合并真实代码。 |

### 架构与领域模型

| Skill | 使用方法与流程 |
| --- | --- |
| [codebase-design](skills/codebase-design/SKILL.md) | 设计模块接口、depth 或 seam 时参考。使用 module、interface、adapter、leverage、locality 等统一术语，必要时执行 deletion test 或 Design It Twice。 |
| [domain-modeling](skills/domain-modeling/SKILL.md) | 领域术语模糊、需要更新 `CONTEXT.md` 或记录 ADR 时使用。挑战歧义、构造边界场景、对照代码，只为不可逆且有真实权衡的决定创建 ADR。 |
| [improve-codebase-architecture](skills/improve-codebase-architecture/SKILL.md) | 做架构体检时使用。扫描近期热点，识别 shallow module，输出 OS 临时目录中的 HTML 报告；用户选择候选后再深入设计。 |

### Issue、研究、教学与工具

| Skill | 使用方法与流程 |
| --- | --- |
| [triage](skills/triage/SKILL.md) | 处理外部 issue 或 PR。检查重复实现和历史拒绝，建议 category/state，等待维护者方向，再验证并生成 agent brief 或 triage notes。 |
| [research](skills/research/SKILL.md) | 需要外部事实或官方文档调查时使用。启动后台 agent，只查 primary sources，将带引用的结果写入仓库 Markdown。 |
| [find-skills](skills/find-skills/SKILL.md) | 想查找或安装新 skill 时使用。先查 skills.sh，再运行 `npx skills find`，核验来源、安装量和 reputation 后再推荐或安装。 |
| [wizard](skills/wizard/SKILL.md) | 只有人能完成的控制台、凭据、CI secret 或一次性迁移使用。基于 `template.sh` 生成交互式 Bash 脚本，完成 `bash -n` 和 `shellcheck` 静态验证。 |
| [teach](skills/teach/SKILL.md) | 需要跨 session 学习一个主题时使用。维护 `MISSION.md`、可信资源、HTML lessons、reference 和 learning records，通过练习反馈推进。 |
| [writing-for-agents](skills/writing-for-agents/SKILL.md) | 编写或修改 skill、`AGENTS.md`、`CLAUDE.md` 或 agent 文档时使用。重点检查 context pointers、渐进披露、完成标准和单一事实源。 |

## Skill 调用方式

直接在 opencode 中输入 skill 名称和任务，例如：

```text
/diagnosing-bugs 登录接口在偶发超时，请先建立一个可重复的失败反馈循环
/writing-plans 实现 ticket .scratch/orders/issues/01-create-order.md
/code-review review since main
```

带有 `disable-model-invocation: true` 的 skill 需要用户主动输入；其他 skill 也可以由 agent 在符合触发条件时自动调用。每个 skill 的 frontmatter 和 `SKILL.md` 是最终规则来源，README 只提供导航和流程摘要。

## Provider 配置

`opencode.jsonc` 中定义了 `volcengine-plan` provider，使用 `@ai-sdk/openai-compatible` 适配器接入火山引擎编码服务。

- `baseURL`: `https://ark.cn-beijing.volces.com/api/coding/v3`
- `apiKey`: 配置中为占位符 `YOUR_API_KEY`，请替换为真实密钥，或通过本地覆盖配置注入（见下文"使用方式"）

可用模型：

| 模型                 | 上下文  | 输出 | 输入模态    |
| -------------------- | ------- | ---- | ----------- |
| ark-code-latest      | 256000  | 4096 | text, image |
| doubao-seed-code     | 256000  | 4096 | text, image |
| doubao-seed-2.0-code | 256000  | 4096 | text, image |
| doubao-seed-2.0-pro  | 256000  | 4096 | text, image |
| doubao-seed-2.0-lite | 256000  | 4096 | text, image |
| glm-5.2              | 1024000 | 4096 | text        |
| glm-latest           | 1024000 | 4096 | text        |
| deepseek-v4-flash    | 1024000 | 4096 | -           |
| deepseek-v4-pro      | 1024000 | 4096 | -           |
| minimax-m2.7         | 200000  | 4096 | text        |
| minimax-m3           | 512000  | 4096 | text, image |
| kimi-k2.6            | 256000  | 4096 | text, image |
| kimi-k2.7-code       | 256000  | 4096 | text        |

## 使用方式

1. 将本仓库内容放到 opencode 的配置目录（如 `~/.config/opencode/`）或项目 `.opencode/` 目录下。
2. 配置 API Key：
   - 方式一：直接编辑 `opencode.jsonc`，将 `apiKey` 占位符替换为真实密钥。
   - 方式二（推荐）：创建 `opencode.local.jsonc`（已被 `.gitignore` 忽略，不会提交），在其中覆盖 `apiKey` 为真实密钥。
3. 第一次使用工程 skills 时运行 `/setup-matt-pocock-skills`，按提示配置 issue tracker 和领域文档布局。
4. 在 opencode 中从 `/ask-matt` 选择流程，或直接使用 `/grill-with-docs`、`/writing-plans`、`/implement` 等入口。

## 安全说明

- 本仓库的 `opencode.jsonc` 仅使用占位符（`YOUR_API_KEY`），**不含真实密钥**。
- 真实 API Key 请通过本地覆盖配置（`opencode.local.jsonc`）注入，请勿将含真实 key 的配置提交到 Git。
- 详见 `.gitignore` 中对本地配置与环境变量文件的忽略规则。
