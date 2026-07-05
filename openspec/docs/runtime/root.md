---
doc_id: runtime-root
title: Runtime Docs
doc_type: index-root
status: active
owner: piccolo-engine
audience: ai-agent
entry_points:
  - openspec/docs/runtime/runtime-overview.md
  - openspec/docs/runtime/runtime-lifecycle-and-dataflow.md
related_docs:
  - openspec/docs/overview/root.md
  - openspec/docs/editor/root.md
  - openspec/docs/toolchain/root.md
tags:
  - runtime
  - resource
  - render
last_reviewed: 2026-07-05
---

# Runtime Docs

## Task Entry
survey-runtime-modules -> openspec/docs/runtime/runtime-overview.md
trace-runtime-startup -> openspec/docs/runtime/runtime-lifecycle-and-dataflow.md
trace-world-loading -> openspec/docs/runtime/runtime-lifecycle-and-dataflow.md
modify-render-swap -> openspec/docs/runtime/runtime-lifecycle-and-dataflow.md

## Scope
本目录覆盖 `engine/source/runtime` 的分层、全局系统装配、资源到对象的装载链路，以及逻辑侧到渲染侧的数据交换。这里 MUST 以代码事实为准，不替代实现细节本身。

## Directory Routing
`runtime-overview.md`：描述 runtime 模块边界、关键类和不变量。  
`runtime-lifecycle-and-dataflow.md`：描述启动顺序、帧循环、world/level/object 装载和渲染交换链路。

## Execution Order
当任务范围较宽时，SHOULD 先读 `runtime-overview.md`。当任务已聚焦到启动、世界装载或渲染同步时，MUST 继续读 `runtime-lifecycle-and-dataflow.md`。

## Maintenance Constraints
如果 `RuntimeGlobalContext::startSystems()` / `shutdownSystems()` 的顺序变更，MUST 更新本目录两个文档。若新增新的资源装载入口或新的 swap data 类型，MUST 更新 `runtime-lifecycle-and-dataflow.md`。
