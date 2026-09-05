---
description: 需求同步。将需求产物从 biz-product-docs 同步到 process-docs/<编号>/，创建标准任务目录（implementation 含 be/fe 角色文件），为后续开发做准备。轻量通道（无 PRD 产物）时以一句话需求轻量创建目录
argument-hint: <需求编号> [一句话需求]；有 PRD 产物走完整同步，无则轻量创建
---

# /agile:sync-req — 需求产物同步

先阅读 skill `sdd-tdd-method`，然后按以下步骤执行。

## 输入

- 需求编号：`$ARGUMENTS` 第一个词（必填；为空时列出 `process-docs/` 下现有编号目录请用户选择）。
- 一句话需求：其余文字（可选）——轻量通道（STO 轻量 / OPS）使用。

## 执行步骤

1. 读 `.agile/workspace.yaml` 获取 `paths.bizProductDocs` 与 `paths.processDocs`。
2. **判断通道**：`<bizProductDocs>/requirements/<编号>/` 存在 PRD 产物 → 完整同步；不存在（或编号未立项）→ 轻量形态。
3. 创建过程目录（幂等，调用 MCP 工具 `agile_task_create`，参数 `{ "taskId": "<编号>" }`；task 能力不暴露为 CLI 命令）。

### 完整同步（PRD 产物存在）

4. `<bizProductDocs>/requirements/<编号>/PRD.md` → 若 `process-docs/<编号>/requirement.md` 仍为模板（含「待填充」/占位），将 PRD 核心内容 + AC 完整并入 `requirement.md`（保留模板中的 AC 章节结构，填充 AC 条目）。
5. 其余产物（AC.md、feature-tree.md、menu-tree.md）**复制**到 `process-docs/<编号>/`（保持文件名）。
6. 源文件保留不删（抽屉三是知识库原件）。
7. 校验：`requirement.md` 中 AC 至少 1 条；否则警告用户回到 `/agile:prd`。

### 轻量形态（无 PRD 产物，STO 轻量 / OPS）

4. `requirement.md`：头部加标记行 `> 本变更走轻量通道（STO 轻量 / OPS）`，正文填一句话需求——STO 轻量需 1–2 条 AC（用户未给则现场收集，AC 红线不豁免）；OPS 填改动说明。
5. `gen-test.md` 就地填一行 `> 本变更走轻量通道，此文档不适用`。
6. 其余文件保持骨架，按 SOP「轻量通道」填法在各自阶段填写。

## 输出

汇报：创建/更新的文件清单（相对路径）、通道形态（完整/轻量）。完整流程提示下一步 `/agile:architect <编号>`；轻量通道提示 design.md 将由 `/agile:architect` 按轻量深度产出（三五行方案简述）。
