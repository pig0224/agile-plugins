---
description: AI 辅助把 workspace 既有项目打包为 agile-templates 模板（/agile:add-template 的逆过程）：盘点 → 方案问答 → 清理审计（敏感数据/团队定制/工程卫生处置清单）→ 打包落盘 + registry.json 登记 → check + 冒烟 → 汇报移交。仅限在 agile workspace 内使用。分工红线显式例外：主会话直接执行，不委派 subagent
argument-hint: "[项目目录名...]；无参扫 projects/ 盘点"
---

# /agile:share-template — 把 workspace 项目打包成模板

`/agile:add-template` 的逆过程姊妹命令：add-template 把想法建成模板（正向建设），本命令把 workspace 既有项目沉淀成可共享模板（反向打包），解决「现有项目想共享模板需人工搬运」。**分工例外声明**：本命令在主会话直接执行、不委派 subagent——方案问答与处置清单的素材在主会话交互中产生，subagent 拿不到（SKILL 分工红线中「个别命令文件显式声明的例外」）。

## 第 0 步：使用位置检测（硬性，先于一切）

- 本命令**只能在 agile workspace 内使用**（与 add-template 相反——源项目在 `projects/` 下）。检测条件：当前 cwd 及父级存在 `.agile/settings.json`
- 不满足 → **立即拒绝执行**，提示用户在 workspace 根目录重新运行。不猜 workspace 位置、不代为定位
- **源项目只读铁律**：`projects/` 下的源项目全程只读——脱敏、中立化、占位符还原等一切修改只发生在目标仓副本，绝不回写源项目

## 三道门（SDD/TDD 组织方式）

本命令按 SDD/TDD 方法论组织，三道门不得跳过：

1. **方案定稿门（SDD）**：② 方案问答的全部决策（打包形态 / 命名 / 目标仓 / description）**经用户确认定稿后才动目标仓**——无确认不写文件
2. **处置确认门**：③ 清理审计产出的处置清单**逐项经用户确认后才执行**——清理审计是本命令相对手工搬运的核心价值，不得跳过审计直接打包
3. **冒烟验收门**：⑤ 验收清单**先行**（先列出将验证的断言，再逐条实际执行），不得「打包完顺手看一眼」

红线不变：工作对象含目标模板仓（模板注册中心，**推送即发版**），**全程不 git add / 不 commit / 不 push**，完成只汇报，由人工审阅后处理。动手写目标仓前先读其 `CLAUDE.md`（协作红线、模板内容约定）与 `docs/registry.md`（命名规范、质量要求）。

**占位符语义（与 `/agile:init` 相反、与 add-template 同向）**：init 把 `{{name}}`/`{{safeName}}` 替换为真实目录名；本命令做**逆向还原**——真实目录名改回 `{{name}}`、安全段改回 `{{safeName}}`，package.json `name` 严格为 `{{name}}`（目标仓 `scripts/check.mjs` 契约 12 强制）。

**沉淀知识去向约定**（打包什么、不打包什么）：

- 项目内沉淀（`docs/conventions.md`、`docs/architecture.md`、`docs/ui.md`、README）随项目打包
- 跨成员共享的约定/规范归总到组合根 `docs/`（组合场景；使用方经 `/agile:knowledge sync-template` 同步进抽屉，与 init 的组合根快照带出链路构成闭环）
- **workspace 抽屉知识（biz-tech-docs / biz-product-docs / tech-specs）不直接打进模板**——模板中立原则 + 防知识泄露；确需共享的先沉淀进项目文档或组合根 docs/ 再打包

**CLI 依赖**：本命令不新增 CLI 能力（打包全由 AI 完成）；冒烟验证需支持 singles/solutions 布局的 CLI（布局自 2.1.0 引入，更早版本不识别；生成清单断点续建语义需 ≥ 2.2.0；组合根耦合资产快照带出需 ≥ 2.3.0）。执行前先 `agile --version` 核实。

## 流程总览（六步）

① 盘点与目标仓定位 → ② 方案问答（定稿门）→ ③ 清理审计（处置门）→ ④ 打包落盘 + registry.json 登记 → ⑤ 校验 + 冒烟（验收门）→ ⑥ 汇报移交

### ① 盘点 + 前置查重 + 目标仓定位

- 参数给出项目目录名则直接核对存在性；无参则扫 `projects/` 一层列候选，读 `.agile/manifests/<目录名>.json` 标注出身：单例模板 X 生成 / 组合 Y 的成员 Z / 陌生目录（手写项目或旧版生成）
- **目标仓定位**：请用户给出本地检出的 agile-templates 仓路径（官方仓检出 / fork / 团队私有仓均可）；可探测建议——settings.json `templates.registry` 为本地路径时用它、workspace 兄弟目录 `../agile-templates` 存在时提示之，**均须用户确认**。校验三件：`registry.json` + `scripts/check.mjs` + `singles/` 同时存在，否则拒绝
- 读目标仓 `registry.json` 现有登记：拟定名与全部模板名 / 组合名 / 成员名逐个核对（**三段全局唯一**——init 后全部平铺落盘 `projects/`，同一命名空间）；对照现有模板/组合确认定位差异（防语义重复）
- 向用户复述「出身盘点 + 目标仓 + 计划」，作为 SDD 起点

### ② 方案问答（SDD 定稿）

| 项 | 说明 |
|---|---|
| 打包形态 | 1 个项目 = **单例模板**；多个项目 = **组合模板**。组合成员名默认沿用生成清单中的原成员名（`manifest.member`），无清单用目录名，均可覆盖 |
| 模板名 / 组合名 / 成员名 | `^[a-z][a-z0-9-]*$`；单例建议 `<技术栈/框架>-<变体>`（如 `go-zero-api`），组合建议 `<系统域>-<定位>`（如 `admin-base`）；**目标 registry 三段全局唯一** |
| description | 一句话职责（registry.json 登记与 `agile template list` 展示用） |
| language / framework | 字符串数组（可省略），如 `["TypeScript"]` / `["Vue", "Vite"]` |
| 目标仓形态 | 官方公共仓 / fork / 团队私有仓——**决定 ③ 处置默认建议**（公共仓默认中立化，私有仓默认保留团队定制） |

**方案定稿门**：全部答案经用户确认（「按此执行？」）后才进入 ③。

### ③ 清理审计（处置清单——核心价值）

逐项审阅源项目内容，产出**处置清单表**（位置 → 处置 → 理由）交用户确认。三个维度：

| 维度 | 检查点 | 默认处置 |
|---|---|---|
| 敏感与业务数据 | `.env` 真实值、硬编码密钥/凭据/内网地址、真实用户数据与业务数据、业务专属 mock、LICENSE/配置中的个人信息 | 剔除或脱敏（改占位样例） |
| 团队定制 | CLAUDE.md 已确认的团队规范段（含确认人/日期）、conventions 团队补充节、docs/ui.md 团队 token 值、`frameworks/<栈>/` 条款引用、真实 ADR 决策记录 | 公共仓：**中立化**（恢复「⛔ 栈领域待人工确认」标记与模板初始骨架语义）；私有仓：默认**保留**（团队实践共享正是打包价值）——逐项用户定夺 |
| 工程卫生 | `node_modules` / `.next` / `dist` / `build` / `coverage` 等产物、锁文件（pnpm-lock.yaml / package-lock.json / yarn.lock / *.tsbuildinfo）、`.git`、`.claude/` 本地设置、IDE 文件、符号链接/junction | 一律剔除（check.mjs 契约 11 产物黑名单全树强制拦截） |

辅助检测示例（源项目根执行，结果并入处置清单）：

```bash
# 疑似密钥/凭据扫描（结果逐条人工判断，不盲信）
grep -rn -iE "(password|secret|token|api[_-]?key)\s*[=:]" --include="*.ts" --include="*.js" --include="*.json" src/
```

```powershell
# PowerShell 5.1：env 类文件与产物目录盘点
Get-ChildItem -Recurse -Force -Include .env,.env.* -File | Select-Object -ExpandProperty FullName
Get-ChildItem -Recurse -Directory -Include node_modules,dist,build,coverage,.next | Select-Object -ExpandProperty FullName
```

**处置确认门**：清单经用户确认后才进入 ④。

### ④ 打包落盘 + registry.json 登记

**落盘**：源项目复制到目标仓 `singles/<模板名>/`（单例）或 `solutions/<组合名>/<成员名>/` × N（组合），复制时按处置清单执行剔除。示例命令（路径换成实际值）：

```powershell
# PowerShell 5.1：复制后剔除（源项目不动，剔除只发生在目标仓副本）
Copy-Item -Recurse projects\<项目目录> ..\agile-templates\singles\<模板名>
$strip = 'node_modules', '.next', 'dist', 'build', 'coverage', '.git', '.claude', '.idea', '.vscode', 'pnpm-lock.yaml', 'package-lock.json', 'yarn.lock'
$strip | ForEach-Object { $p = Join-Path '..\agile-templates\singles\<模板名>' $_; if (Test-Path $p) { Remove-Item $p -Recurse -Force -Confirm:$false } }
```

```bash
# POSIX bash：rsync 边复制边剔除（尾斜杠语义 = 复制目录内容）
rsync -a --exclude=node_modules --exclude=.next --exclude=dist --exclude=build --exclude=coverage \
  --exclude=.git --exclude=.claude --exclude=.idea --exclude=.vscode \
  --exclude=pnpm-lock.yaml --exclude=package-lock.json --exclude=yarn.lock \
  projects/<项目目录>/ ../agile-templates/singles/<模板名>/
```

**占位符逆向还原**（复制后对目标目录执行）：

- 已知映射：实际落地目录名 → `{{name}}`；其安全段（小写字母数字折叠，如 `order-service` → `orderservice`）→ `{{safeName}}`
- 替换范围：文本内容与目录/文件名（Java 包路径 `src/main/java/com/example/orderservice/` → `.../{{safeName}}/`）
- **上下文判断**：包名、go module、目录名、文档标题、配置默认值等**身份性出现**必换；URL 路径、文案叙述中的**语义性出现**逐处判断——拿不准的列清单问用户（宁缺勿滥：漏换只损失通用性，错换产出错误模板）
- package.json `name` 严格改为 `{{name}}`；完成后对目标目录扫实际名残留（`grep -rn "<目录名>" .` / `Get-ChildItem -Recurse | Select-String "<目录名>"`），残留逐处定夺

**规范骨架复核**（check.mjs 契约 7）：`CLAUDE.md` / `docs/conventions.md` / `docs/architecture.md` 必须在位——源自模板的项目天然有（注意中立化后「⛔ 栈领域待人工确认」标记已恢复）；手写项目缺则按 `/agile:add-template` ③ 通用要求补齐；README 须含测试命令（契约 13）。

**组合根两件套**（组合场景，check.mjs 契约 14）：

- workspace 存在 `.agile/solutions/<原组合名>/` 快照（`init project` ≥ 2.3.0 带出）→ 整体复制回目标仓 `solutions/<组合名>/{CLAUDE.md,docs/}`（**不做占位替换**——组合根资产禁含 `{{name}}`/`{{safeName}}`），并审阅内容是否仍准确（组合更名时逐处核对旧名引用）
- 无快照 → 按契约 14 现建：`solutions/<组合名>/CLAUDE.md`（组合定位一句话 + 成员清单（成员名 → 职责，**不写具体落盘目录名**——`--name` 键值覆盖在生成时才确定）+ 耦合资产导航）+ `solutions/<组合名>/docs/` 逐篇落盘（归总判据：≥2 成员共享或跨成员协作协议；每篇顶部 frontmatter 标 `类型: tech|product`——`/agile:knowledge sync-template` 按此同步进抽屉）
- **迁移纪律**：成员骨架中已存在的跨成员共享内容迁入组合根 docs/，成员侧不保留正文副本（同 add-template 流程 D）

**registry.json 登记**：**只追加、不重排/不改既有条目**（diff 最小化）。单例 → `singles` 数组；组合 → `solutions` 数组 + `projects` 成员数组（**数组顺序 = 生成顺序**；成员不登记进 `singles`）。条目形状同 add-template ④（name + 一句话 description 必填，language / framework 可选）。

### ⑤ 校验 + 冒烟（验收清单先行）

`node scripts/check.mjs`（目标仓根目录）全绿——重点覆盖：条目形状与未知字段、JSON 重复键、数组重复登记、目录派生存在性、登记与成员目录**双向一致**（组合根 `docs/` 豁免）、三段全局唯一、规范骨架三文件、内容卫生三项（产物黑名单 / package.json name 占位符 / README 测试命令）、组合根两件套（组合场景：CLAUDE.md + docs/ 存在性、逐篇 `类型: tech|product` frontmatter、占位符禁用）。

**先向用户列出本次冒烟断言清单，经确认后逐条实际执行**（临时 workspace + 目标仓本地路径作模板源，直读不走缓存）：

```bash
mkdir ../tpl-smoke && cd ../tpl-smoke
agile init workspace
agile config set template-repo <目标仓本地绝对路径>
agile init project --template <模板名或组合名>
```

- **单例验证三件事**：生成物完整（骨架三文件带出）、`{{name}}` / `{{safeName}}` 替换正确、项目测试可跑
- **组合验证六件事**（同 add-template ⑤）：成员平铺落盘、占位符替换抽查、成员测试可跑、`--name` 键值覆盖可生效（可选）、重跑补缺语义正确、组合根两件套快照带出（需 CLI ≥ 2.3.0，低于则跳过并注明）
- **round-trip 对照**：再生成项目与源项目关键结构对照（目录树 / 关键文件 / README 命令），确认打包无遗漏
- 完成后清理临时目录

### ⑥ 汇报移交

- 目标仓变更清单（新增模板目录 / registry.json 追加条目）+ 处置清单执行结果（剔除/脱敏/中立化逐项）+ check 与冒烟结果
- 沉淀知识去向说明：项目内沉淀随打包；跨成员规范在组合根 docs/（使用方经 `/agile:knowledge sync-template` 进抽屉）；workspace 抽屉知识未入模板
- **移交红线**：AI 不 add / 不 commit / 不 push——目标仓变更由人工审阅后处理；**模板仓 push 即发版**（无 npm、无 tag，使用方拉到即生效），发布步骤见文档站「模板开发指南 · 导出后如何发布」
- 使用方验证路径：`agile template update`（或 `agile sync`）→ `agile template list` → `agile init project --template <名>`
