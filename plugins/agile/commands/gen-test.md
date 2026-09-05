---
description: 测试案例生成（Stage 1）。基于 AC 与设计产出测试案例文档 gen-test.md，先于实现
argument-hint: <需求编号，如 STO-001>
---

# /agile:gen-test — 测试案例生成（Stage 1）

先阅读 skill `sdd-tdd-method`，然后按以下步骤执行。

## 前置校验

1. 需求编号：`$ARGUMENTS`（为空时列出 `process-docs/` 下现有编号目录让用户选择）。
2. 校验 `process-docs/<编号>/requirement.md` 已填充（含 AC）——测试案例来自 AC。缺失则停止，提示先 /agile:prd + /agile:sync-req。
3. 校验 `process-docs/<编号>/design.md` 已填充（案例需要接口/数据模型信息）。缺失则警告并提示 /agile:architect（可选择仅基于 AC 生成，需用户确认）。

## 执行步骤

1. 调用 **test-engineer** subagent（Task 工具委派），传入：
   - 需求编号、requirement.md 与 design.md 路径、涉及的仓库列表（来自 design.md 涉及模块表）
2. subagent 产出 `process-docs/<编号>/gen-test.md`（案例清单 TC 表格、数据准备、自动化映射）。**案例清单分「后端用例」「前端用例」两节**——开发期各自只在自己节内补充/勾选，避免前后端并行时的合并冲突。**e2e 用例（浏览器端到端自动化）归入「前端用例」节**：类型列标 `e2e`，自动化映射指向前端项目的 e2e 脚本（如 `e2e/*.spec.ts`），只覆盖关键路径而非全量。
3. 校验产出：每条 AC 至少 1 正常 + 1 边界/异常案例；不满足则让 subagent 补齐。

## 输出

汇报：案例数量（按类型/优先级统计）、自动化覆盖率（单测/e2e）、文件路径；提示下一步 /agile:backend 或 /agile:frontend。
