---
description: 生成任务计划文档，保存到当前项目 .opencode/plan/ 目录下
---

生成任务计划文档，并保存为 `.md` 文件到当前项目根目录下的 `.opencode/plan/` 目录。

**输入**：`$ARGUMENTS` 是用户的需求描述。如果为空，使用 **question 工具**（开放式，无预设选项）询问用户：

> "你想规划什么任务？描述你想要实现或修改的内容。"

在理解需求前不要继续。

---

## 执行流程

### 1. 只读探索（不写任何业务代码）

使用 `glob`、`grep`、`read` 等只读工具调研与需求相关的代码、配置、文档，理解：

- 当前项目结构与技术栈
- 涉及的现有文件及其内容、模式、约定
- 需要参照的对标文件（如同类功能、相似模块的实现）
- 配置参数的来源（从哪个文件读取、哪个常量定义）

充分探索后再动笔写 plan，避免凭空猜测。

### 2. 生成文件名

格式：`YYYY-MM-DD-<task-name>.md`

- 日期：取当天（如 `2026-06-21`）
- `<task-name>`：根据需求内容生成简短的 kebab-case 名称（如 `admin-start-prod`、`add-user-auth`、`fix-booking-pagination`）
- 完整示例：`2026-06-21-admin-start-prod.md`

### 3. 生成 plan 文档

严格遵循以下 Markdown 结构（每一节都要有，顺序固定）：

```markdown
# <任务简短标题>

## 现状

<描述当前状态、背景、为什么需要改动。说明涉及的文件/模块的现状，缺失什么，为什么不满足需求。>

## Task 1: <任务简述>

<详细说明这个任务要做什么。如创建某文件、修改某配置、新增某函数。>

<如果涉及代码，给出完整的代码示例，用带语言标记的代码块：>

```js
const path = require('path');
// ... 完整代码
```

<对于关键参数/配置项，用列表说明其来源依据：>

- `<参数名>` -- <来源说明，如"来自 module-federation.config.js 中的定义">
- `<参数名>` -- <来源说明，如"来自 webpack.config.js 中的 publicPath">

## Task 2: <任务简述>

<同上结构：详细说明 + 代码示例 + 参数来源>

## Task 3: <任务简述>

<如有更多任务，继续递增编号>
```

### 4. 保存文件

- 确保当前项目根目录下存在 `.opencode/plan/` 目录（不存在则用 bash 工具创建：`New-Item -ItemType Directory -Path ".opencode\plan" -Force`，`workdir` 设为项目根目录）
- 使用 **write 工具**将 plan 内容写入 `.opencode/plan/YYYY-MM-DD-<task-name>.md`（绝对路径，基于当前项目根目录）

### 5. 输出摘要

保存完成后，在对话中输出简短摘要：

- plan 文件的绝对路径
- 简述 plan 包含的任务数量与核心改动
- 提示用户：可查看/编辑该 plan 文件，确认后再决定如何执行

不要在对话中重复完整 plan 内容（文件里已有）。

---

## 格式约束（必须严格遵守）

1. **一级标题 `#`**：任务的简短标题，一句话概括
2. **必含 `## 现状` 章节**：紧跟标题之后，描述背景与现状
3. **分任务用 `## Task N: <描述>`**：N 从 1 递增，每个任务一个二级标题
4. **代码示例带语言标记**：```js、```json、```tsx、```bash 等，不要用无标记代码块
5. **关键参数说明来源**：用 `- \`参数名\` -- <来源>` 列表项，让用户知道每个值从哪来
6. **不执行任何业务代码改动**：本命令只生成 plan 文档，不修改项目源码、配置、Git 状态
7. **基于真实探索**：plan 中的文件路径、函数名、配置项必须来自第 1 步的只读探索，不要凭空编造

---

## 参考示例

用户输入 `/plan 为 admin 包补充 start:prod 生产启动命令`，生成的 plan 文件 `2026-06-21-admin-start-prod.md` 应类似：

```markdown
# 为 admin 包补充 start:prod 生产启动命令

## 现状

`packages/admin` 是 `packages/` 目录下唯一缺少 `start:prod` 脚本的包，同时缺少 `config.js` 和 `service.js`。

## Task 1: 创建 `packages/admin/config.js`

参照 `packages/user/config.js` 的模式，创建配置文件：

```js
const path = require('path');
const fs = require('fs');

const appDirectory = fs.realpathSync(process.cwd());
const resolveApp = relativePath => path.resolve(appDirectory, relativePath);

module.exports = {
  appBuildPublicPath: "/child/admin",
  appBuild: resolveApp(resolveApp("dist")),
  port: 3004
}
```

- `port: 3004` -- 来自 `module-federation.config.js` 中的定义
- `appBuildPublicPath: "/child/admin"` -- 来自 `webpack.config.js` 中的 `publicPath`

## Task 2: 创建 `packages/admin/service.js`

与 `packages/user/service.js` 完全相同的 Express 静态服务：

```js
const express = require('express');
const path = require('path');
const config = require('./config');

const app = express();
app.use('*', function (req, res, next) {
  next();
});

app.use(config.appBuildPublicPath, express.static(config.appBuild));

app.get('/*', function (req, res) {
  res.sendFile(path.join(config.appBuild, 'index.html'));
});
app.listen(config.port, () => console.log(`app listening at http://localhost:${config.port}`));
```

## Task 3: 修改 `packages/admin/package.json`

1. 在 `scripts` 中添加 `start:prod`：
   ```json
   "start:prod": "node ./service.js"
   ```

2. 在 `devDependencies` 中添加 `express` 依赖（与其他子应用一致，使用 `4.18.2`）：
   ```json
   "express": "4.18.2"
   ```
```

---

## 行为准则

- 只读探索要充分，避免凭空猜测文件路径或函数签名
- plan 文档是给用户审阅的，要清晰、具体、可执行
- 代码示例要完整可复制，不要省略关键部分
- 参数来源说明核心特征，必须保留
- 文件名简短且有语义，便于检索
