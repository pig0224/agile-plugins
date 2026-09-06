---
description: PRD 生成。由负责人执行：把产品在外部平台（腾讯文档/飞书等）编写的需求文档结构化为 PRD、验收标准（AC）、功能树与菜单树（写入抽屉三 biz-product-docs）
argument-hint: <需求编号或需求描述，如 STO-001 或一段需求文字>
---

# /agile:prd — PRD 生成

先阅读 skill `sdd-tdd-method`，然后按以下步骤执行。

## 输入解析

- `$ARGUMENTS` 中若含需求编号（形如 STO-xxx / BUG-xxx），以其为任务编号；否则自动分配下一个编号（列出 `process-docs/` **与抽屉三 `requirements/`** 下现有编号目录，取最大值顺延——抽屉三可能已有立项未同步的编号，漏扫会撞号）。
- 其余文字视为需求描述；若为空，向用户询问需求描述后继续。需求描述来源：产品在外部平台（腾讯文档/飞书等）编写的需求文档——用户提供文档要点、链接内容或粘贴原文。

## 执行步骤

1. 读 `.agile/settings.json` 获取 `paths.bizProductDocs`（抽屉三路径）。
2. 调用 **product-manager** subagent（Task 工具委派），传入：
   - 需求编号与需求描述（`$ARGUMENTS` 全文）
   - 抽屉三路径
3. subagent 产出 `PRD.md / AC.md / feature-tree.md / menu-tree.md` 到 `<bizProductDocs>/requirements/<编号>/`。
4. 向用户汇报：产物文件清单、AC 数量、待确认问题。

## 完成后建议

提示用户下一步：`/agile:sync-req <编号>` 把需求产物同步到过程目录。
