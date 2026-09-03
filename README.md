# my-opencode-config

个人的 [opencode](https://opencode.ai) 配置仓库，包含全局指令、自定义命令、技能（skills）与模型 provider 配置。

## 文件结构

```
.
├── AGENTS.md            # 全局指令：默认使用简体中文交流
├── command/
│   └── grill-with-plan.md  # /grill-with-plan 命令：澄清需求并产出待确认的执行计划
├── skills/              # 技能库（superpowers 工作流 + 第三方技能）
│   ├── grill-with-plan/     # 澄清 → 计划 → 确认 → 执行 的完整工作流
│   ├── grilling/            # 需求澄清（设计树式追问）
│   ├── grill-with-docs/     # 澄清并同步产出 ADR / 术语表
│   ├── writing-plans/       # 编写实现计划（保存到 docs/superpowers/plans/）
│   ├── subagent-driven-development/  # 子代理驱动开发（每任务独立子代理 + 评审）
│   ├── executing-plans/     # 计划执行（带检查点）
│   ├── test-driven-development/      # TDD
│   ├── systematic-debugging/         # 系统化调试
│   ├── requesting-code-review/       # 请求代码评审
│   ├── receiving-code-review/        # 接收代码评审
│   ├── finishing-a-development-branch/ # 完成开发分支
│   ├── using-git-worktrees/          # 隔离工作区
│   ├── verification-before-completion/ # 完成前验证
│   ├── writing-skills/               # 编写技能
│   ├── dispatching-parallel-agents/  # 并行子代理调度
│   ├── domain-modeling/              # 领域建模（CONTEXT.md / ADR）
│   ├── using-superpowers/            # 技能调用总入口
│   └── vercel-react-best-practices/  # Vercel React/Next.js 性能最佳实践（第三方）
└── opencode.jsonc       # opencode 配置（provider 与模型定义，本地使用，见下文说明）
```

## 自定义命令

### /grill-with-plan

澄清一个不明确的实现需求，产出执行计划并等待用户确认后再执行。详见 `command/grill-with-plan.md`。

工作流：`grilling`（澄清）→ `writing-plans`（写计划）→ 用户确认 → `subagent-driven-development` 或 `executing-plans`（执行）。计划文档保存到 `docs/superpowers/plans/`。

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
3. 在 opencode 中使用 `/grill-with-plan` 澄清并规划任务，确认计划后执行。

## 安全说明

- 本仓库的 `opencode.jsonc` 仅使用占位符（`YOUR_API_KEY`），**不含真实密钥**。
- 真实 API Key 请通过本地覆盖配置（`opencode.local.jsonc`）注入，请勿将含真实 key 的配置提交到 Git。
- 详见 `.gitignore` 中对本地配置与环境变量文件的忽略规则。
