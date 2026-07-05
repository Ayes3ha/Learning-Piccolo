# Reflection And Build Pipeline

Last reviewed: 2026-07-05

## Scope

本文描述 Piccolo 的构建期工具链，包括反射/序列化代码生成、预编译触发链路，以及 shader 编译目标。

## Build Target Dependency

`engine/CMakeLists.txt` 当前把工具链依赖串成如下关系：

```text
PiccoloParser
  -> produces parser executable in engine/bin

PiccoloPreCompile
  -> invokes PiccoloParser with precompile.json + parser_header.h

PiccoloRuntime
  -> depends on PiccoloPreCompile
  -> includes engine/source/_generated and shader/generated outputs

PiccoloShaderCompile
  -> compiles GLSL and emits generated shader artifacts
```

这意味着 runtime 编译前 MUST 先有 parser 产物和 shader 产物。

## Reflection / Serializer Generation

反射声明入口在 `engine/source/runtime/core/meta/reflection/reflection.h`，运行时与构建期通过同一套宏协作：

- 源码中的 `REFLECTION_TYPE`、`CLASS`、`REFLECTION_BODY` 提供元数据标记。
- `PiccoloParser` 在 `engine/source/meta_parser` 中通过 libclang 读取这些标记。
- `parser/main.cpp` 接收六个关键参数：项目配置文件、聚合 include 文件、源码根目录、系统 include、模块名、错误输出开关。
- `precompile.cmake` 把这些参数拼成对 parser 的调用，并输出到 `engine/source/_generated/reflection`、`engine/source/_generated/serializer`。

## Precompile Invocation Path

`engine/source/precompile/precompile.cmake` 当前执行逻辑可概括为：

```text
configure_file(precompile.json.in -> engine/bin/precompile.json)
select parser executable by host platform
set parser input header to ${CMAKE_BINARY_DIR}/parser_header.h
custom target PiccoloPreCompile
  -> run PiccoloParser precompile.json parser_header.h engine/source sys_include Piccolo 0
```

重点约束：

- parser 的输入根目录固定指向 `engine/source`，因此新增反射类型时 SHOULD 保证头文件位于该树下并可被聚合头包含。
- runtime 和 asset 序列化依赖 `_generated/serializer/all_serializer.h`；若生成链路失败，`AssetManager` 的 `loadAsset()` / `saveAsset()` 会失去类型覆盖。

## Shader Compilation Path

`engine/shader/CMakeLists.txt` 当前会递归收集 `engine/shader/glsl/` 下的 `.vert/.frag/.comp/.geom/...` 文件，并调用 `cmake/ShaderCompile.cmake` 中的 `compile_shader()`：

```text
collect GLSL files
  -> compile with glslangValidator
  -> emit generated artifacts under engine/shader/generated
  -> runtime includes generated/cpp during PiccoloRuntime build
```

因此 shader 改动 MAY 同时影响：

- `engine/shader/generated/spv` 下的二进制产物
- runtime 编译时包含的 generated C++ 头

## Runtime Consumption Points

- `AssetManager` 通过 `_generated/serializer/all_serializer.h` 读写 world/level/object/global 等 JSON 资源。
- 多个 runtime 文件直接 include `_generated/serializer/all_serializer.h`，例如：
  - `engine/source/runtime/function/framework/world/world_manager.cpp`
  - `engine/source/runtime/function/framework/object/object.cpp`
  - `engine/source/runtime/resource/asset_manager/asset_manager.h`
- `PiccoloRuntime` 在 `engine/source/runtime/CMakeLists.txt` 中显式依赖 `PiccoloShaderCompile`，并包含 `engine/shader/generated/cpp`。

## Validation Checklist

- 新增反射类型后，MUST 检查对应头是否真的进入 parser 扫描范围，而不只是被编译器看到。
- 修改 parser 或 precompile 参数后，SHOULD 验证 `engine/bin/precompile.json`、parser 可执行文件路径和系统 include 选择是否仍匹配宿主平台。
- 修改 shader 编译链路后，MUST 检查 `glslangValidator` 路径、`engine/shader/generated` 输出目录以及 runtime include 路径是否仍一致。
