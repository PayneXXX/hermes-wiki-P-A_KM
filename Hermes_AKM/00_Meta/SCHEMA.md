---
title: "AKM 库宗法"
created: 2026-04-26
updated: 2026-04-26
type: meta
tags: [meta, schema]
---

# AKM 库宗法

## 领域

Hermes Agent 集团的公用知识库。包含通用技能、项目技能、知识背景、操作模板与接口定义。

## 命名规范

- 文件名：kebab-case，小写，用连字符连接，无空格。
  - 正确：`alpha-data-scraper.md`
  - 错误：`Alpha Data Scraper.md`
- ID 格式：`<TYPE>-<SCOPE>-<NNN>`，如 `SKILL-A-001`、`KNOW-C-005`。
  - TYPE: SKILL / KNOW / CONCEPT / COMP
  - SCOPE: A(属于某项目) / C(Common)

## Frontmatter 规范

每个 Compiled 层文件必须带以下 YAML：

```yaml
---
id: SKILL-A-001              # 全局唯一 ID
title: "xxx"                 # 显示标题
created: YYYY-MM-DD          # 创建日期
updated: YYYY-MM-DD          # 最后更新
type: skill | knowledge | concept | comparison
scope: common | Project_Alpha # 归属范围
tags: [from taxonomy]        # 必须来自本 Schema 的 taxonomy
sources: [raw/articles/xxx]  # 原材料来源
confidence: high | medium | low
contested: false             # 是否存在争议
contradictions: []           # 与哪些页面矛盾
---
```

## 标签体系 (Taxonomy)

添加新标签前，必须先在此处注册，再使用。

- **技能类**: skill, automation, scraping, deployment, testing, git, workflow
- **知识类**: concept, architecture, protocol, benchmark, dataset
- **工具类**: tool, api, mcp, sdk, cli
- **元类**: meta, schema, template, stub
- **项目类**: project-alpha, project-beta（以项目名为准）

## 页面阈值 (Page Thresholds)

- 当一个实体/概念在 **2+ 来源** 中出现，或在单一来源中处于核心地位 → 建页面。
- 仅仅一笔带过、注脚、小细节 → 不建页面，在现有页面中提及。
- 页面超过 **200 行** → 拆分为子页面，交叉链接。
- 内容完全过时 → 移至 `_archive/`，从 Index 移除。

## 技能上浮政策

- 复用率 **≥50%** 的项目→浮至 `10_Common/Skills/`。
- 浮上时在原项目位置留 **Stub 重定向文件**，保留链接不断裂。
- 原标识从 `SKILL-A-xxx` 改为 `SKILL-C-xxx`，更新 ID_Registry。

## 更新政策 (Update Policy)

新信息与旧内容矛盾时：
1. 检查日期 — 新来源一般优先于旧来源。
2. 如果确实存在矛盾，在页面中注明两种立场并标注日期和来源。
3. Frontmatter 中标记 `contested: true`、`contradictions: [页面ID]`。
4. Lint 时将这些页面擕出交佩恩复审。

## 原材料层 (Raw) 规范

- Raw 层不可变。正文不修改，错误在 Compiled 层更正。
- Raw 文件必须带以下 Frontmatter：
  ```yaml
  ---
  source_url: https://...
  ingested: YYYY-MM-DD
  sha256: <内容哈希>
  ---
  ```
- 重复 Ingest 同一 URL 时，比较 sha256：一致则跳过，不一致则标记 Drift。

## 链接规范

- 每个 Compiled 页面至少包含 **2 个出站 `[[wikilink]]`**。
- 孤儿页面（0 入站链接）是严重问题，Lint 必须捕获。
- 多源综合的段落，应在段落末尾注明 `^[raw/articles/xxx.md]` 溯源。
