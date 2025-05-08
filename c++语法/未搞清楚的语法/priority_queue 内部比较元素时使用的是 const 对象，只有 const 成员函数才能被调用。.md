这是一个非常好的问题！我们来详细解释一下：

你写的这段代码：

```cpp
bool operator < (const State &rth) const {
    return dis > rth.dis;
}
```

其中的 `const` 是**成员函数的常量性修饰符（const qualifier）**，它非常重要。下面我们会从多个角度解释为什么这个 `const` 是必要的。

---

## ✅ 一、什么是成员函数的 `const` 修饰？

在 C++ 中，当你在一个类的成员函数后面加上 `const` 关键字时，表示这个函数是一个 **常量成员函数（const member function）**。

这意味着：
- 这个函数不能修改类的成员变量（除非变量被声明为 `mutable`）。
- 它可以被 `const` 对象调用。

例如：

```cpp
struct A {
    int x;
    void foo() const { cout << x; } // 可以访问x但不能修改
};
```

---

## ✅ 二、为什么重载 `<` 操作符必须是 `const` 函数？

你在使用 `priority_queue` 的时候，底层会频繁地调用比较操作符（比如 `operator<`），用于维护堆结构。这些比较操作不会修改对象的状态。

所以，如果你没有将 `operator<` 声明为 `const`，那么当 `priority_queue` 尝试对两个 `const State` 类型的对象进行比较时，就会出错，因为非 `const` 成员函数不能作用于 `const` 对象。

### ❌ 错误示例（没有 `const`）：

```cpp
struct State {
    int dis;
    bool operator < (const State &rth) {  // 没有 const
        return dis > rth.dis;
    }
};
```

当你这样写的时候：

```cpp
std::priority_queue<State> pq;
pq.push(State{5});
```

编译器可能会报错，提示类似：

```
error: passing ‘const State’ as ‘this’ argument discards qualifiers
```

意思就是：你试图用一个 `const State` 调用一个非 `const` 的成员函数，这会导致“抛弃了 const 性质”，这是不允许的。

---

## ✅ 三、为什么要设计成 const？

C++ 的设计哲学是强调类型安全和语义正确性：

- 如果一个函数不修改对象，就应该将其声明为 `const`。
- 这样可以让编译器帮你避免错误，也能让使用者知道这个函数不会改变对象状态。
- 在标准库中（如 STL 容器和算法），很多地方都会传递 `const` 对象，因此要求你的自定义类型也支持 `const` 正确性。

---

## ✅ 四、总结

| 问题                                | 答案                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| 为什么 `operator<` 必须加 `const`？ | 因为 `priority_queue` 内部比较元素时使用的是 `const` 对象，只有 `const` 成员函数才能被调用。 |
| 不加 `const` 会怎样？               | 编译时报错：“passing 'const Type' as 'this' argument discards qualifiers” |
| `const` 修饰符的作用是什么？        | 表示该成员函数不会修改对象的状态，并允许被 `const` 对象调用。 |

---

## ✅ 最佳实践

```cpp
struct State {
    int dis;

    // ✅ 正确的比较操作符重载，带 const 修饰
    bool operator < (const State &rth) const {
        return dis > rth.dis;
    }
};
```

这样写后，`priority_queue` 才能正常工作！

---

如果你还有关于运算符重载、优先队列或 const 成员函数的问题，欢迎继续提问 😊