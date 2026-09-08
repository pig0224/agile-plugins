---
description: AI 辅助建设 agile-templates 新模板：四流程（A 从零手写单例 / B 派生改造单例 / C 上游脚手架引入 / D 组合模板）× 六步（位置检测 → 场景判定 → 设计问答定稿 → 骨架生成 → registry.json 登记 + check → 冒烟 → 汇报）。仅限在 agile-templates 仓库根目录使用。SDD/TDD 组织：设计定稿先行、测试基线、冒烟验收。分工红线显式例外：主会话直接执行，不委派 subagent
argument-hint: [模板名/组合名或技术栈描述]；无参进入交互设计
---

# /agile:add-template — AI 辅助建设模板

先阅读 skill `sdd-tdd-method`（在 workspace 内时）。**分工例外声明**：本命令在主会话直接执行、不委派 subagent——设计问答素材在主会话交互中产生，subagent 拿不到（SKILL 分工红线中「个别命令文件显式声明的例外」）。

## 第 0 步：使用位置检测（硬性，先于一切）

- 本命令**只能在 agile-templates 仓库根目录使用**。检测条件（当前 cwd 同时满足）：`registry.json`、`scripts/check.mjs`、`singles/` 三者存在
- 不满足 → **立即拒绝执行**，提示用户：`cd <agile-templates 仓库根目录>` 后重新运行。不猜仓库位置、不代为定位——workspace 内通常不含本仓库（模板仓是独立 git 仓库），在 workspace 下运行属于误用，直接报错避免场景歧义

## SDD/TDD 组织方式

本命令按 SDD/TDD 方法论组织，三道门不得跳过：

1. **设计定稿门（SDD）**：② 设计问答的全部关键决策（名字 / 定位 / 成员构成 / 派生起点 / 工具链）**经用户确认定稿后才动手写盘**——无确认不写文件
2. **测试基线（TDD）**：③ 每个新骨架自带至少一个可运行测试，且**骨架阶段就该跑绿**（模板是起点不是半成品）；跑不绿的骨架不得进入登记
3. **冒烟验收门**：⑤ 验收清单**先行**（先列出将验证的断言，再逐条实际执行），不得「生成完顺手看一眼」

红线不变：工作对象 = agile-templates 仓库（模板注册中心，推送即发版），**全程不 git add / 不 commit / 不 push**，完成只汇报，由人工审阅后处理。动手前先读仓库内 `CLAUDE.md`（协作红线、模板内容约定）与 `docs/registry.md`（命名规范、质量要求）。

**占位符与 `/agile:init` 语义相反**：init 把 `{{name}}`/`{{safeName}}` 替换为真实项目名；建模板必须**原样保留**占位符。

**CLI 依赖**：冒烟需支持 singles/solutions 布局与组合生成的 CLI（布局自 2.1.0 引入，更早版本不识别；生成清单断点续建语义需 ≥ 2.2.0；组合根耦合资产快照带出需 ≥ 2.3.0，随本特性发布）；执行前先 `agile --version` 核实。

## 流程总览（四流程 × 共用收口）

| 流程 | 场景 | 骨架生成方式（③） |
|---|---|---|
| **A 从零手写单例** | 新技术栈、无相近基座 | 手写 `singles/<模板名>/` 全部内容 |
| **B 派生改造单例** | 有最接近的现有单例（如 `vue3-nuxt` ← `vue3-vite`） | 复制基座 → 清理基座名残留 → 按新栈定制 |
| **C 上游脚手架引入** | 官方 create-xxx 脚手架产物模板化 | 引入产物 → 占位符化 → 中立化 → 补规范骨架 |
| **D 组合模板** | 一次平铺生成多个成员项目的系统底座（如「营销站 + 控制台 + Mock」） | 成员骨架 = 复制最接近的单例（或 A/B/C 新产物）作起点；仓库无对应技术栈单例的成员按 A 从零手写（硬性要求见 ③ 流程 D）；组合根两件套归总跨成员耦合资产（见 ③ 流程 D） |

A/B/C 产出单例模板，D 产出组合模板；**④ 登记 + 校验、⑤ 冒烟、⑥ 汇报为四流程共用收口**。

### ① 场景判定 + 前置查重

- 读 `registry.json` 现有 singles / solutions（可对照 `registry.schema.json` 理解字段）：防重名、防语义重复——先向用户确认新建内容与现有模板/组合（同栈或近栈）的定位差异
- 命名规范 `^[a-z][a-z0-9-]*$`；单例建议 `<技术栈/框架>-<变体>`（如 `vue3-nuxt`、`go-grpc`），组合建议 `<系统域>-<定位>`（如 `admin-base`）
- **流程判定**：按上表由参数与用户意图判定 A/B/C/D；C 需用户给出上游脚手架命令，D 进入成员构成问答
- **前置查重（不等 check.mjs 才拦）**：拟定名与全部模板名 / 组合名 / 既有成员名逐个核对（三段全局唯一——init 后全部平铺落盘 `projects/`，同一命名空间）
- 向用户复述「判定结果 + 计划」，作为 SDD 起点

### ② 设计问答（SDD 设计定稿）

**A / B / C（单例问答）**：

| 项 | 说明 |
|---|---|
| 模板名 | 按命名建议生成候选，人工审定 |
| description | 一句话职责（registry.json 登记与 `agile template list` 展示用） |
| language / framework | 字符串数组（可省略），如 `["TypeScript"]` / `["Vue", "Vite"]` |
| 定位差异 | 与现有模板的差异一句话 |
| 工具链 | 构建/测试/lint 命令——**写入模板前必须实际核实**（查官方文档或本地试跑），不得凭记忆写 scripts |
| 起点 | A：从零；B：基座模板名 + 改造要点；C：上游脚手架命令 + 精简策略 |

**D（组合问答——成员构成）**：

| 项 | 说明 |
|---|---|
| 组合名 | `<系统域>-<定位>` 候选，人工审定；**全局唯一** |
| description | 一句话说明该组合生成的系统 |
| 成员构成 | 成员 = `projects` 数组条目（与 singles 同形状：name + 一句话 description；language/framework 可选），**数组顺序 = 生成顺序**；成员名取职责域且**全局唯一** |
| 派生起点 | 每成员指定最接近的单例模板（复制作起点，可用 A/B/C 刚建的）；**仓库无对应技术栈单例时**（如组合引入仓库尚无的栈），该成员按**流程 A 从零手写**，并附加两条硬性要求（见 ③ 流程 D）；既无相近起点也不想从零时与用户确认——可先登记组合能力、成员内容分批补 |
| 耦合资产盘点 | 盘点跨成员共享的约定/规范/知识清单（接口契约、跨成员协作协议、统一错误码、登录态共享方案等），逐项标注 `tech`（系统怎么实现 → biz-tech-docs）或 `product`（业务规则/UI → biz-product-docs）。**归总判据：≥2 成员共享或跨成员协作协议才归总到组合根；单成员内部约定留成员项目，不上收**（见 ③ 流程 D） |

**设计定稿门**：全部答案经用户确认（「按此执行？」）后才进入 ③。

### ③ 骨架生成（按流程分支；TDD 基线）

**通用要求（四流程一致）**：

- **规范骨架三文件**（`scripts/check.mjs` 强制校验，缺一 CI 报红）：
  - `CLAUDE.md`：项目级入口索引（技术栈 / 命令速查 / 硬规则 / 规范索引）；命令速查与实际 package.json scripts / Makefile 目标一致；团队规范段指向 `../../biz-tech-docs/` 并带**「⛔ 栈领域待人工确认」**标记
  - `docs/conventions.md`：目录（初始骨架如实 + 增长建议）/ 命名 / 测试 + 团队补充约定节
  - `docs/architecture.md`：ADR 骨架（背景 / 决策 / 后果三段式）+ ADR-001 初始条目
- 前端栈模板加 `docs/ui.md`（非强制校验，check.mjs 不查）：UI 设计 token 清单空表 + Token 管理方式 + 使用规则骨架（形态对齐 vue3-vite / react-vite 现有文件）
- README（运行/测试命令）+ 构建特征文件（package.json / go.mod / pom.xml / tsconfig.json 之一）+ **至少一个可运行测试**（TDD 起点）+ .gitignore
- **模板中立原则**：预填默认值只来自模板自身选型与社区惯例，**不引入 `frameworks/<栈>/` 具体条款**——团队库领域只在项目级经 `/agile:init` 第 ④ 步确认后引入
- 目录结构如实，不预建空目录

**流程 A（从零手写）**：按设计定稿手写全部骨架内容。

**流程 B（派生改造）**：复制基座模板全部内容到 `singles/<新名>/`（占位符 `{{name}}`/`{{safeName}}` 原样保留）→ **清理基座名残留**：目录名、CLAUDE.md 技术栈段与命令速查、conventions 的命令、package.json name / go.mod module 等身份字段（保留占位符）、README 定位段 → 按新栈调整依赖与配置 → 测试跑绿。

**流程 C（上游脚手架引入）**：在工作区外临时目录跑上游 create-xxx 生成产物 → **精简**：删业务示例页与个人信息（git config、LICENSE 署名、上游品牌引用）→ **占位符化**：package.json `name` → `{{name}}`、Java 包名/目录 → `{{safeName}}`（只落在 `^[a-z][a-z0-9-]*$` 合法位置，不放进带空格/大写/引号的语法敏感位）→ **中立化**：去掉上游痕迹与个人化默认值，工具链保持脚手架原生默认 → 移入 `singles/<模板名>/` → 补规范骨架三文件 → 测试跑绿。

**流程 D（组合）**：

1. 为每个成员建 `solutions/<组合名>/<成员名>/` ← **复制其派生起点单例模板的全部内容**（占位符原样保留；规范骨架三文件随复制带出，复制后确认仍在）。**仓库无对应技术栈单例的成员**（如组合引入仓库尚无的栈）：不强行复制不相干的基座——该成员改按**流程 A 从零手写**，并附加两条硬性要求：
   - **依赖版本必须实查**：经 `npm dist-tags` / 官方文档核实当前稳定版后写入，不得凭记忆（② 工具链「实际核实」要求扩展到依赖版本）
   - **工程配置形态对齐上游官方脚手架默认输出**（如 `create-next-app` / `create-vite` 的产物形态），保持「模板产物 = 脚手架同构 + 规范骨架叠加」，不凭记忆自造 tsconfig / lint 等配置
2. 建**组合根耦合资产两件套**（`scripts/check.mjs` 契约 14 强制；workspace「1 根 5 抽屉」范式的模板侧保障——跨成员知识不入成员项目，平铺生成后各成员才不会带副本散落 `projects/`）：
   - `solutions/<组合名>/CLAUDE.md`：组合定位一句话 + 成员清单（成员名 → 职责，**不写具体落盘目录名**——`--name` 键值覆盖在生成时才确定）+ 耦合资产导航（docs/ 每篇一行：标题 + 类型 + 摘要）
   - `solutions/<组合名>/docs/`：按 ② 耦合资产盘点清单逐篇落盘，每篇顶部 frontmatter 标 `类型: tech|product`（`/agile:knowledge sync-template` 按此同步进抽屉：tech → biz-tech-docs，product → biz-product-docs）
   - **迁移纪律**：成员骨架中已存在的跨成员共享内容迁入组合根 docs/，成员侧不保留正文副本（成员 CLAUDE.md 团队规范段本就指向 `../../biz-tech-docs/`，同步后知识在抽屉可查）；**组合根资产不做占位替换**——不出现 `{{name}}` / `{{safeName}}`（check.mjs 会拦）
3. 做**最小组合级定制**（按问答中的成员职责定位）：各成员 README / CLAUDE.md 首段改写为组合语境的定位描述（含成员间协作约定一句话）；不预造业务功能
4. 组合专属深度定制（新页面、新接口、成员间协议等）**按实际需求另行开发**（SDD/TDD 流程或人工），本命令只负责把组合结构与登记打通；定制期间保持成员测试可跑（起点自带）

**③ 出口检查（测试基线）**：每个新骨架目录内测试实际跑绿后，才允许进入 ④。**跑绿后、进入 ④ 前，清理成员/模板目录内的安装与构建产物**——产物不入库（`scripts/check.mjs` 契约 11 产物黑名单全树强制拦截，CI 会红），但会污染后续流程：CLI 直读模板仓复制时会受产物干扰（≤ 2.1.0 撞 junction 直接崩溃 EISDIR，agile-cli issue #<编号，由维护者填>；≥ 2.2.0 复制侧已修复为自动忽略产物，模板目录保持无产物仍是基线要求）。按栈清理实际产生的产物，常见清单：`node_modules`、`.next`、`dist`、`build`、`coverage`、`pnpm-lock.yaml`、`package-lock.json`、`next-env.d.ts`——完整黑名单以 `scripts/check.mjs` 契约 11 为准（另含 `.turbo`、`.vitest`、`yarn.lock`、`*.tsbuildinfo`，符号链接/junction 一并报错）。示例命令（对每个成员/模板目录执行，路径换成实际目录）：

```powershell
# PowerShell 5.1（在 agile-templates 仓库根目录执行）
$artifacts = 'node_modules', '.next', 'dist', 'build', 'coverage', 'pnpm-lock.yaml', 'package-lock.json', 'next-env.d.ts'
$artifacts | ForEach-Object { $p = Join-Path 'solutions/<组合名>/<成员名>' $_; if (Test-Path $p) { Remove-Item $p -Recurse -Force -Confirm:$false } }
```

```bash
# POSIX bash（brace expansion；rm -rf 对不存在的路径静默跳过）
rm -rf solutions/<组合名>/<成员名>/{node_modules,.next,dist,build,coverage,pnpm-lock.yaml,package-lock.json,next-env.d.ts}
```

### ④ 登记 + 校验（共用收口）

**单例（A/B/C）**——`registry.json` 的 `singles` 数组登记（无 path 字段，目录由名字派生）：

```json
{
  "name": "<模板名>",
  "description": "<一句话职责>",
  "language": ["<语言>"],
  "framework": ["<框架>"]
}
```

**组合（D）**——`registry.json` 的 `solutions` 数组登记（成员**不**登记进 `singles`）：

```json
{
  "name": "<组合名>",
  "description": "<系统定位一句话>",
  "projects": [
    { "name": "<成员名>", "description": "<一句话职责描述>" }
  ]
}
```

数组顺序 = 生成顺序；成员名 `^[a-z][a-z0-9-]*$` 且全局唯一。

`node scripts/check.mjs` 全绿——重点覆盖：条目形状与未知字段、JSON 重复键、数组重复登记（singles / solutions / 同组合 projects）、目录派生存在性、登记与成员目录**双向一致**（缺成员目录 / 幽灵成员目录均报错；组合根 `docs/` 豁免——它是耦合资产目录不是成员）、成员名**全局唯一**（vs 模板 / 组合名 / 其他组合成员）、规范骨架三文件、根一级目录白名单、模板内容卫生三项（产物黑名单 / package.json name 占位符 / README 测试命令存在性）、组合根耦合资产两件套（D：CLAUDE.md + docs/ 存在性、docs/*.md 逐篇 `类型: tech|product` frontmatter、组合根资产占位符禁用）。

### ⑤ 冒烟验证（共用收口，验收清单先行）

先核实 CLI ≥ 2.2.0（`agile --version`），不满足则提示升级后重试。**再核实模板目录无安装产物残留**（③ 出口检查清理的兜底复核）——④ 的 `node scripts/check.mjs` 已按契约 11 全树强制扫描产物黑名单（产物目录 / 锁文件 / `*.tsbuildinfo` / 符号链接，含 `.turbo`、`.vitest`），check 全绿即无残留，无需重复手工扫描。

**先向用户列出本次冒烟断言清单（按流程选取下述验证点），经确认后逐条实际执行。**

**单例（A/B/C）**：

```bash
mkdir ../tpl-smoke && cd ../tpl-smoke
agile init workspace
agile config set template-repo <agile-templates 本地绝对路径>   # 直读不走缓存
agile init project --template <新模板名> --name demo-<模板名>
```

验证三件事：生成物完整（规范骨架三文件带出）、`{{name}}` / `{{safeName}}` 替换正确、项目测试可跑（`npm test` / `make test` 等）。

**组合（D）**：

```bash
mkdir ../tpl-smoke && cd ../tpl-smoke
agile init workspace
agile config set template-repo <agile-templates 本地绝对路径>
agile template list                  # 组合应展示成员清单（含各成员 description）
agile init project --template <组合名>   # 缺省 --name：各成员用组合项目名称
```

验证六件事：全部成员**平铺**落盘 `projects/<成员名>/`（无系统目录、无系统 README）、`{{name}}` = 实际成员目录名（抽查各成员 package.json name 等）、成员项目测试可跑、`--name <成员名>=<目录名>` 键值覆盖可生效（可选抽查）、**重跑同一 init** 补缺语义正确（已存在成员跳过 + warn；本地 CLI 支持生成清单时可加测：删除某成员一个生成文件后重跑，应报「与生成清单不符」硬错误而非静默跳过）、**组合根两件套快照带出**（`.agile/solutions/<组合名>/` 出现 CLAUDE.md + docs/，init 输出对应提示行与 `/agile:knowledge` 同步指引；需 ≥ 2.3.0 CLI——组合资产带出随该版本发布，`agile --version` 低于该版本时此项跳过并注明）。完成后清理临时目录。

### ⑥ 汇报（共用收口）

文件清单（组合附「成员 → 派生起点」映射表与组合根两件套内容清单）+ 设计定稿摘要 + check / 冒烟结果 + registry.json 变更说明（单例 singles 数组 / 组合 solutions 数组）+ 提醒人工 add / push（本仓推送即发版，人工处理）。
