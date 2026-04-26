# 附录 D：C++ 版本特性速查表

## C++11

高频可见特性：

- `auto`
- `nullptr`
- 范围 `for`
- `std::unique_ptr` / `std::shared_ptr`
- 移动语义、右值引用
- `override`、`final`
- `constexpr`（早期形式）

## C++14

高频可见特性：

- 泛型 lambda
- 更宽松的 `constexpr`
- `std::make_unique`

## C++17

本书基线版本，高频特性：

- `std::string_view`
- `std::optional`
- `std::variant`
- `if constexpr`
- 结构化绑定
- 文件系统库 `std::filesystem`

## C++20

高频特性：

- Concepts
- ranges
- `std::span`
- 三路比较 `<=>`
- 协程

## C++23

高频特性：

- `std::expected`
- 更多 ranges 增强
- 标准库补充改进

## 阅读建议

看到某个类型或语法时，可以快速反推出：

- 项目至少依赖哪个标准版本
- 是否能在旧代码库里理所当然地使用该特性
- 某些“本该存在”的现代写法为什么在老工程里缺席

---

[← 附录C](./附录C-常见编译链接错误信息地图.md) | [返回总目录](../README.md) | [→ 附录E](./附录E-阅读工具包配置指南.md)
