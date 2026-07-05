---
name: spec-writer
description: 维护并建立面向 AI Agent 阅读的项目文档库。凡是用户提到"补文档/写规范/整理模块文档/更新 docs 索引/把代码沉淀成文档"，尤其涉及 openspec/docs、模块入口、子模块说明、README 索引维护时，都应优先使用本技能，即使用户没有明确说"用 spec-writer"。
---

# spec-writer

将代码模块知识沉淀为可索引、可复用、可供 AI Agent 快速读取的项目文档。

## 目标

- 文档读者是 **AI Agent**，而非终端用户或新人培训对象。
- 把"代码事实 + 业务约束 + 操作方式"整理成结构化文档。
- 文档按模块归档到 `openspec/docs/`。
- 保证 `openspec/docs/README.md` 与各目录 `root.md` 索引同步。

## 核心概念

### 目录根文档（root.md）

每个一级子目录必须维护一个 `root.md`，是 AI Agent 进入该目录的唯一入口文档：

```
openspec/docs/
├── README.md              # 全局入口，只索引到各 root.md
├── ui/
│   └── root.md            # UI 目录入口
├── buff/
│   └── root.md           # Buff 目录入口
├── dr/
│   └── root.md           # DR 工具链目录入口
├── guide/
│   └── root.md           # 引导系统目录入口
├── modules/
│   └── root.md           # 模块说明目录入口
├── battle/
│   └── root.md           # 战斗机制目录入口
└── docs-style-v2.md      # 文档风格规范基线
```

### 文档类型（doc_type）

- `index-root`：目录根文档（如 `ui/root.md`）
- `system-spec`：系统规范（如 `ui-event-system.md`）
- `mechanism`：机制说明（如 `battle-level-logic.md`）
- `playbook`：操作流程/配方（如 `dr-spec.md`）
- `module-overview`：模块总览（如 `modules/game-logic.md`）

## 触发场景

出现以下任一意图时触发：

- 用户要求"给某模块补文档/写规范/沉淀规范"。
- 用户要求"整理/新建/维护 docs 索引"。
- 用户要求"把 scattered 信息汇总到 docs"。
- 用户要求"新增 docs 子目录并建立索引"。
- 用户提到"AI 阅读友好/知识库化/模块维度文档化"。

## 输入约定

- `module_name`: 模块名称（如 `UI`、`Buff`、`DR`）。
- `module_entry`: 1~N 个模块入口文件/目录。
- `related_files`: 相关代码、配置、协议、表格、脚本等。
- `doc_scope`: 新建 / 补充 / 重构（可选）。

## 文档库机制（必须遵守）

- 根目录固定为 `openspec/docs/`。
- 文档按"模块"划分子目录。
- `openspec/docs/README.md` **只索引**各目录 `root.md`，不维护全量子文档清单。
- 每个一级目录 **必须**存在且维护 `root.md`。
- 只有 `root.md` 需要统一 front matter；专题文档不强制。
- 每次新增 / 重命名 / 删除文档后，必须同步更新对应目录的 `root.md`，再更新 `openspec/docs/README.md`。

## root.md 规范（强制）

### root.md front matter（必须字段）

```yaml
---
doc_id: <unique-kebab-id>
title: <title>
doc_type: index-root
status: active | deprecated | archived | draft
owner: <team-or-module>
audience: ai-agent
entry_points:
  - <doc-or-code-path>
related_docs:
  - <doc-path>
tags:
  - <tag>
last_reviewed: YYYY-MM-DD
---
```

**禁止在 root.md front matter 中出现的字段**：`ai_primary`、`language`、`applies_to`、`source_of_truth`。

### root.md 正文结构

```markdown
# <Title>

## Task Entry
<task-key> -> <doc-path>

## Scope
<覆盖范围与边界>

## Directory Routing
<本目录文档地图，一句话描述>

## Execution Order
<按任务类型的推荐执行顺序>

## Maintenance Constraints
<新增/重命名/删除文档时的维护要求>
```

### Task Entry 写法规范

- 每个入口格式：`<task-key> -> <doc-path>`
- `task-key` 使用 kebab-case，如 `implement-ui-window`、`debug-ui-event`
- 不要出现面向培训的叙述（禁止词：`新同学`、`入门`、`学习路径`、`建议先读`）

### 禁用词（强制）

文档正文中 **禁止出现**：
- 新同学 / 新人 / 入门 / 学习路径 / 建议先读
- 面向人类 onboarding 的描述性语言

替换为任务导向表达：
- `Task Entry` / `Execution Order` / `前置依赖` / `验证步骤`

## README.md 规范

- 只索引到各目录 `root.md`，不展开子文档清单。
- 必须包含阅读协议说明：README -> root.md -> 专题文档的顺序。

## 文档正文风格（agent-first）

- 以"做什么 / 改哪里 / 怎么验"为叙述导向。
- 约束使用 `MUST / SHOULD / MAY` 语义。
- 代码引用使用相对仓库路径。
- 不复制大段源码；用"片段 + 路径"替代。

## 执行流程

### 1) 确认任务类型

- 新建目录 / 模块文档：先确认是否需要新建 `root.md`
- 补充 / 重构文档：先读现有 `root.md` 确认路由，再落笔
- 更新索引：先改 `root.md`，再改 `README.md`

### 2) 代码事实抽取

- 从 `module_entry` 出发，识别：
  - 模块职责与边界
  - 核心类 / 核心流程
  - 上下游依赖
  - 外部契约（配置、协议、事件、存档等）
  - 关键约束与易错点
- 结论必须可追溯到代码文件，不写"猜测性架构"。

### 3) 设计文档拆分

优先按"读者任务"拆分，而非按文件堆砌：

- `*-overview.md`：模块总览与边界
- `*-workflow.md`：关键流程与时序
- `*-contracts.md`：输入输出、协议、配置契约
- `*-recipes.md`：常见变更路径与操作配方
- `*-faq.md`：常见问题与排障

### 4) 生成 / 更新文档内容

每篇文档建议包含：

- 适用范围
- 关键结构（类 / 接口 / 配置）
- 关键流程（建议用 ASCII 图）
- 约束与不变量
- 常见变更方式（做什么、改哪里、如何验证）
- 文件引用（可点击路径）

### 5) 更新索引（必须）

- 更新对应目录的 `root.md`（路由表 + Task Entry）。
- 仅在确实新增一级目录时，才更新 `openspec/docs/README.md`。

## 输出格式

```text
doc_plan:
  - <将创建 / 更新的文档清单>
doc_changes:
  - <实际改动，含路径>
index_changes:
  - <root.md / README.md 变更项>
coverage:
  - <本次覆盖的模块维度>
gaps:
  - <缺失信息或建议补充项>
```

## 质量标准

- 可机读：front matter 字段齐全，Task Entry 可被正则匹配。
- 可检索：Task Entry 覆盖常见任务类型。
- 可执行：约束明确，验证步骤可操作。
- 可维护：root.md 与 README.md 同步，无悬空索引。

## 边界与禁忌

- 不把实现细节写成主观推断。
- 不复制大段无结构代码到文档中。
- 不创建"只有标题没有结论"的空文档。
- 不在 `root.md` 以外的文档中强制要求 front matter。
- 禁止在文档中出现面向 human onboarding 的词（见禁用词）。
- 不更新索引却遗漏实际文档（或反之）。

## 示例触发语

- "帮我把 GuideSys 模块整理到 docs，入口是 `GuideSys.cs`。"
- "这个 Buff 子系统太散了，按模块维度沉淀文档并更新 root.md。"
- "根据我给的几个文件，把 UI 生命周期机制写成 AI 可读文档。"
- "新建一个 `battle/` 目录，需要配套 root.md。"
