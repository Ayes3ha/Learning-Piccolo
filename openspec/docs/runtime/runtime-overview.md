# Runtime Overview

Last reviewed: 2026-07-05

## Scope

本文概括 `engine/source/runtime` 的模块布局和关键对象关系，适用于判断修改点和依赖边界。

## Directory Layout

`engine/source/runtime` 当前按四层组织：

| Directory | Responsibility | Representative Paths |
| --- | --- | --- |
| `core` | 基础设施：数学、日志、反射、序列化 | `core/math`, `core/log`, `core/meta` |
| `platform` | 平台封装：文件、路径 | `platform/file_service`, `platform/path` |
| `resource` | 配置与资产读写、资源类型定义 | `resource/config_manager`, `resource/asset_manager`, `resource/res_type` |
| `function` | 世界、组件、输入、渲染、物理、粒子、UI | `function/framework`, `function/render`, `function/input`, `function/physics` |

## Runtime Service Graph

`RuntimeGlobalContext::startSystems()` 当前按以下顺序创建系统：

```text
ConfigManager
FileSystem
LogSystem
AssetManager
PhysicsManager
WorldManager
WindowSystem
InputSystem
ParticleManager
RenderSystem
DebugDrawManager
RenderDebugConfig
```

`shutdownSystems()` 基本按反向依赖关系回收资源，因此新增系统时 MUST 同时审查初始化顺序和清理顺序。

## Core Runtime Objects

| Object | Path | Responsibility |
| --- | --- | --- |
| `PiccoloEngine` | `engine/source/runtime/engine.cpp` | 启动 runtime、计算 delta time、驱动逻辑帧和渲染帧 |
| `WorldManager` | `engine/source/runtime/function/framework/world/world_manager.cpp` | 加载 world/level、维护当前 active level、支持重载和保存 |
| `Level` | `engine/source/runtime/function/framework/level/level.cpp` | 创建 `GObject`、驱动物体 tick、持有物理场景 |
| `GObject` | `engine/source/runtime/function/framework/object/object.cpp` | 聚合组件、从 definition/instance 资源装载组件 |
| `Component` | `engine/source/runtime/function/framework/component/component.h` | 通用组件基类，定义 `postLoadResource()` 和 `tick()` |
| `RenderSystem` | `engine/source/runtime/function/render/render_system.cpp` | 初始化 RHI、RenderScene、RenderResource、RenderPipeline，并消费 swap data |

## Resource and Content Boundary

- `ConfigManager` 提供 root folder、asset folder、默认 world URL、全局渲染/粒子资源 URL。
- `AssetManager::loadAsset()` / `saveAsset()` 通过 `_generated/serializer/all_serializer.h` 做 JSON <-> C++ 对象转换。
- `WorldRes`、`LevelRes`、`ObjectDefinitionRes`、`ObjectInstanceRes` 位于 `engine/source/runtime/resource/res_type/common/`，是 world 内容装载链路的外部契约。

## Runtime Invariants

- world 首次实际加载发生在 `WorldManager::tick()`，而不是 `startSystems()` 期间。
- `GObject::load()` 先装载实例组件，再补齐定义组件；实例组件与定义组件同类型时，定义组件 MUST 被跳过。
- `GObject::tick()` 在 editor mode 下会受 `g_editor_tick_component_types` 过滤；当前默认只注册 `TransformComponent` 和 `MeshComponent`。
- `RenderSystem` 只消费经过 `RenderSwapContext` 交换的数据；逻辑侧 SHOULD 避免直接写渲染场景内部状态。

## Common Change Paths

- 调整系统生命周期：从 `engine/source/runtime/function/global/global_context.cpp` 入手。
- 调整 world/level/object 资源协议：从 `engine/source/runtime/resource/res_type/common/*.h` 与 `AssetManager` 入手。
- 调整组件装载或 editor-mode tick 规则：从 `engine/source/runtime/function/framework/object/object.cpp` 与组件子类入手。
- 调整渲染同步：从 `engine/source/runtime/function/render/render_system.cpp` 以及 swap data 生产者入手。

## Validation Hints

- 修改系统启动顺序后，SHOULD 检查 `PiccoloEngine::startEngine()` 到 `tickOneFrame()` 的依赖是否仍满足。
- 修改资源协议后，MUST 同时检查运行时 include 的 `_generated/serializer/all_serializer.h` 是否仍能覆盖对应类型。
- 修改 editor mode 相关 runtime 行为后，SHOULD 结合 `openspec/docs/editor/editor-workflow.md` 复核选择、gizmo 和对象详情面板流程。
