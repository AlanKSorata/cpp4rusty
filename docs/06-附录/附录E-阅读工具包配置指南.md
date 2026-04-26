# 附录 E：阅读工具包配置指南

## E.1 Compiler Explorer 在线分析模板与汇编

适用场景：

- 想快速验证一小段模板代码能否实例化
- 想看 `std::move`、内联、RVO 等生成效果
- 不想先搭本地最小工程

最小使用建议：

- 先贴入一个可独立编译的最小片段
- 编译选项先选 `-std=c++17` 或 `-std=c++20`
- 如果你在验证模板约束或优化行为，尽量去掉无关业务代码

## E.2 VS Code + clangd 智能跳转与提示

最小建议：

- 安装 clangd
- 让项目生成 `compile_commands.json`
- 打开整个仓库，而不是只开单个文件

核心价值：

- 跳转定义
- 查找引用
- 查看类型悬浮提示
- 识别宏展开前后的导航边界

最小命令模板：

```bash
cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

如果项目已经生成了 `build/compile_commands.json`，多数编辑器与 `clangd` 就能基于它建立准确索引。

## E.3 如何使用 `-E`、`-ast-dump` 查看预处理与 AST

### `-E`

用于查看预处理后代码，适合排查：

- 宏展开结果
- 条件编译后的真实代码形态
- include 链造成的代码拼接效果

最小命令：

```bash
clang++ -std=c++17 -E src/foo.cpp
```

如果你只想看前面一段展开结果，可把输出重定向到文件或分页查看。

### `-ast-dump`

用于查看编译器理解后的抽象语法树，适合排查：

- 名字绑定
- 模板实例化轮廓
- 表达式和声明的真实结构

最小命令：

```bash
clang++ -std=c++17 -Xclang -ast-dump -fsyntax-only src/foo.cpp
```

如果项目依赖复杂 include 路径，优先从 `compile_commands.json` 中拿到真实编译参数再执行。

## E.4 调用图生成：Doxygen / cflow 快速设置

适用场景：

- 想理解大型项目从入口到核心模块的调用链
- 想快速把陌生项目切成几个稳定的阅读区域

阅读建议：

- 调用图只用于建立局部路径感，不要试图一次看全仓库
- 优先围绕入口函数、核心类和热点路径生成局部图

## E.5 Sanitizer 最小启用模板

如果你不仅在阅读，还要辅助验证生命周期与并发猜测，可启用 Sanitizer。

### AddressSanitizer / UndefinedBehaviorSanitizer

```bash
cmake -S . -B build-asan \
  -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined -fno-omit-frame-pointer" \
  -DCMAKE_EXE_LINKER_FLAGS="-fsanitize=address,undefined"
```

### ThreadSanitizer

```bash
cmake -S . -B build-tsan \
  -DCMAKE_CXX_FLAGS="-fsanitize=thread -fno-omit-frame-pointer" \
  -DCMAKE_EXE_LINKER_FLAGS="-fsanitize=thread"
```

这些不是发布配置，而是帮助你验证“这里会不会悬垂、越界、竞争”的排查工具。

## E.6 一份足够实用的本地阅读工具包

如果你不想一开始装很多东西，最小组合可以是：

- `cmake`
- `clang++`
- `clangd`
- 一个支持跳转定义和查找引用的编辑器

对阅读任务而言，这套组合的性价比通常已经足够高。

## 工具使用原则

- 工具不能替代语义理解，但能显著降低定位成本。
- 对阅读任务来说，跳转定义和查找引用通常比“一次性全部编译成功”更重要。
- 当你怀疑自己误解了宏、模板或条件编译时，优先借助工具验证。
- 先用最小命令验证猜测，再逐步增加复杂度；不要一上来把整个仓库工具链全部堆满。

---

[← 附录D](./附录D-cpp-版本特性速查表.md) | [返回总目录](../README.md) | [→ 附录F](./附录F-术语对照表.md)
