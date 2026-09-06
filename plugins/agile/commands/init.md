---
description: AI 陪同初始化项目：模板选择与骨架生成、项目约定问答定制（目录/命名/测试）、团队库匹配人工确认、辅助开发能力配置（环境检测、框架 AI 能力包推荐）。无参数——自动感知刚由 CLI 创建的未定制项目并续接，或进入全新创建交互。可由 AI 主动调用（用户说"建个项目"时）。分工红线显式例外：模板选择与全部问答素材在主会话上下文中，本命令不委派 subagent、主会话直接执行
---

# /agile:init — AI 陪同初始化项目

先阅读 skill `sdd-tdd-method`（在 workspace 内时）。**分工例外声明**：本命令在主会话直接执行、不委派 subagent——模板选择与全部问答素材在主会话交互中产生，subagent 拿不到（SKILL 分工红线中「个别命令文件显式声明的例外」）。

CLI `agile init project` 保持机器语义（原样生成模板默认值）；本命令在其上叠加 AI 问答定制与辅助能力配置。

## 三原则

1. **默认值兜底**：开头一问「定制 or 全默认」（推荐全默认）；选全默认则产物与纯 CLI init 完全一致——问答不是门槛
2. **分级落盘**（见第 ⑤ 步）：只读检测直接做；写文件类经确认后 AI 落地；安装类（skills、权限白名单）只输出建议清单，人工自己配
3. **定制不回写模板**：项目个性留在项目内；高频定制走 `/agile:knowledge capture` 反馈模板演进（人工决定）

## 流程（六步）

### ① 场景感知（无参数，自动定位）

- 必须在 agile workspace 内（读 `.agile/settings.json` 的 `paths`），否则提示并停止
- **续接刚创建的项目**：扫 `projects/` 下 CLAUDE.md 团队规范段仍带「⛔ 栈领域待人工确认」标记的项目（= CLI 刚生成、未走完本命令定制的）；其技术栈节自带来源模板（如「模板 vue3-vite 生成」），无需额外元数据
  - 恰好一个 → 展示项目名与来源模板，经确认后**跳过第 ② 步，直接进 ③**
  - 多个 → 列出让用户挑
- **全新创建**：无候选 → 运行 `agile template list`，结合 registry 的 language / framework 字段与用户需求交互选定模板、确定项目名，执行 ②

### ② 生成骨架（仅全新创建执行；感知续接的项目跳过本步）

```bash
agile init project <name> --template <模板名>
```

（CLI 自动 git add 生成物；commit 时机由人工决定。）

### ③ 项目约定问答（≤2 轮 AskUserQuestion）

每题预选项 = 模板预填值。维度：

| 维度 | 问什么 |
|---|---|
| 目录结构 | 在模板「增长时的建议布局」上增删（如要不要 stores/、api/ 层，页面目录 views 还是 pages） |
| 命名细节 | 测试文件后缀（.spec vs .test）、组件/Hook 命名偏好 |
| 测试约定 | 覆盖要求、e2e 关键路径清单（前端模板） |
| 依赖选型 | 引入的组件库/关键依赖（供第 ⑤ 步检索框架 AI 能力） |

按回答改写 `docs/conventions.md`（「增长时的建议布局」→「本项目约定」等）与 `CLAUDE.md` 硬规则；**未问到的维度保留模板默认值**。

### ④ 团队库匹配确认（SKILL 硬规则「团队库匹配人工确认」的流程化执行）

- 列出 `biz-tech-docs/frameworks/` 实际存在的目录，按项目技术栈给出建议匹配项，**AI 提问、人工回答**
- 确认 → 把项目 `CLAUDE.md` 团队规范段改写为具体领域路径 + 确认人与日期
- 确认无匹配 → 显式提示缺口（`/agile:knowledge capture` 沉淀或 tech-specs 提案），不臆造领域名
- 用户明确「暂不确认」→ 保留「⛔ 待人工确认」标记，本会话不再重复发起；后续会话首次需要栈领域时按 SKILL 硬规则触发
- **未确认前只引用通用领域**；此状态决定第 ⑤ 步推荐来源（见下）

### ⑤ 辅助开发能力配置（四级）

| 级别 | 项 | 动作 |
|---|---|---|
| 事实登记（不问，只读） | 环境检测：`node / go / java / mvn / make / git` 版本 + Playwright 浏览器 | 结果写入 CLAUDE.md 新增「环境要求」节（工具 + 最低版本 + 本机实况）；缺失项分流：npm 能装的进入下行推荐，JDK/Maven 类列入人工待办 |
| 确认制，AI 自动安装 | 框架 AI 能力包之 CLI（含 Playwright 浏览器 `npx playwright install chromium`） | 每项单独确认后执行；优先项目 devDep + `npx` 调用（版本锁进 package.json），须全局安装的先说明影响 |
| 确认制，AI 写文件 | 能力包之 MCP / design.md / llms.txt | MCP → 写项目 `.mcp.json`；design.md / llms.txt → 登记 CLAUDE.md「辅助开发配置」节 + 用途（如「AI 生成 UI 前读 design.md」） |
| 仅建议，人工配 | skills 安装、权限白名单 | 输出复制即用的命令/清单进汇报；白名单只放测试/构建/只读类命令；不写任何配置 |

**框架 AI 能力包推荐来源**（按序取第一个可用）：

1. **团队库**（仅当第 ④ 步已确认栈领域）：读 `frameworks/<栈>/` 内团队认可的推荐
2. **命令内置映射**（下表，URL/包名于 2026-09-06 核实可达；使用时失联即弃、现场检索替代）
3. **现场检索**：WebSearch 核实存在才推荐（防幻觉包名），拿不准的列「待你提供」

| 栈 / 库 | 条目 |
|---|---|
| Vue | llms.txt `https://vuejs.org/llms.txt` |
| Vite（vue3-vite / react-vite 构建层） | llms.txt `https://vite.dev/llms.txt` |
| React | llms.txt `https://react.dev/llms.txt` |
| Node.js | llms.txt `https://nodejs.org/llms.txt` |
| Ant Design（组件库选型后） | AI 能力包：CLI `npm install -g @ant-design/cli`、MCP `npx -y @ant-design/cli mcp`、skill `npx skills add ant-design/ant-design-cli`、design.md `https://ant.design/design.md`、llms.txt `https://ant.design/llms.txt` |
| 前端通用 | e2e 配套 Playwright MCP（主）+ Chrome DevTools MCP（辅助调试，对齐 SOP 建议工具表） |
| Go | 官方暂无 llms.txt / MCP（2026-09 核实 404）——用 `go doc` / pkg.go.dev；社区方案现场检索 |
| Spring Boot | 官方暂无 llms.txt（2026-09 核实）——文档按需读 spring.io |
| TypeScript | 官方无 llms.txt（2026-09 核实 404） |

守则：只装/只配用户确认过的单项，不连带全家桶；包名与 URL 必须核实存在才推荐。

### ⑥ 汇报

- 生成物清单与问答结果摘要（哪些定制、哪些保留默认/待确认标记）
- skills 安装与权限白名单建议清单（复制即用）
- 环境缺失待办
- 提醒：CLI 已 git add 生成物，**问答改写的文件需人工补充 git add**

## 产物（项目 CLAUDE.md 改写/新增一览）

- 团队规范段：④ 确认后改写为具体领域 + 确认人与日期
- 新增「环境要求」：⑤ 事实登记
- 新增「辅助开发配置」：已落地的 CLI / MCP / design.md / llms.txt 及用途（skills 待人工装后可补登记）

## 红线

- 本命令不改 agile-templates 模板；不执行 git commit / push；不执行 skills 安装与权限配置（仅建议）
- 引用团队库条目时遵守 SKILL「有效性过滤」硬规则（仅「有效」状态）
