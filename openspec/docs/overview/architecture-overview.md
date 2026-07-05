# Piccolo Architecture Overview

Last reviewed: 2026-07-05

## Scope

本文提供仓库级入口路由，用于快速判断任务应该落到 runtime、editor 还是 toolchain。结论基于以下代码和构建文件：

- `CMakeLists.txt`
- `engine/CMakeLists.txt`
- `engine/source/runtime/engine.cpp`
- `engine/source/editor/source/main.cpp`
- `engine/source/runtime/function/global/global_context.cpp`

## Top-Level Targets

`CMakeLists.txt` 把仓库根目标委托给 `engine/`，`engine/CMakeLists.txt` 再挂接实际构建单元：

| Target | Path | Responsibility |
| --- | --- | --- |
| `PiccoloRuntime` | `engine/source/runtime` | runtime 核心静态库 |
| `PiccoloEditor` | `engine/source/editor` | 编辑器可执行程序 |
| `PiccoloParser` | `engine/source/meta_parser` | 反射/序列化代码生成器 |
| `PiccoloPreCompile` | `engine/source/precompile` | 触发 parser 生成 `_generated` 代码 |
| `PiccoloShaderCompile` | `engine/shader` | GLSL -> SPIR-V/C++ 头生成 |

## Repository Routing

```text
repo root
  -> engine/source/runtime      runtime systems, resources, gameplay-side logic
  -> engine/source/editor       editor loop, UI, picking, gizmo, asset browsing
  -> engine/source/meta_parser  libclang-based codegen
  -> engine/source/precompile   build-time parser invocation
  -> engine/shader              shader compile pipeline and generated outputs
```

## Runtime / Editor / Toolchain Boundary

```text
main()
  -> PiccoloEngine::startEngine()
     -> Reflection::TypeMetaRegister::metaRegister()
     -> RuntimeGlobalContext::startSystems()
  -> PiccoloEngine::initialize()
  -> PiccoloEditor::initialize()
  -> PiccoloEditor::run()
```

- runtime 负责系统初始化、世界加载、逻辑帧、渲染帧和资源读写。
- editor 复用同一个 runtime，只在 engine tick 前追加 editor scene/input/UI 行为。
- toolchain 在构建期生成 `_generated/reflection`、`_generated/serializer` 和 shader 产物，runtime 再在编译期/运行期消费这些结果。

## Source Layout

| Area | Main Paths | Notes |
| --- | --- | --- |
| runtime | `engine/source/runtime/core`, `platform`, `resource`, `function` | 引擎底座、资源系统和功能系统 |
| editor | `engine/source/editor/include`, `engine/source/editor/source` | 编辑器主循环、ImGui UI、拾取与 gizmo |
| toolchain | `engine/source/meta_parser`, `engine/source/precompile`, `engine/shader` | 代码生成与 shader 编译 |
| content | `engine/asset`, `engine/configs` | world/level/object/global 配置与资源 |

## Task Routing

- 如果任务涉及 `RuntimeGlobalContext`、`WorldManager`、`RenderSystem` 或资源反序列化，跳到 `openspec/docs/runtime/root.md`。
- 如果任务涉及 ImGui 面板、对象选择、gizmo、编辑器输入，跳到 `openspec/docs/editor/root.md`。
- 如果任务涉及 `CLASS/REFLECTION_BODY`、`PiccoloParser`、`precompile.cmake` 或 shader 生成，跳到 `openspec/docs/toolchain/root.md`。

## Key Constraints

- `PiccoloEditor` 不维护独立引擎实例；editor 改动 SHOULD 视为 runtime 外挂层改动。
- runtime 与 toolchain 通过 `_generated/*` 目录耦合；修改反射声明时 MUST 同时考虑构建期生成链路。
- `engine/asset` 与 `engine/configs` 共同决定默认 world、全局渲染资源和编辑器资源路径；内容侧改动 SHOULD 结合 runtime 文档一起追踪。
