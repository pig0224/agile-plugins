---
description: UI 设计与组件库全生命周期：建设（含选型判定与立项）、页面原型、维护升级（OPS 轻量）。可由 AI 主动调用（用户提出 UI/组件库诉求时）
argument-hint: <子命令与参数，如 build 建设组件库 | prototype STO-001 页面原型 | maintain 升级组件>
---

# /agile:ui — UI 设计与组件库全生命周期

先阅读 skill `sdd-tdd-method`，然后按 `$ARGUMENTS` 判断模式（缺省时询问用户选择）：

## 模式一：build（组件库建设）

1. **前置选型判定**（先问用户，避免重复造轮子）：
   - **选现成组件库**（如 ant.design）：**不进建设流程**——登记选型结论（项目 `CLAUDE.md` / `docs/conventions.md`），引导执行 `/agile:init` 配置框架 AI 能力包（CLI / MCP / design.md / llms.txt），本模式结束
   - **自建**：进入建设流程（下一步起）
2. **立项**：组件库从 0 到 1 是产品级动作，走**独立 STO 立项**（从 `/agile:prd <编号> 组件库建设` 起完整流程；规模小经确认可走轻量通道），后续开发按 SOP 生命周期执行
3. 读抽屉三 UI 规范、抽屉二前端工程规范；团队库匹配已确认时读 `biz-tech-docs/frameworks/<前端栈>/`（未确认不读栈领域，按 SKILL 优先级链）
4. 调用 **ui-designer** subagent：从 0 到 1 建设组件库——**设计 token 是第一批产物**（同步写入项目 `docs/ui.md` 的 token 清单）、基础组件 5-8 个起步、目录结构、README、测试（TDD）
5. 完成后按目录状态落库：**若组件库目录尚未初始化为项目**——用 `agile init project <name> --template vue3-vite|react-vite` 落库（继承规范骨架三文件 + 预置 `docs/ui.md`）；**若目录已存在**（第 4 步建设时已建）——跳过 init（该命令对已存在目录必报错），直接在该目录内补齐/调整（规范骨架三文件、`docs/ui.md`），并登记模板注册中心
6. 长期结论（组件 API 约定、主题定制经验）建议 `/agile:knowledge capture` 沉淀至团队库（组件用法 → `team frameworks/<前端栈>/`）

## 模式二：prototype STO-xxx（页面原型）

**定位**：prd 后、frontend 前的**可选动作**（建议执行以对齐页面结构与交互，不设门禁）。

1. 校验 `process-docs/<编号>/requirement.md` 存在（否则先 /agile:prd）。
2. **必读链条**（按 SKILL §6 优先级，受团队库匹配状态约束）：项目 `docs/ui.md`（有则必读——原型的视觉描述用其 token 名，不写裸色值）→ 抽屉三 UI 规范 → `frameworks/<前端栈>/`（仅匹配确认后）。
3. 调用 **ui-designer** subagent：产出 `<bizProductDocs>/prototypes/<编号>/page-*.md`（结构、交互说明、mermaid 流程、规范缺口清单）。
4. 汇报原型文件与规范缺口（缺口反馈产品，页面实现归 `/agile:frontend`）。

## 模式三：maintain（组件维护/升级）

**流程形态**：默认走 **OPS 轻量通道**（纯技术维护项；变更含业务可见行为时提醒升级 STO 轻量）——经 `/agile:sync-req OPS-xxx <变更一句话>` 轻量创建过程目录，worktree / PR 照走，design.md = 三五行变更简述。

1. 调用 **ui-designer** subagent：盘点组件库（读 CHANGELOG 与组件目录），按 `$ARGUMENTS` 中描述的变更需求执行：
   - 升级：改实现 + 更新测试 + CHANGELOG 登记
   - 废弃：标记 deprecated + 迁移指引
2. **改设计 token 属于本模式**：token 变更须同步回写项目 `docs/ui.md`（单一事实源）。
3. 受影响页面：grep 组件库引用，列出受影响仓库与文件，建议批量验证方式。
4. 踩坑与经验建议 `/agile:knowledge capture` 沉淀。

## 通用要求

- 所有产物全中文、遵循抽屉三 UI 规范；组件先测试后实现（TDD）。
- **分工边界**：本命令只管组件库与页面原型——页面实现归 `/agile:frontend`，长期结论沉淀归 `/agile:knowledge capture`。
- 汇报格式：产出/变更清单、测试结果、规范缺口/兼容性影响。
