---
description: AI 辅助建设 agile-templates 新模板：设计问答（栈/变体/工具链/是否派生）、骨架生成（含规范骨架三文件、占位符原样保留）、registry.yaml 登记、check.mjs 校验、init project 冒烟验证。在 agile-templates 仓库内使用。分工红线显式例外：设计问答在主会话交互完成，本命令不委派 subagent、主会话直接执行
argument-hint: [模板名或技术栈描述]；无参进入交互设计
---

# /agile:add-template — AI 辅助建设模板

先阅读 skill `sdd-tdd-method`（在 workspace 内时）。**分工例外声明**：本命令在主会话直接执行、不委派 subagent——设计问答素材在主会话交互中产生，subagent 拿不到（SKILL 分工红线中「个别命令文件显式声明的例外」）。

## 适用与红线

- 工作对象 = agile-templates 仓库（模板注册中心，推送即发版）：**全程不 git add / 不 commit / 不 push**，完成只汇报，由人工审阅后处理
- 动手前先读仓库内 `CLAUDE.md`（协作红线、模板内容约定）与 `docs/registry.md`（命名规范、质量要求）
- **占位符与 `/agile:init` 语义相反**：init 把 `{{name}}`/`{{safeName}}` 替换为真实项目名；建模板必须**原样保留**占位符

## 流程（六步）

### ① 前置

- 定位 agile-templates 仓（workspace 内或独立检出均可）
- 读 `registry.yaml` 现有模板：防重名、防语义重复——先向用户确认新模板与现有模板（同栈或近栈）的定位差异
- 命名规范 `^[a-z][a-z0-9-]*$`，建议 `<技术栈/框架>-<变体>`（如 `vue3-nuxt`、`go-grpc`）

### ② 设计问答

| 项 | 说明 |
|---|---|
| 模板名 | 按命名建议生成候选，人工拍板 |
| language / framework | registry.yaml 登记用 |
| 定位差异 | 与现有模板的差异一句话 |
| 工具链 | 构建/测试/lint 命令——**写入模板前必须实际核实**（查官方文档或本地试跑），不得凭记忆写 scripts |
| 是否派生 | 从现有模板复制改造（如 vue3-nuxt ← vue3-vite）或从零手写 |

### ③ 生成骨架（新建 `<模板名>/` 目录）

- **项目骨架**：手写或派生改造；目录结构如实，不预建空目录
- **规范骨架三文件**（`scripts/check.mjs` 强制校验，缺一 CI 报红）：
  - `CLAUDE.md`：项目级入口索引（技术栈 / 命令速查 / 硬规则 / 规范索引）；命令速查与实际 package.json scripts / Makefile 目标一致；团队规范段指向 `../../biz-tech-docs/` 并带**「⛔ 栈领域待人工确认」**标记
  - `docs/conventions.md`：目录（初始骨架如实 + 增长建议）/ 命名 / 测试 + 团队补充约定节
  - `docs/architecture.md`：ADR 骨架（背景 / 决策 / 后果三段式）+ ADR-001 初始条目
- README（运行/测试命令）+ 构建特征文件（package.json / go.mod / pom.xml / tsconfig.json 之一）+ **至少一个可运行测试**（TDD 起点）+ .gitignore
- **模板中立原则**：预填默认值只来自模板自身选型与社区惯例，**不引入 `frameworks/<栈>/` 具体条款**——团队库领域只在项目级经 `/agile:init` 第 ④ 步确认后引入

### ④ 登记 + 校验

- `registry.yaml` 的 `templates:` 下登记（name / description / language / framework / path）
- `node scripts/check.mjs` 全绿（命名 / 目录同名 / 唯一性 / path 合法 / 重复键 / 三文件存在性）

### ⑤ 冒烟验证（必须实际执行，不得跳过）

```bash
mkdir ../tpl-smoke && cd ../tpl-smoke
agile init workspace
agile config set template-repo <agile-templates 本地绝对路径>   # 直读不走缓存
agile init project demo-<模板名> --template <新模板名>
```

验证三件事：生成物完整（规范骨架三文件带出）、`{{name}}` / `{{safeName}}` 替换正确、项目测试可跑（`npm test` / `make test` 等）。完成后清理临时目录。

### ⑥ 汇报

文件清单 + check / 冒烟结果 + registry.yaml 变更说明 + 提醒人工 add / push（本仓推送即发版）。
