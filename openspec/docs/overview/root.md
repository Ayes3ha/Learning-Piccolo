---
doc_id: overview-root
title: Overview Docs
doc_type: index-root
status: active
owner: piccolo-engine
audience: ai-agent
entry_points:
  - openspec/docs/overview/architecture-overview.md
related_docs:
  - openspec/docs/runtime/root.md
  - openspec/docs/editor/root.md
  - openspec/docs/toolchain/root.md
tags:
  - architecture
  - overview
last_reviewed: 2026-07-05
---

# Overview Docs

## Task Entry
trace-engine-layout -> openspec/docs/overview/architecture-overview.md
choose-module-entry -> openspec/docs/overview/architecture-overview.md
map-build-targets -> openspec/docs/overview/architecture-overview.md

## Scope
本目录覆盖仓库级代码地图、核心构建目标以及 runtime/editor/toolchain 的边界关系。这里 SHOULD 保持高层视角，不展开某个子系统的实现细节。

## Directory Routing
`architecture-overview.md`：汇总顶层目标、源码入口、运行时与工具链之间的调用边界。

## Execution Order
当任务尚未落到具体子系统时，MUST 先读 `architecture-overview.md`，再跳转到 `runtime/root.md`、`editor/root.md` 或 `toolchain/root.md`。

## Maintenance Constraints
新增或删除顶层构建目标时，MUST 更新 `architecture-overview.md`。若新增一级文档目录，MUST 同步更新 `openspec/docs/README.md`。
