# 第7章 代数数据类型：`std::variant` 与访问者模式

## 本章导读

帮助你把 `variant` 代码读成有结构的分支模型，而不是一堆模板语法。

## 适合谁读

熟悉 Rust `enum`，但不熟悉 `std::visit` 与 visitor 模式的读者。

对 Rust 开发者来说，`enum` 是极其自然的表达工具。但在 C++ 里，代数数据类型并没有这么直给。现代 C++ 借助 `std::variant` 提供了一种接近方案，不过它在穷尽性、访问方式和错误暴露上都与 Rust `enum` 有显著差异。

## 7.1 `enum` ↔ `std::variant`：结构定义

```cpp
using Value = std::variant<int, double, std::string>;
```

这表示 `Value` 在任一时刻持有其中一种类型。

### 阅读时的第一层理解

- 这是一个“多选一”的值容器
- 当前活跃分支由运行时状态决定
- 类型集合在编译期固定

### 和 Rust `enum` 的重要差别

Rust `enum` 能天然把标签和携带数据组织成命名分支；`variant` 更像“若干备选类型的联合体封装”。结构表达通常没 Rust 那么直接。

## 7.2 `match` ↔ `std::visit`：穷尽性检查的缺失与补救

访问 `variant` 最常见的现代方式是 `std::visit`。

```cpp
std::visit([](const auto& x) {
    use(x);
}, value);
```

### 阅读时要特别留意

- 这里在对当前活跃类型做分发
- 访问逻辑可能被隐藏在 lambda 重载集中
- 穷尽性不像 Rust `match` 那样天然显式可读

Rust 中，漏掉分支通常编译不过；C++ 中，很多情况下你需要自己确认处理是否完整。

## 7.3 `std::get` / `std::get_if`：按类型或索引取值

```cpp
auto p = std::get_if<std::string>(&value);
```

### 阅读区别

- `std::get<T>`：若当前分支不是 `T`，可能抛异常或编译失败（取决于用法）
- `std::get_if<T>`：返回指针，失败时为空

这更像一种“动态检查当前分支”的读法，而不是 Rust 那样由模式匹配显式拆开控制流。

## 7.4 `std::monostate`：空状态的处理

如果希望 `variant` 里带一个“空分支”，常见写法是：

```cpp
using Token = std::variant<std::monostate, int, std::string>;
```

这可以近似理解为显式塞进一个“什么都没有”的分支，但它不像 `Option<T>` 那样天然表达缺失语义，更多是构造层面的占位技巧。

## 7.5 递归 variant：构建一个 JSON 值类型的完整示例

现代 C++ 中，JSON 值常见一种思路：

- null
- bool
- number
- string
- array
- object

这些可以通过 `variant` 配合容器和间接持有来表达。

### 阅读意义

这类代码通常会同时用到：

- `variant`
- 智能指针或容器
- `visit`
- 递归数据结构

如果你能读懂这类例子，就已经建立了较强的现代 C++ 数据结构阅读能力。

## 7.6 习语解码：重载 lambda 实现“伪模式匹配”

你经常会看到这样的辅助写法：

```cpp
auto visitor = Overloaded{
    [](int x) { ... },
    [](const std::string& s) { ... }
};
std::visit(visitor, value);
```

这是一种模拟模式匹配体验的常见技巧。

### 阅读提示

- `Overloaded` 通常是个小模板工具
- 它把多个 lambda 合并成一个可调用对象
- 真正的分支逻辑被拆散在多个 lambda 中

## 7.7 陷阱：`std::visit` 不保证穷尽，错配类型将抛异常或编译错误

Rust 开发者在这里最容易过度乐观。`variant` 确实提供类型安全的替代分支容器，但它不像 Rust `enum` 那样强力约束分支处理形状。

### 阅读时要警惕

- 某些访问逻辑是否遗漏了某种活跃类型
- `get` 是否建立在未经检查的假设上
- 新增分支后，旧 visitor 是否都被同步更新

## 7.8 练习：扩展递归 variant，增加一种新类型并完成 visitor

假设已有一个 JSON 值 `variant`，请尝试加入 `binary` 或 `date` 新分支，并思考：

1. 哪些 visitor 必须修改？
2. 哪些 `get_if` 分支要新增？
3. 与 Rust `enum` 新增一个分支相比，这里的漏改风险在哪里？

## 7.9 常见误判

- `variant` 不等于 Rust `enum` 的完整等价物。
- `visit` 不等于自带穷尽性保证的 `match`。
- `get<T>` 不等于“编译器已经证明当前一定是 T”。
- `monostate` 不等于语义清晰的 `None`。
- 分支类型集合固定，不等于所有调用点都会自动同步更新。

## 7.10 阅读时先问什么

遇到 `variant` 代码时，先问：

1. 当前有哪些可能分支？
2. 这里是通过 `visit`、`get` 还是 `get_if` 访问？
3. 分支处理是否完整，新增类型后哪些位置会漏改？
4. 这段代码是在表达建模意图，还是在为历史类型系统打补丁？

## 延伸阅读

- `variant` 在完整案例中的落地形式可继续看：[第12章 贯穿项目：微型 JSON 解析库完整阅读](../../../05-第四部分-实战阅读/第12章-微型-json-解析库/README.md)
- 如果你想先降低模板和 visitor 带来的语法压力，可先补读：[第8章 模板基础](../../../04-第三部分-激战区/第8章-模板基础/README.md)

## 本章检查清单

- 知道 `variant` 是现代 C++ 中最接近代数数据类型的工具
- 知道 `visit` 是主要分发入口
- 知道 `get_if` 比盲目 `get` 更安全
- 知道穷尽性通常需要你自己验证
- 看到重载 lambda visitor 时能读出“这在模拟模式匹配”
- 新增分支时会自然想到 visitor 和访问逻辑的同步风险

---

[← 第6章](../第6章-错误处理/README.md) | [返回总目录](../README.md) | [→ 第三部分](../../04-第三部分-激战区/README.md)
