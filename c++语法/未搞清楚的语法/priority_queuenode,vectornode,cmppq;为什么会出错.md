`decltype` 是 C++11 引入的一个 **类型推导关键字**，用于在编译时 **自动推导表达式的类型**。它不会实际执行表达式，只是根据表达式的 **静态类型信息** 推导出其类型。

---

## 📌 语法：

```cpp
decltype(expression)
```

- `expression` 是一个表达式（变量、函数调用、lambda 等）
- `decltype(expression)` 的结果是这个表达式的 **类型**

---

## 🔍 示例讲解

### 示例 1：基本类型

```cpp
int x = 42;
decltype(x) y = 100; // y 的类型是 int
```

等价于：
```cpp
int y = 100;
```

---

### 示例 2：引用类型

```cpp
int a = 10;
int& b = a;

decltype(b) c = a; // c 是 int&
```

---

### 示例 3：函数返回值类型

```cpp
int add(int x, int y) { return x + y; }

decltype(add(1, 2)) result = 3; // result 类型是 int
```

---

### 示例 4：Lambda 表达式类型

```cpp
auto cmp = [](int a, int b){ return a < b; };
std::priority_queue<int, std::vector<int>, decltype(cmp)> pq(cmp);
```

- `cmp` 是一个 lambda 对象
- `decltype(cmp)` 推导出这个 lambda 的具体类型（是一个匿名的函数对象类类型）

---

## ✅ 在你的代码中使用 `decltype(cmp)` 的意义：

你写的是：

```cpp
auto cmp = [](auto& first, auto& second) -> bool {
    return first.ch > second.ch;
};
priority_queue<node, vector<node>, decltype(cmp)> pq(cmp);
```

解释如下：

| 写法            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| `decltype(cmp)` | 告诉编译器：我用 `cmp` 这个 lambda 的类型作为优先队列的比较器类型 |
| `pq(cmp)`       | 构造优先队列的时候传入这个具体的比较器对象                   |

这样就既满足了模板参数需要一个 **类型**，也传入了具体的比较逻辑。

---

## 💡 小贴士：`auto` vs `decltype`

| 关键字     | 用途                                 |
| ---------- | ------------------------------------ |
| `auto`     | 自动推导变量的类型（从初始化表达式） |
| `decltype` | 获取表达式的静态类型（不求值）       |

---

## ✅ 总结

| 表达式                    | 含义                             |
| ------------------------- | -------------------------------- |
| `decltype(x)`             | 得到变量 `x` 的类型              |
| `decltype(func())`        | 得到函数返回值的类型             |
| `decltype(auto)`（C++14） | 结合两者的能力，更灵活的类型推导 |

如果你有更多关于 `decltype` 或者泛型编程的问题，欢迎继续问我！