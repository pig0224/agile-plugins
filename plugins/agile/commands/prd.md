---
description: PRD 生成。由负责人执行：把产品在外部平台（腾讯文档/飞书等）编写的需求文档结构化为 PRD、验收标准（AC）、功能树与菜单树（写入抽屉三 biz-product-docs）
argument-hint: "<需求编号或需求描述，如 STO-001 或一段需求文字> [--prompt <提示词稿路径>]"
---

# /agile:prd — PRD 生成

先阅读 skill `sdd-tdd-method`，然后按以下步骤执行。

## 输入解析

- `$ARGUMENTS` 中若含需求编号（形如 STO-xxx / BUG-xxx），以其为任务编号；否则自动分配下一个编号（列出 `process-docs/` **与抽屉三 `requirements/`** 下现有编号目录，取最大值顺延——抽屉三可能已有立项未同步的编号，漏扫会撞号）。
- `$ARGUMENTS` 含 `--prompt <路径>` 时以该文件为需求描述源（自由命名提示词稿的显式指定方式）；文件不存在即停止并报告路径有误。
- 需求描述按以下**三层**确定（提示词工作流为**可选**，缺任何一层直接落下一层，不阻塞、不强制）：
  1. **读稿编译**：`<bizProductDocs>/prompts/<编号>.md` 存在 → 以提示词稿为需求描述，`$ARGUMENTS` 其余文字作为叠加补充一并传入（目录约定见 `prompts/README.md`）。
  2. **澄清落稿**：稿不存在且 `$ARGUMENTS` 只有编号（或描述明显不足）→ 先与用户对话澄清（按 `prompts/README.md` 骨架维度**有缺才问**：用户/场景、核心规则、边界与异常、明确不做、参考资料），有共识后落稿 `prompts/<编号>.md`（目录或 README 不存在时按 README 约定创建）并向用户展示；用户确认后继续编译，用户表示「先只落稿」则汇报后停止。
  3. **直接编译**：`$ARGUMENTS` 带完整需求描述 → 按描述直接编译。需求描述来源：产品在外部平台（腾讯文档/飞书等）编写的需求文档——用户提供文档要点、链接内容或粘贴原文。

## 执行步骤

1. 读 `.agile/settings.json` 获取 `paths.bizProductDocs`（抽屉三路径）。
2. **续作感知**：`<bizProductDocs>/requirements/<编号>/` 已存在（续作/重跑）时，按 skill「人工修订感知流程」先检测既有 PRD/AC 的人工修改（git diff + 重读对照）——可采纳的并入本次产出依据，人工改过的内容不得被重新生成覆盖；存疑先返回人工确认。
3. 调用 **product-manager** subagent（Task 工具委派），传入：
   - 需求编号与需求描述（来源为提示词稿时传稿全文并告知来源路径，`$ARGUMENTS` 叠加文字一并传入）
   - 抽屉三路径
4. subagent 产出 `PRD.md / AC.md / feature-tree.md / menu-tree.md` 到 `<bizProductDocs>/requirements/<编号>/`（已存在的文件不覆盖，续作感知结论一并传入）；来源为提示词稿时，PRD.md 头部登记 `来源: prompts/<编号>.md`。
5. 向用户汇报：产物文件清单、AC 数量、待确认问题。

## 完成后建议

提示用户下一步：`/agile:sync-req <编号>` 把需求产物同步到过程目录。若本次为直接编译（无提示词稿），可顺带提示：让 AI 把需求描述收敛为 `prompts/<编号>.md`，后续变更改稿重跑（可选，不强制）。
