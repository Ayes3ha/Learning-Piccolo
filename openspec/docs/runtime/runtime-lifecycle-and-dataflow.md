# Runtime Lifecycle And Dataflow

Last reviewed: 2026-07-05

## Scope

本文描述 runtime 的启动顺序、帧循环、world/level/object 装载，以及逻辑侧到渲染侧的交换路径。

## Startup Sequence

入口链路由 `engine/source/editor/source/main.cpp` 和 `engine/source/runtime/engine.cpp` 驱动：

```text
main()
  -> PiccoloEngine::startEngine(config)
     -> Reflection::TypeMetaRegister::metaRegister()
     -> RuntimeGlobalContext::startSystems(config)
  -> PiccoloEngine::initialize()
  -> PiccoloEditor::initialize(engine)
```

关键点：

- 反射注册先于系统启动。
- `ConfigManager` 必须最早初始化，因为后续 `AssetManager`、`WindowSystem`、`RenderSystem` 都依赖它提供路径或资源 URL。
- `WindowSystem` 在 `RenderSystem` 之前创建，因为 `RenderSystem::initialize()` 需要 window handle 初始化 `VulkanRHI`。

## Frame Loop

`PiccoloEngine::tickOneFrame()` 当前顺序如下：

```text
logicalTick(delta_time)
  -> WorldManager::tick(delta_time)
  -> InputSystem::tick()
calculateFPS(delta_time)
RenderSystem::swapLogicRenderData()
rendererTick(delta_time)
  -> RenderSystem::tick(delta_time)
WindowSystem::pollEvents()
WindowSystem::setTitle("Piccolo - <fps> FPS")
```

含义：

- world 逻辑总是在输入系统 tick 之前执行。
- 逻辑侧与渲染侧之间只有一次显式交换点：`swapLogicRenderData()`。
- 渲染帧结束后才轮询窗口事件，因此输入回调对下一帧生效。

## World / Level / Object Load Path

默认 world 来自 `ConfigManager::getDefaultWorldUrl()`。首次 `WorldManager::tick()` 会触发实际加载：

```text
ConfigManager
  -> default world url
WorldManager::loadWorld(world_url)
  -> AssetManager::loadAsset(world_url, WorldRes)
  -> loadLevel(WorldRes.m_default_level_url)
Level::load(level_url)
  -> AssetManager::loadAsset(level_url, LevelRes)
  -> create PhysicsScene
  -> createObject(ObjectInstanceRes) for each level object
GObject::load(instance)
  -> use instanced components first
  -> AssetManager::loadAsset(definition_url, ObjectDefinitionRes)
  -> append missing definition components
  -> component->postLoadResource(parent)
```

## Object And Component Rules

- `Level::createObject()` 分配 `GObjectID`，并在 `GObject::load()` 成功后才写入 `m_gobjects`。
- `Level::tick()` 会先 tick 全部 `GObject`，再在非 editor mode 下 tick 当前 active character，最后 tick `PhysicsScene`。
- `Component::postLoadResource()` 是组件拿到父对象引用的标准入口；新组件接入时 SHOULD 在这里建立跨组件引用或缓存。

## Render Swap Consumption

`RenderSystem::processSwapData()` 是逻辑侧数据进入渲染侧的唯一标准入口。当前消费的数据分为：

| Swap Payload | Consumer Behavior |
| --- | --- |
| `m_level_resource_desc` | 上传全局 IBL / color grading 资源 |
| `m_game_object_resource_desc` | 为对象 part 分配实例 ID、装载 mesh/material、更新 `RenderScene` |
| `m_game_object_to_delete` | 从 `RenderScene` 删除对象实体 |
| `m_camera_swap_data` | 更新 `RenderCamera` 的 FOV、view matrix、camera type |
| `m_particle_submit_request` | 创建 emitter 并初始化 `ParticlePass` |
| `m_emitter_tick_request` / `m_emitter_transform_request` | 驱动粒子 emitter tick 与 transform 更新 |

处理顺序是有意义的：全局资源 -> 对象资源 -> 删除 -> 相机 -> 粒子。新增 swap data 类型时 SHOULD 明确它与现有顺序的先后关系。

## Render Tick Skeleton

在 `RenderSystem::tick()` 中，渲染帧目前遵循：

```text
processSwapData()
RHI::prepareContext()
RenderResource::updatePerFrameBuffer()
RenderScene::updateVisibleObjects()
RenderPipeline::preparePassData()
DebugDrawManager::tick()
RenderPipeline::{forwardRender|deferredRender}()
```

如果逻辑侧改动未能体现在画面中，MUST 先检查对应组件是否真的写入了 `RenderSwapContext`，再检查 `processSwapData()` 是否消费并落进 `RenderScene` / `RenderResource`。

## Validation Checklist

- 修改 world 装载协议后，SHOULD 检查 `engine/asset/world/*.json`、`engine/asset/level/*.json`、`engine/asset/objects/*.json` 是否仍符合 serializer 预期。
- 修改组件装载逻辑后，MUST 检查 `postLoadResource()` 是否仍在实例组件与定义组件两条路径都被调用。
- 修改 render swap 路径后，SHOULD 对照 `RenderSystem::processSwapData()` 验证新增字段是否被重置，避免旧数据跨帧残留。
