# 第1章 编译与项目结构：从 Cargo 到 CMake

## 本章导读

建立 C++ 项目的第一阅读入口。

## 适合谁读

已经熟悉 Cargo 与 Rust 模块系统，但刚开始接触 CMake、头文件和链接模型的读者。

Rust 开发者阅读 C++ 项目时，第一道门槛通常不是语法，而是项目结构。你在 Rust 世界里习惯了 `Cargo.toml`、`src/lib.rs`、`src/main.rs`、crate 与模块树；到了 C++，你首先会遇到的是 `CMakeLists.txt`、头文件、源文件、目标、链接关系以及看上去不那么统一的目录布局。

本章的目标不是教你完整掌握 CMake，而是帮助你建立一个足够稳定的阅读入口：**看到一个 C++ 仓库时，知道先看哪里、如何判断代码是如何被编译与链接起来的。**

## 1.1 `Cargo.toml` ↔ `CMakeLists.txt`：依赖与目标

Rust 项目中，`Cargo.toml` 同时承担包元数据、依赖声明、构建配置入口等角色。C++ 项目里，这些职责往往分散，但如果项目使用 CMake，最核心入口通常是顶层 `CMakeLists.txt`。

### 你在 Rust 中熟悉的东西

- 包名、版本、依赖列表
- 二进制目标和库目标
- feature、profile、workspace

### 在 CMake 中对应关注什么

- `project(...)`：项目名、语言、版本
- `add_executable(...)`：定义可执行文件目标
- `add_library(...)`：定义静态库或动态库目标
- `target_link_libraries(...)`：声明链接依赖
- `target_include_directories(...)`：声明头文件搜索路径
- `find_package(...)`：寻找外部依赖

### 阅读时的关键问题

读 `CMakeLists.txt` 时，优先回答以下问题：

1. 这个仓库产出的是可执行程序、库，还是两者都有？
2. 源文件是如何被分配到不同 target 的？
3. 哪些第三方库参与构建？
4. 哪些 include 路径会影响头文件解析？
5. 有没有平台分支、编译选项或测试目标？

### 最小示例

```cmake
cmake_minimum_required(VERSION 3.20)
project(demo LANGUAGES CXX)

add_library(core src/core.cpp)
target_include_directories(core PUBLIC include)

add_executable(app src/main.cpp)
target_link_libraries(app PRIVATE core)
```

如果把它翻译成 Rust 阅读直觉，可以近似理解为：

- 有一个叫 `core` 的库目标
- 有一个叫 `app` 的可执行目标
- `app` 依赖 `core`
- `include/` 里的头文件属于 `core` 的公开接口

## 1.2 模块系统：`mod` 与 `use` ↔ 头文件/命名空间/`using`

Rust 的模块树是语言级结构；C++ 的组织方式更松散，通常由以下几层共同构成：

- 文件层：`.h` / `.hpp` / `.cpp`
- 命名空间层：`namespace foo { ... }`
- 构建层：哪些文件被编入哪个 target

### 核心差异

Rust 中，模块声明通常直接决定名字空间与可见性结构。C++ 中，文件路径、命名空间与构建目标之间没有绝对绑定关系。

所以阅读时不要默认：

- 头文件路径就等于逻辑模块树
- 命名空间一定和目录同名
- 包含了头文件就一定链接了实现

### `using` 要谨慎看

`using std::string;` 或 `using namespace std;` 只是名字引入，不是模块导入。它不会像 Rust 的 `use` 那样暗示较清晰的命名关系，反而可能增加查找真实类型来源的成本。

## 1.3 预处理器初识：`#include` 守卫、`#pragma once`

C++ 的 `#include` 本质上更接近文本展开，而不是 Rust 模块导入。这是理解头文件系统的关键。

### 常见写法

```cpp
#pragma once

class Foo {
public:
    void run();
};
```

或者传统 include guard：

```cpp
#ifndef FOO_H
#define FOO_H

class Foo {};

#endif
```

### 阅读要点

- `#include` 不是“引用某个编译单元”，而是把文件内容插入当前上下文。
- 头文件通常放声明，源文件通常放定义，但这只是常见习惯，不是铁律。
- 模板、内联函数、头文件库经常把实现直接放在头文件中。

## 1.4 条件编译：`#[cfg]` 与 `#ifdef` / `#if`

Rust 的 `#[cfg(...)]` 有较结构化的形式；C++ 里的条件编译更多依赖宏。

```cpp
#ifdef _WIN32
void init_windows();
#else
void init_posix();
#endif
```

### 阅读时优先看什么

- 平台分支：Windows / Linux / macOS
- 调试分支：`NDEBUG`、日志开关
- 编译器分支：`__clang__`、`__GNUC__`
- 特性检测：某个标准库或语言特性是否存在

条件编译会直接改变“这份源码在当前平台上到底长什么样”。因此遇到奇怪 API 或缺失定义时，先看宏条件是否屏蔽了相关代码。

## 1.5 实战：搭建最小 C++ 阅读环境（编译器、CMake、LSP）

如果你主要目标是阅读，不需要一次性安装一整套重型工具链，但至少应该具备：

- 一个支持 C++17 的编译器，如 clang++ 或 g++
- CMake，用于生成编译数据库
- clangd 或等效 LSP，用于跳转定义、查引用、悬浮信息
- 支持查看 include 层级和调用关系的编辑器

### 推荐最小工作流

1. 用 CMake 配置项目。
2. 生成 `compile_commands.json`。
3. 用 clangd 打开仓库。
4. 从入口 target 或顶层头文件开始导航。

对阅读而言，LSP 的价值常常高于真正编译成功，因为它能让你更快建立结构感。

## 1.6 习语解码：为什么头文件里总能看到 `class Foo;`

这叫**前向声明**。

```cpp
class Foo;

class Bar {
public:
    Foo* foo_;
};
```

### 它在解决什么问题

如果当前头文件只需要知道“有一个 `Foo` 类型存在”，但不需要知道其完整定义，就可以只写前向声明，避免额外 `#include`。

### 阅读意义

看到前向声明时，你应意识到：

- 这里有意降低头文件耦合
- 当前文件可能只持有指针或引用
- 真正的定义通常在别处，需要跳转继续查找

## 1.7 常见编译/链接错误速查（入门版）

### 编译错误

- `unknown type name` / `does not name a type`
  - 常见原因：缺少头文件、命名空间不对、前向声明不足
- `no matching function`
  - 常见原因：重载不匹配、`const` 限定不同、模板推导失败
- `use of incomplete type`
  - 常见原因：只有前向声明，没有完整定义，却试图按值使用

### 链接错误

- `undefined reference`
- `unresolved external symbol`

常见原因：

- 声明有了，定义没编进 target
- 函数签名不一致
- 模板定义不在可见位置
- 库链接顺序或链接声明有问题

## 1.8 练习：解读一个双文件项目的编译流程

假设目录如下：

```text
include/
  math.h
src/
  math.cpp
  main.cpp
CMakeLists.txt
```

练习时请尝试回答：

1. 哪个文件暴露接口，哪个文件提供实现？
2. `main.cpp` 如何获得 `math.h` 中的声明？
3. `math.cpp` 不被加入 target 会发生什么？
4. 如果头文件里声明了函数，但源文件签名不同，错误会出现在编译期还是链接期？

## 本章检查清单

- 能从 `CMakeLists.txt` 看出主要 target 与依赖关系
- 知道头文件、源文件、命名空间、target 是不同层次的问题
- 理解 `#include` 是文本级包含
- 看到前向声明时能意识到这是解耦信号
- 遇到链接错误时会优先怀疑“声明和定义没有真正接上”

---

[← 第一部分](../README.md) | [返回总目录](../README.md) | [→ 第2章](../第2章-类型系统与基础语法/README.md)
