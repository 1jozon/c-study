`assert` 是一个用于调试目的的宏，它在C和C++编程语言中被用来测试程序中的假设条件。如果断言失败（即表达式的值为假 `false` 或者数值上为 `0`），则会触发错误处理，通常会导致程序终止并输出相关信息以帮助开发者定位问题。

### 基本用法

`assert` 宏定义在 `<assert.h>`（C语言）或 `<cassert>`（C++语言）头文件中。其基本语法如下：

```c
#include <assert.h> // C语言
// 或
#include <cassert>  // C++语言

assert(expression);
```

- **expression**：这是一个你认为应该为真的表达式。如果该表达式的计算结果为非零（即真），则程序继续执行；如果计算结果为零（即假），则 `assert` 会调用错误处理函数，通常会打印出错信息，并终止程序。

### 触发断言失败时的行为

当断言失败时，`assert` 会：
1. 打印包含失败断言、源代码文件名以及行号的信息。
2. 调用 `abort()` 函数终止程序执行。

例如，在以下代码中：

```c
#include <assert.h>

int main() {
    int x = 5;
    assert(x == 10); // 断言x等于10，但实际上x等于5
    return 0;
}
```

运行此代码时，如果 `x != 10`，则会输出类似如下的错误信息并终止程序：

```
 Assertion failed: (x == 10), function main, file example.c, line 6.
Abort trap: 6
```

### 禁用断言

为了提高生产环境下的性能，可以通过定义 `NDEBUG` 宏来禁用 `assert` 宏。这意味着所有的 `assert` 检查将在编译期间被移除，不会对程序的运行效率产生影响。可以在编译器命令行参数中添加 `-DNDEBUG` 来定义这个宏，或者在源代码的开头位置手动定义：

```c
#define NDEBUG
#include <assert.h>
```

一旦定义了 `NDEBUG`，所有 `assert` 宏都会变成空操作，即它们什么也不做。

### 使用场景

- **验证输入数据的有效性**：确保传入函数的数据符合预期。
- **检查逻辑错误**：在开发阶段确认某些不应发生的逻辑路径没有被执行。
- **资源状态验证**：比如检查指针是否为空，数组索引是否越界等。

### 示例代码

下面是一个使用 `assert` 的示例，演示如何通过断言来保证程序的正确性：

```c
#include <stdio.h>
#include <assert.h>

double divide(double numerator, double denominator) {
    assert(denominator != 0); // 断言分母不为0
    return numerator / denominator;
}

int main() {
    double result;

    result = divide(10.0, 2.0);
    printf("Result: %f\n", result); // 应输出 5.0

    result = divide(10.0, 0.0); // 这将触发断言失败
    printf("This line will not be executed.\n");

    return 0;
}
```

在这个例子中，尝试除以零的操作将会导致 `assert` 失败，程序会在执行到 `divide(10.0, 0.0)` 时终止，并输出相应的错误信息。

总之，`assert` 是一种强大的工具，可以帮助你在开发过程中快速发现潜在的问题。不过需要注意的是，它主要用于开发和调试阶段，在发布版本中应考虑禁用断言以避免不必要的性能开销。