---
doc_id: editor-root
title: Editor Docs
doc_type: index-root
status: active
owner: piccolo-engine
audience: ai-agent
entry_points:
  - openspec/docs/editor/editor-workflow.md
related_docs:
  - openspec/docs/overview/root.md
  - openspec/docs/runtime/root.md
tags:
  - editor
  - imgui
  - gizmo
last_reviewed: 2026-07-05
---

# Editor Docs

## Task Entry
trace-editor-loop -> openspec/docs/editor/editor-workflow.md
modify-editor-ui -> openspec/docs/editor/editor-workflow.md
debug-selection-gizmo -> openspec/docs/editor/editor-workflow.md
trace-editor-input -> openspec/docs/editor/editor-workflow.md

## Scope
本目录覆盖 `engine/source/editor` 的主循环、全局上下文、UI 面板、输入处理、对象选择和 gizmo 操作。这里 SHOULD 关注 editor 对 runtime 的复用方式与数据回写路径。

## Directory Routing
`editor-workflow.md`：描述 editor mode 的启动、窗口布局、输入命令、拾取和 gizmo 变换链路。

## Execution Order
处理 editor 问题时，MUST 先确认 runtime 侧依赖是否稳定，再读 `editor-workflow.md`。涉及对象变换或拾取时，SHOULD 重点阅读其中的 selection/gizmo 部分。

## Maintenance Constraints
如果 `PiccoloEditor::initialize()`、`EditorGlobalContext::initialize()`、`EditorInputManager` 回调注册或 `EditorUI` 窗口布局发生变化，MUST 更新 `editor-workflow.md`。
