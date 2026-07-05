# Editor Workflow

Last reviewed: 2026-07-05

## Scope

本文描述 editor mode 如何挂载到 runtime 上，以及 UI、输入、对象选择和 gizmo 操作如何闭环。

## Initialization Path

editor 的启动路径由以下文件共同决定：

- `engine/source/editor/source/main.cpp`
- `engine/source/editor/source/editor.cpp`
- `engine/source/editor/source/editor_global_context.cpp`
- `engine/source/editor/source/editor_ui.cpp`

初始化顺序如下：

```text
main()
  -> PiccoloEngine::startEngine()
  -> PiccoloEngine::initialize()
  -> PiccoloEditor::initialize(engine)
     -> g_is_editor_mode = true
     -> EditorGlobalContext::initialize()
     -> EditorSceneManager::setEditorCamera(render camera)
     -> EditorSceneManager::uploadAxisResource()
     -> EditorUI::initialize()
```

关键约束：

- editor 复用 runtime 创建好的 `WindowSystem` 和 `RenderSystem`。
- `g_is_editor_mode` 是 editor/runtime 共享的全局开关；依赖该标记的逻辑 MUST 同时考虑 runtime 行为变化。
- `PiccoloEditor` 构造时只注册 `TransformComponent` 和 `MeshComponent` 进入 editor tick 白名单。

## Run Loop

`PiccoloEditor::run()` 当前在 engine tick 前插入 editor 侧逻辑：

```text
while (true)
  -> delta_time = engine.calculateDeltaTime()
  -> EditorSceneManager::tick(delta_time)
  -> EditorInputManager::tick(delta_time)
  -> engine.tickOneFrame(delta_time)
```

这意味着：

- editor 相机和 gizmo 计算发生在 runtime `WorldManager::tick()` 之前。
- editor 仍依赖 runtime 的 `tickOneFrame()` 完成对象渲染、窗口事件轮询和 FPS 标题刷新。

## UI Surface

`EditorUI::showEditorUI()` 目前组织五类窗口：

| Window | Responsibility |
| --- | --- |
| `Editor menu` | DockSpace、菜单、调试开关、窗口显隐 |
| `World Objects` | 当前 level 的 `GObject` 列表与选择 |
| `Game Engine` | 渲染目标、gizmo 切换、相机状态展示 |
| `File Content` | 资产树与 object definition 点击创建 |
| `Components Details` | 基于反射的详情面板 |

细节约束：

- 详情面板通过 `m_editor_ui_creator` 按类型名注册绘制函数；新增可编辑类型时 SHOULD 先在这里补 UI creator。
- `File Content` 点击 `object` 类型节点会构造新的 `ObjectInstanceRes`，然后调用 `Level::createObject()` 创建对象。
- `Reload Current Level` 菜单项会同时调用 `WorldManager::reloadCurrentLevel()` 和 `RenderSystem::clearForLevelReloading()`。

## Input Model

`EditorInputManager::registerInput()` 把窗口回调全部注册到同一个 manager：

- reset
- cursor position / enter
- scroll
- mouse button
- window close
- key

键位行为当前由 bitmask `m_editor_command` 管理：

- `W/A/S/D/Q/E`：editor 相机平移
- `T/R/C`：切换平移 / 旋转 / 缩放 gizmo
- `Delete`：删除当前选中对象
- 鼠标右键拖动：旋转 editor 相机
- 鼠标左键拖动：沿当前 gizmo 轴变换对象

## Selection And Gizmo Path

对象选择和 gizmo 变换由 `EditorInputManager` 与 `EditorSceneManager` 配合完成：

```text
left click in game window
  -> RenderSystem::getGuidOfPickedMesh(picked_uv)
  -> RenderSystem::getGObjectIDByMeshID(mesh_id)
  -> EditorSceneManager::onGObjectSelected(gobject_id)
  -> drawSelectedEntityAxis()

left drag on axis
  -> EditorSceneManager::moveEntity(...)
  -> TransformComponent::setPosition/Rotation/Scale(...)
```

关键约束：

- `EditorSceneManager::tick()` 会把当前选中对象的 `TransformComponent` 标记为 dirty，确保后续 runtime/render 路径感知到变换变化。
- gizmo 资源通过 `uploadAxisResource()` 预先占用保留的 instance ID 与 mesh asset ID，避免与普通对象冲突。
- 当删除对象时，editor 除了从当前 level 移除对象，还会向 `RenderSwapContext` 追加 delete request，确保渲染侧同步删除。

## Validation Checklist

- 修改 editor 输入逻辑后，SHOULD 同时检查鼠标坐标到 engine window UV 的换算是否仍与 `Game Engine` 窗口布局一致。
- 修改详情面板反射绘制后，MUST 检查 `Transform`、`Vector3`、`Quaternion` 和 `std::string` 的既有 UI creator 是否仍能工作。
- 修改对象选择或删除逻辑后，SHOULD 联动检查 runtime 文档中的 render swap 删除链路，避免场景对象和渲染实体失步。
