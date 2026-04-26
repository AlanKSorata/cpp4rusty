# 附录 F：术语对照表

| Rust 术语 | C++ 近似术语 | 简明解释 |
| --- | --- | --- |
| 所有权 | ownership / resource ownership | 谁负责对象最终释放 |
| 借用 | reference / observer | 只访问，不一定拥有 |
| 生命周期 | lifetime | 对象或引用保持有效的时间区间 |
| trait | interface / abstract base / concept | 抽象行为约束 |
| trait object | virtual dispatch / base pointer | 运行时多态 |
| `Drop` | destructor / RAII | 离开作用域时自动清理 |
| crate | library / target / module group | 需结合构建系统理解 |
| module | header / source / namespace | C++ 没有完全对应的单一结构 |
| pattern matching | `switch` / `std::visit` | 常缺穷尽性检查 |
| unsafe | raw pointer / UB-prone region | 在 C++ 中很多能力默认就在这一区域 |

## 使用建议

把这些对照表当作阅读起点，而不是最终结论。真正进入代码时，仍然要回到：

- 生命周期是谁保证的
- 所有权是否明确
- 类型边界是否靠语言保障，还是只靠约定

---

[← 附录E](./附录E-阅读工具包配置指南.md) | [返回总目录](../README.md) | [→ 总目录](../README.md)
