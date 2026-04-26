# 第12章 贯穿项目：微型 JSON 解析库完整阅读

## 本章导读

用一个完整案例把前面分散的语言点重新串起来。

## 适合谁读

想通过项目级样例验证自己是否已经具备实际阅读能力的读者。

这一章把前面的概念汇总到一个小而完整的案例里。相比零散知识点，贯穿式阅读更接近真实工作：你不是只看 `variant`，也不是只看 `string_view`，而是要在同一个项目中同时理解构建、数据结构、错误流和资源管理。

本章不只讲“一个 JSON 库通常会长什么样”，而是给出一份你可以直接跟着演练的最小阅读模型。

## 12.1 项目组织与 CMake 构建结构

先假设仓库结构如下：

```text
mini_json/
  CMakeLists.txt
  include/mini_json/value.h
  include/mini_json/parser.h
  src/value.cpp
  src/lexer.cpp
  src/parser.cpp
  tests/parser_test.cpp
```

一个微型 JSON 解析库通常至少包含：

- 公开头文件：对外暴露 `Value`、解析函数、格式化函数
- 若干源文件：词法分析、语法分析、打印
- 一个测试 target

最小 CMake 入口通常像这样：

```cmake
add_library(mini_json
    src/value.cpp
    src/lexer.cpp
    src/parser.cpp
)
target_include_directories(mini_json PUBLIC include)

add_executable(parser_test tests/parser_test.cpp)
target_link_libraries(parser_test PRIVATE mini_json)
```

阅读时，请先确定：

- 库 target 叫什么
- 哪些文件属于对外接口
- 是否存在示例程序或测试作为入口
- 哪些源文件真正被编进目标

### 第一轮阅读动作

1. 先看顶层 `CMakeLists.txt`，确认库 target 和测试 target。
2. 再看 `include/mini_json/parser.h`，确认公开 API。
3. 然后看 `tests/parser_test.cpp`，因为测试往往是最快的调用链入口。

## 12.2 数据结构：递归 `variant` 的 JSON 表示

JSON 值通常可表示为：

- null
- bool
- number
- string
- array
- object

一个最小但足够真实的定义可能长这样：

```cpp
namespace mini_json {

struct Value;
using Array = std::vector<Value>;
using Object = std::map<std::string, Value>;

struct Value {
    using Storage = std::variant<
        std::monostate,
        bool,
        double,
        std::string,
        Array,
        Object
    >;

    Storage data;
};

} // namespace mini_json
```

### 阅读重点

- `variant` 如何表示多种值分支
- 数组和对象是否通过 `vector` / `map` 组织
- 递归结构如何避免无限大小定义问题

### Rust 开发者容易先误判的点

- 这很像 `enum Value`，但它没有命名分支与穷尽性保护。
- 新增 `variant` 分支后，不会自动强迫所有访问点一起更新。
- 容器类型只说明存储形式，不说明生命周期和异常行为。

## 12.3 词法分析：`string_view` 零拷贝与错误返回

词法分析器通常会持有输入视图，并在其上推进位置索引。

例如：

```cpp
class Lexer {
public:
    explicit Lexer(std::string_view input) : input_(input) {}
    Token next();

private:
    std::string_view input_;
    std::size_t pos_ = 0;
};
```

### 这里的阅读关键点

- `string_view` 是否只观察调用者传入缓冲区
- 错误是通过异常、返回值还是状态码传播
- token 是否引用原始输入片段而不是复制字符串

### 生命周期标注示例

当你看到 `Lexer(std::string_view input)` 时，应立刻补一句：

> `Lexer` 不拥有输入，它只借看；原始字符串必须比 `Lexer` 活得更久。

如果调用点是：

```cpp
auto value = parse(std::string("{\"x\":1}"));
```

你要继续确认：

- `parse()` 是否在函数内部同步消费完整个输入
- 是否把某个 `string_view` 保存进了返回值或长期对象

这类检查在 Rust 里常由借用系统兜底，在 C++ 阅读里需要你手工完成。

## 12.4 语法分析：递归下降中 `unique_ptr` 与移动语义

如果实现选择递归下降，阅读时你应关注：

- 解析函数之间如何递归调用
- 中间对象是否按值返回
- 是否使用移动语义减少中间复制
- 在失败路径上资源是否自然释放

一个简化的解析入口可能是：

```cpp
std::expected<Value, ParseError> parse(std::string_view input) {
    Lexer lexer(input);
    Parser parser(lexer);
    return parser.parse_value();
}
```

### 一次调用链追踪

假设测试里有：

```cpp
auto result = parse("{\"name\":\"alice\",\"age\":3}");
```

你可以按这个顺序跟：

1. `parse()` 创建 `Lexer`，建立对原始输入的观察。
2. `Parser` 从 `Lexer` 取 token，决定当前走 object / array / scalar 哪个分支。
3. `parse_value()` 递归构造 `Value`。
4. 失败时沿 `std::expected` 返回 `ParseError`。
5. 成功时按值返回 `Value`，中间临时对象依赖 RAII 自动清理。

### 阅读时要主动标的 3 个点

- 这个函数返回的是拥有值、观察值，还是错误包装。
- 这个对象是否可能保存输入缓冲区上的视图。
- 失败路径上已经构造的中间状态是否会自然销毁。

## 12.5 美丽打印：使用 `std::visit` 实现格式化输出

格式化输出通常是 `variant` 阅读的好练习，因为它要求你对所有 JSON 分支逐一处理。

```cpp
std::string format(const Value& v) {
    return std::visit(Overloaded{
        [](std::monostate) { return "null"s; },
        [](bool b) { return b ? "true"s : "false"s; },
        [](double d) { return std::to_string(d); },
        [](const std::string& s) { return "\"" + s + "\""; },
        [](const Array& a) { return format_array(a); },
        [](const Object& o) { return format_object(o); },
    }, v.data);
}
```

### 阅读时可以检查

- 分支处理是否完整
- visitor 是否清晰表达不同类型逻辑
- 容器嵌套打印时递归是否容易跟踪

### 常见漏看点

- 新增 `variant` 分支后，这个 visitor 是否同步更新。
- 辅助函数是否产生了额外拷贝。
- 复杂 visitor 是否把真正逻辑藏进过多模板辅助层。

## 12.6 向 Rust 映射：同一设计在 Rust 中的等价实现

把整个库映射回 Rust 时，可以得到大致对应关系：

- `variant` ↔ `enum`
- `string_view` ↔ `&str`
- `optional` / `expected` ↔ `Option` / `Result`
- `unique_ptr` ↔ `Box`
- RAII ↔ `Drop`

但映射的意义不在于把 C++ 伪装成 Rust，而是借助 Rust 心智更快识别哪些边界在 C++ 里缺失了静态保护。

### 强制做一次双语对照

读完整个小库后，最好自己回答：

- 如果这是 Rust，我会把哪个类型设计成 `enum`？
- 如果这是 Rust，哪些 `string_view` 位置会变成带生命周期的借用？
- 如果这是 Rust，哪些错误路径会被强制写进 `Result` 链？

## 12.7 扩展任务：添加注释解析或新的 JSON 值类型

作为练习，你可以尝试思考：

- 如果支持 JSON with comments，需要改哪些层？
- 如果增加新值类型，哪些 visitor 和打印逻辑会受影响？
- 如果返回错误位置，错误类型设计应如何变化？

## 12.8 一页式阅读模板：把这章方法带去读别的小库

遇到任意一个小型 C++ 库时，先按下面模板记笔记：

### 结构

- 对外入口头文件是哪些
- 哪个 target 真正产出库
- 测试或示例入口在哪里

### 数据

- 核心类型有哪些
- 哪些类型拥有资源
- 哪些类型只是观察视图

### 控制流

- 从一个公开 API 进入后，会经过哪些主要函数
- 错误沿哪条通道返回
- 哪些地方用异常，哪些地方用返回值

### 风险

- 有没有 `string_view` / 裸指针 / 引用成员
- 有没有 `variant` / `visit` 的分支漏改风险
- 有没有对象移动、容器失效、递归深度等风险

## 延伸阅读

- 本章会大量复用 `variant` 的阅读能力，可先回看：[第7章 代数数据类型](../../../03-第二部分-边界区/第7章-代数数据类型/README.md)
- 本章也会用到 `string_view` 与资源移动的直觉，可分别参考：[第4章 字符串、容器与迭代器](../../../02-第一部分-安全区/第4章-字符串-容器与迭代器/README.md) 与 [第9章 资源管理与移动语义](../../../04-第三部分-激战区/第9章-资源管理与移动语义/README.md)

## 本章检查清单

- 能从构建入口定位库接口与实现分层
- 能识别递归 `variant` 数据模型
- 能跟踪 `string_view` 的生命周期边界
- 能沿解析调用链追踪错误传播方式
- 能把案例中的现代 C++ 设计映射回熟悉的 Rust 概念
- 能用“结构-数据-控制流-风险”四栏快速完成小库阅读笔记

---

[← 第四部分](../README.md) | [返回总目录](../README.md) | [→ 第13章](../第13章-常见-cpp-工程模式/README.md)
