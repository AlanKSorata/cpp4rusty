# 附录 B：标准库容器与 Rust 对应表

## 序列容器

| Rust | C++ | 说明 |
| --- | --- | --- |
| `Vec<T>` | `std::vector<T>` | 最常见动态数组 |
| `VecDeque<T>` | `std::deque<T>` | 双端高效插入删除 |
| `LinkedList<T>` | `std::list<T>` | 双向链表，工程中较少优先选择 |
| `String` | `std::string` | 拥有型字符串 |
| `&str` | `std::string_view` | 只读观察视图，需警惕悬挂 |

## 关联容器

| Rust | C++ | 说明 |
| --- | --- | --- |
| `HashMap<K, V>` | `std::unordered_map<K, V>` | 哈希表 |
| `HashSet<T>` | `std::unordered_set<T>` | 哈希集合 |
| `BTreeMap<K, V>` | `std::map<K, V>` | 有序映射，通常基于树 |
| `BTreeSet<T>` | `std::set<T>` | 有序集合 |

## 适配器

| Rust 近似概念 | C++ | 说明 |
| --- | --- | --- |
| 栈接口 | `std::stack<T>` | 容器适配器 |
| 队列接口 | `std::queue<T>` | FIFO 适配器 |
| 优先队列 | `std::priority_queue<T>` | 堆语义适配器 |

## 智能指针与辅助类型

| Rust | C++ | 说明 |
| --- | --- | --- |
| `Box<T>` | `std::unique_ptr<T>` | 独占所有权 |
| `Rc<T>` / `Arc<T>` | `std::shared_ptr<T>` | 共享所有权 |
| `Weak<T>` | `std::weak_ptr<T>` | 弱引用 |
| `Option<T>` | `std::optional<T>` | 可选值 |
| `Result<T, E>` | `std::expected<T, E>` | C++23 |
| `enum` | `std::variant<...>` | 需手动访问分支 |

## 阅读提醒

- 名字相近不代表边界完全相同。
- C++ 容器常伴随迭代器失效问题。
- 观察型类型如 `string_view`、裸指针、迭代器都需要手动验证生命周期。

---

[← 附录A](./附录A-核心关键字与运算符速查.md) | [返回总目录](../README.md) | [→ 附录C](./附录C-常见编译链接错误信息地图.md)
