> 一套面向人类与 AI Agent 的双轨知识库体系。基于 Obsidian 管理，受启于 Karpathy's LLM Wiki。

## 设计目的

常见的知识库有两个问题：
1. **人看的库被机器写坏**：Agent 无节制地生成机器文件，导致人在手机上打开 Obsidian 满屏是 `SKILL-A-001`。
2. **机器吃的库不结构**：每次执行任务都全量加载，上下文爆炸，重复造轮子。

本体系的解决方案是 **物理隔离**：
- **Penn_PKM**：人的知识库。轻量、灵活、联想式。
- **Hermes_AKM**：Agent 的知识库。严格、结构化、按需检索。

两者同时在 Obsidian 中打开，但服务于不同的使用者。

---

## 核心概念

| 概念 | 解释 |
|------|------|
| **PKM** (Person Knowledge Management) | （你）的个人知识库。放在 iCloud，所有设备随时访问。 |
| **AKM** (Agent Knowledge Management) | Agent 集团的公用知识库。放在本地，Git 管理，机器优先。 |
| **Raw 层** | 不可变的原材料。Agent 只读不改，保证溯源可追。 |
| **Compiled 层** | 从 Raw 加工后的技能、知识、概念页面。Agent 实际调用的内容。 |
| **地图分层** | 全局入口 (极小) → 项目地图 (专属) → 具体文件。绝不一次性加载所有地图。 |
| **技能上浮** | 复用率 ≥50% 的项目级技能，晋升至通用技能库。原位置留 Stub 重定向。 |
| **Lint** | 定期体检：孤儿页面、断链、过期内容、标签污染、矛盾标记。 |

---

## 整体架构

```
~/KnowledgeBase/
├── Penn_PKM/                          # 个人知识库 (iCloud)
│   ├── 00_Meta/
│   │   ├── PKM_Guide.md                 # 个人库导航
│   │   └── Tags.md                      # 标签体系
│   ├── Inbox/                           # 低摩擦输入（日记、灵感、剪藏）
│   ├── Sources/                         # 确认保留的原文
│   ├── Notes/                           # 原子化笔记
│   ├── MOC/                             # 内容地图（人工维护）
│   └── Projects/                        # 个人项目
│
└── Hermes_AKM/                        # Agent 知识库 (本地 + Git)
    ├── 00_Meta/
    │   ├── SCHEMA.md                    # 库宗法：命名、YAML、阈值、政策
    │   ├── AKM_Guide.md                 # 全局路由入口（极小）
    │   ├── Index.md                     # 全库 Compiled 层总目录
    │   ├── ID_Registry.md               # 技能总览极简表（构建 Agent 时使用）
    │   ├── Log.md                       # 操作日志（Append-only）
    │   └── Templates/                   # 模板文件
    │       ├── Skill_Template.md
    │       ├── Knowledge_Template.md
    │       ├── Project_MOC_Template.md
    │       ├── Stub_Redirect_Template.md
    │       └── Raw_Template.md
    ├── 10_Raw/
    │   ├── articles/                    # 网页剪藏
    │   ├── papers/                      # 论文/PDF
    │   ├── transcripts/                 # 对话记录
    │   └── assets/                      # 图片、图表
    └── 20_Compiled/
        ├── 10_Common/
        │   ├── Common_MOC.md            # 通用技能地图
        │   ├── Skills/                  # 复用率 ≥50% 的技能
        │   └── Knowledge/               # 跨项目通用知识
        └── 20_Project_Spaces/
            └── Project_X/               # 按项目分区
                ├── X_MOC.md             # 项目专属地图
                ├── 01_Skills/           # 项目内技能 (复用率 < 50%)
                ├── 02_Knowledge/        # 项目内知识
                └── 03_Context/          # 项目配置、需求、临时草稿
```

---

## 设计原则

### 1. 人机分离
- PKM 人读为主，不强制 YAML，标签自由。
- AKM 机器主导，每个文件必须带规范 Frontmatter。

### 2. Raw 不可变
- `10_Raw/` 里的原材料一旦入库不再修改。
- 错误或更新在 Compiled 层处理，通过 `sha256` 检测原来源漂移。

### 3. 技能与知识分离
- **Skill**：HOW。操作流程、调用协议、脚本。
- **Knowledge**：WHAT。事实、数据、概念、背景。
- 同一信息单元只有一个主地址，不得双栖。

### 4. 分层地图加载
- 日常执行：AKM_Guide (全局) → Project_MOC (项目) → 具体文件。
- 构建 Agent 时：才读取 ID_Registry 总览表。
- **无关项目的地图不加载**，控制上下文污染。

### 5. 技能上浮阈值
- 复用率 **≥ 50%** 的项目级技能，晋升至 `10_Common/Skills/` 。
- 原位置留 Stub 重定向文件，保证链接不断裂。

### 6. 链接优先于层级
- 每个 Compiled 页面至少包含 2 个出站 `[[wikilink]]`。
- 孤儿页面（0 入站链接）是严重问题，Lint 必须捕获。

---

## 核心规范

### YAML Frontmatter (必填)

所有 `20_Compiled/` 下的文件必须带头：

```yaml
---
id: SKILL-A-001              # 全局唯一 ID：TYPE-SCOPE-NNN
title: "页面标题"           # 人读标题
created: YYYY-MM-DD          # 创建日期
updated: YYYY-MM-DD          # 最后更新
type: skill | knowledge | concept | comparison
scope: common | Project_Alpha # 归属范围
tags: [from taxonomy]        # 必须来自 SCHEMA taxonomy
sources: [raw/articles/xxx]  # 原材料来源
confidence: high | medium | low
contested: false
contradictions: []
---
```

### ID 体系

| 组成 | 含义 | 示例 |
|------|------|------|
| TYPE | SKILL / KNOW / CONCEPT / COMP | `SKILL` 技能，`KNOW` 知识 |
| SCOPE | C (Common) / A/B... (项目缩写) | `C` 通用，`A` 属于 Project Alpha |
| NNN | 三位序号 | `001`, `002` |

示例：`SKILL-C-003` （第3号通用技能），`KNOW-A-012` （Alpha 项目第12号知识）。

### Page Threshold (建页阈值)

- 实体/概念在 **2+ 来源** 出现，或单来源中处于核心 → 建页面。
- 仅一笔带过、注脚 → 不建页面。
- 超过 **200 行** → 拆分为子页面。

---

## 工作流

### 1. Ingest (入库)

将外部信息吸收进入 AKM 的流程：

1. **抓取原文**：URL → 网页剪藏 / PDF → 提取 / 文本 → 存入 `10_Raw/`。
2. **生成 sha256**：计算内容哈希，用于后续 Drift 检测。
3. **初筛确认**：给人一个摘要清单，确认哪些值得加工。
4. **分切处理**：人判断是混合文件还是纯技能/知识，Agent 执行切分。
5. **Cross-reference**：新页面至少链出 2 个 `[[wikilink]]`。
6. **更新导航**：登记 Index，追加 Log。

### 2. Query (查询)

Agent 执行任务时查资料：

1. **Orientation**：读 `AKM_Guide.md` → 读目标 `Project_MOC.md`。
2. **定位文件**：根据 MOC 索引找到具体 Skills / Knowledge。
3. **按需加载**：只读需要的文件。如遇 `confidence: low` 或 `contested: true`，在答案中标注不确定性。

### 3. Lint (体检)

定期或按需运行，检查项目：

- 孤儿页面（无入站链接）
- 断链（`[[xxx]]` 指向不存在）
- Index 完整性（文件系统 vs 索引对照）
- Frontmatter 合法性
- 过期内容（`updated` 超过 90 天）
- 矛盾标记 (`contested: true`)
- 页面膨胀（>200 行）
- 标签污染（非 taxonomy 内的标签）
- 日志轮转（满 500 条自动分年份归档）

### 4. Agent Build (构建新 Agent)

1. 读取 `ID_Registry.md` 技能总览表。
2. 对照新 Agent 工作流，匹配可复用技能。
3. 识别缺口：缺失的技能手写或网络检索补充。
4. 新 Agent 只加载它需要的 Skills + 对应 Project_MOC，不加载全库。

---

## 双库联动

| 场景 | 流向 | 动作 |
|------|------|------|
| 你发现好文章 | PKM → AKM | 确认后 Agent 抓 Raw 、加工、入库 |
| 任务产生 SOP | AKM → PKM | 技能存 AKM，同步一份人话版到 PKM |
| 你记笔记时产生研究点 | PKM → AKM | 标注 `#needs-research`，Agent 补充研究 |
| Agent 发现知识过时 | AKM → PKM | Lint 报告整理成清单推给你复审 |

---

## 快速开始

### 部署到你的环境

1. **复制骨架**
   ```bash
   git clone https://github.com/<your-repo>/hermes-dual-vault.git
   cd hermes-dual-vault
   ```

2. **配置 PKM**
   - 将 `Penn_PKM/` 移动到你的 iCloud Obsidian 目录：
     `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/`
   - 在 Obsidian 中打开 Vault。

3. **配置 AKM**
   - 将 `Hermes_AKM/` 移到你想要的本地路径，例如 `~/Hermes_AKM`。
   - 初始化 Git：`git init && git add . && git commit -m "init"`
   - 在 Obsidian 中打开第二个 Vault（文件夹模式），选择 `~/Hermes_AKM`。

4. **配置 Agent**
   - 设置环境变量 `AKM_PATH=~/Hermes_AKM`。
   - 确保 Agent 工具集包含 `file` 和 `terminal`。
   - 使用 `llm-wiki` 或自定义 Ingest 流程开始填充。

### 创建第一个项目

在 `Hermes_AKM/20_Compiled/20_Project_Spaces/` 下：

```bash
mkdir Project_Alpha
cd Project_Alpha
# 复制模板
# 01_Skills/, 02_Knowledge/, 03_Context/
```

然后让 Agent 执行 Orientation：读 `SCHEMA.md` + `AKM_Guide.md` + 新建的 `Alpha_MOC.md`。

---

## 与 Karpathy's LLM Wiki 的关系

本体系直接受启于 Andrej Karpathy 的 LLM Wiki 模式，借鉴了以下机制：
- **三层架构**：Raw → Compiled → Schema
- **Orientation Ritual**：每次会话先读规范再动手
- **Append-only Log** 与 **Lint 体检**
- **Provenance 溯源**标记 `^[raw/xxx.md]`
- **Page Threshold** 控制内容膨胀

差异在于：
- 引入了 **双库隔离** （PKM + AKM）以区分人机使用场景。
- 引入了 **技能上浮机制** 和 **ID Registry** 以服务多 Agent 协作。
- 专为 Hermes / OpenClaw 等工具集优化了目录结构和命名约定。

---

## 许可证

MIT License

---

---
*本项目由佩恩（Payne）与小马（Hermes Agent）共同设计。*
