---
doc_id: toolchain-root
title: Toolchain Docs
doc_type: index-root
status: active
owner: piccolo-engine
audience: ai-agent
entry_points:
  - openspec/docs/toolchain/reflection-and-build-pipeline.md
related_docs:
  - openspec/docs/overview/root.md
  - openspec/docs/runtime/root.md
tags:
  - toolchain
  - reflection
  - shader
last_reviewed: 2026-07-05
---

# Toolchain Docs

## Task Entry
trace-reflection-codegen -> openspec/docs/toolchain/reflection-and-build-pipeline.md
debug-precompile -> openspec/docs/toolchain/reflection-and-build-pipeline.md
trace-shader-compile -> openspec/docs/toolchain/reflection-and-build-pipeline.md
map-build-target-dependencies -> openspec/docs/toolchain/reflection-and-build-pipeline.md

## Scope
本目录覆盖构建期工具链，包括反射/序列化代码生成、预编译入口和 shader 编译目标。这里 MUST 描述生成链路与运行时消费关系。

## Directory Routing
`reflection-and-build-pipeline.md`：描述 `PiccoloParser`、`PiccoloPreCompile`、`PiccoloShaderCompile` 的职责和依赖顺序。

## Execution Order
遇到 `_generated` 缺失、反射类型未生效或 shader 产物异常时，MUST 先读 `reflection-and-build-pipeline.md`，再回到对应 runtime 模块定位消费点。

## Maintenance Constraints
如果 `engine/CMakeLists.txt`、`engine/source/precompile/precompile.cmake`、`engine/source/meta_parser` 或 `engine/shader/CMakeLists.txt` 的调用关系变化，MUST 更新 `reflection-and-build-pipeline.md`。
