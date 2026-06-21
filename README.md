# my-opencode-config

个人的 [opencode](https://opencode.ai) 配置仓库，包含全局指令、自定义命令与模型 provider 配置。

## 文件结构

```
.
├── AGENTS.md            # 全局指令：默认使用简体中文交流
├── command/
│   ├── plan.md          # /plan 命令：生成任务计划文档到 .opencode/plan/
│   └── run.md           # /run 命令：执行 plan 文件中定义的各个 Task
└── opencode.jsonc       # opencode 配置（provider 与模型定义，本地使用，见下文说明）
```

## 自定义命令

### /plan

生成任务计划文档，保存到当前项目 `.opencode/plan/` 目录。详见 `command/plan.md`。

### /run

执行 `/plan` 生成的计划文件中定义的各个 Task，按顺序完成代码改动并验证。详见 `command/run.md`。

## Provider 配置

`opencode.jsonc` 中定义了 `volcengine-plan` provider，使用 `@ai-sdk/openai-compatible` 适配器接入火山引擎编码服务。

- `baseURL`: `https://ark.cn-beijing.volces.com/api/coding/v3`
- `apiKey`: 通过环境变量 `$API_KEY` 注入（请勿硬编码真实密钥）

可用模型：

| 模型 | 上下文 | 输出 | 输入模态 |
| --- | --- | --- | --- |
| ark-code-latest | 256000 | 4096 | text, image |
| doubao-seed-code | 256000 | 4096 | text, image |
| doubao-seed-2.0-code | 256000 | 4096 | text, image |
| doubao-seed-2.0-pro | 256000 | 4096 | text, image |
| doubao-seed-2.0-lite | 256000 | 4096 | text, image |
| glm-5.2 | 1024000 | 4096 | text |
| glm-latest | 1024000 | 4096 | text |
| deepseek-v4-flash | 1024000 | 4096 | - |
| deepseek-v4-pro | 1024000 | 4096 | - |
| minimax-m2.7 | 200000 | 4096 | text |
| minimax-m3 | 512000 | 4096 | text, image |
| kimi-k2.6 | 256000 | 4096 | text, image |
| kimi-k2.7-code | 256000 | 4096 | text |

## 使用方式

1. 将本仓库内容放到 opencode 的配置目录（如 `~/.config/opencode/`）或项目 `.opencode/` 目录下。
2. 设置环境变量 `API_KEY`（火山引擎 ARK API Key）后再启动 opencode。
3. 在 opencode 中使用 `/plan` 规划任务，用 `/run` 执行计划。

## 安全说明

- 本仓库的 `opencode.jsonc` 仅使用占位符（`$API_KEY` / `YOUR_API_KEY`），**不含真实密钥**。
- 真实 API Key 通过环境变量注入，请勿将含真实 key 的配置提交到 Git。
- 详见 `.gitignore` 中对本地配置与环境变量文件的忽略规则。
