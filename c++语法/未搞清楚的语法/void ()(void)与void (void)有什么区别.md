在C/C++中，`void* (*)(void*)` 和 `void* (void*)` 是两种不同的类型，它们的含义有本质区别：

---

### 1. `void* (void*)`

这是一个 **函数类型**，表示：

- 一个函数，接受一个 `void*` 类型的参数。
- 返回值是 `void*` 类型。

例如：
```c
void* my_function(void* arg) {
    // do something
    return NULL;
}
```

这个函数 `my_function` 的类型就是 `void* (void*)`。

---

### 2. `void* (*)(void*)`

这是一个 **函数指针类型**，表示：

- 一个指向函数的指针。
- 该函数接受一个 `void*` 参数。
- 返回值是 `void*` 类型。

你可以用它来指向上面那个函数：
```c
void* (*func_ptr)(void*) = &my_function;
```

或者直接赋值（函数名会被自动转换为指针）：
```c
void* (*func_ptr)(void*) = my_function;
```

---

### 总结对比

| 类型               | 含义         | 示例                                  |
| ------------------ | ------------ | ------------------------------------- |
| `void* (void*)`    | 函数类型     | 函数定义：`void* my_func(void*)`      |
| `void* (*)(void*)` | 函数指针类型 | 指向函数的指针：`void* (*ptr)(void*)` |

---

### 实际应用举例

#### 使用函数指针作为参数（线程创建常见用法）：
```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void* (*start_routine)(void*), void *arg);
```

这里第三个参数的类型是 `void* (*)(void*)`，即要求你传入一个函数指针，该函数原型必须是 `void* func(void*)`。

---

如有更多关于函数指针、回调函数或线程的问题，欢迎继续提问！