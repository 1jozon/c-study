# bind

在C++中，`std::bind` 是一个非常有用的工具，它允许你创建函数的“绑定版本”，即可以预先设定某些参数值的函数包装器。这使得你可以更灵活地使用函数指针、函数对象以及成员函数。`std::bind` 主要位于 `<functional>` 头文件中。

### 基本用法

`std::bind` 的基本语法如下：

```cpp
auto bound_function = std::bind(some_function, arg1, arg2, ..., argN);
```

这里，`some_function` 是你要绑定的函数或可调用对象，`arg1` 到 `argN` 是传递给该函数的参数。这些参数可以是具体的值、占位符（用于延迟指定参数）或是其他绑定表达式的结果。

### 占位符

为了支持延迟绑定部分参数，C++ 提供了一组占位符 `_1`, `_2`, `_3` 等等，它们也定义在 `<functional>` 中。占位符表示实际调用时提供的参数的位置。

### 示例

#### 1. 绑定普通函数

假设有一个简单的函数：

```cpp
void print_sum(int a, int b) {
    std::cout << "Sum: " << (a + b) << std::endl;
}
```

我们可以使用 `std::bind` 来创建一个新的函数对象，其中第一个参数被固定为 `5`：

```cpp
#include <iostream>
#include <functional> // For std::bind and placeholders

void print_sum(int a, int b) {
    std::cout << "Sum: " << (a + b) << std::endl;
}

int main() {
    auto bound_print_sum = std::bind(print_sum, 5, std::placeholders::_1);
    
    bound_print_sum(10); // 输出: Sum: 15
}
```

在这个例子中，`bound_print_sum` 是一个新的函数对象，它总是将第一个参数设为 `5`，而第二个参数则由调用者提供。

#### 2. 绑定成员函数

对于类的成员函数，除了函数本身外，还需要绑定到的对象实例。这里我们使用另一个占位符 `_1` 来代表调用时传入的对象实例。

```cpp
#include <iostream>
#include <functional>

class Calculator {
public:
    void multiply_and_print(int a, int b) const {
        std::cout << "Result: " << (a * b) << std::endl;
    }
};

int main() {
    Calculator calc;
    auto bound_multiply_and_print = std::bind(&Calculator::multiply_and_print, &calc, std::placeholders::_1, 4);

    bound_multiply_and_print(3); // 输出: Result: 12
}
```

在这个例子中，`bound_multiply_and_print` 被绑定到了 `Calculator` 类的一个实例上，并且第二个参数固定为 `4`。

#### 3. 使用多个占位符

当你想要延迟绑定多个参数时，可以使用多个占位符：

```cpp
#include <iostream>
#include <functional>

void print_three_numbers(int x, int y, int z) {
    std::cout << "Numbers: " << x << ", " << y << ", " << z << std::endl;
}

int main() {
    auto bound_function = std::bind(print_three_numbers, std::placeholders::_1, 5, std::placeholders::_2);
    
    bound_function(1, 3); // 输出: Numbers: 1, 5, 3
}
```

在这个例子中，第一个和第三个参数由调用者提供，而中间的参数被固定为 `5`。

### 注意事项

- **性能**：虽然 `std::bind` 非常方便，但它可能会引入一些运行时开销。对于性能敏感的应用程序，可能需要考虑使用lambda表达式作为替代方案。
  
- **兼容性**：从C++11开始引入了lambda表达式，它提供了比 `std::bind` 更加直观和灵活的方式来实现类似的功能。因此，在现代C++代码中，通常推荐使用lambda表达式而不是 `std::bind`，除非有特别的理由需要使用后者。

例如，上面的例子可以通过lambda表达式简化为：

```cpp
auto bound_print_sum = [](int b) { print_sum(5, b); };
bound_print_sum(10); // 输出: Sum: 15
```

这种方式不仅更加直观，而且通常具有更好的性能。然而，`std::bind` 在处理复杂场景，如自动推导返回类型或需要深度嵌套的绑定时，仍然非常有用。

# function

在C++中，`std::function` 是一个通用的、多态的函数封装器，它位于 `<functional>` 头文件中。`std::function` 可以用来存储、复制和调用任何可调用的目标（如普通函数、lambda表达式、绑定表达式或其他函数对象），只要它们的签名与 `std::function` 的模板参数匹配。

### 基本概念

`std::function` 的声明形式如下：

```cpp
std::function<返回类型(参数类型列表)> 函数名;
```

它可以存储任何类型的可调用对象，只要这些对象能够被调用并且其签名与 `std::function` 定义的签名相匹配。这使得 `std::function` 非常适合用于回调机制、异步编程以及其他需要将不同类型的可调用实体统一处理的场景。

### 示例

#### 1. 存储普通函数

首先定义一个普通函数，并使用 `std::function` 来存储它。

```cpp
#include <iostream>
#include <functional> // For std::function

int add(int a, int b) {
    return a + b;
}

int main() {
    std::function<int(int, int)> func = add; // 使用 std::function 存储普通函数
    std::cout << "Result: " << func(2, 3) << std::endl; // 输出: Result: 5
}
```

#### 2. 存储Lambda表达式

`std::function` 同样可以存储Lambda表达式。

```cpp
#include <iostream>
#include <functional>

int main() {
    std::function<int(int, int)> lambda_func = [](int a, int b) -> int {
        return a * b;
    };
    std::cout << "Result: " << lambda_func(4, 5) << std::endl; // 输出: Result: 20
}
```

#### 3. 存储成员函数

为了存储成员函数，你需要提供一个指向该成员函数的指针以及一个指向该类实例的指针或引用。

```cpp
#include <iostream>
#include <functional>

class Calculator {
public:
    int multiply(int a, int b) const {
        return a * b;
    }
};

int main() {
    Calculator calc;
    std::function<int(const Calculator*, int, int)> member_func = &Calculator::multiply;
    
    // 调用时需要传递类实例作为第一个参数
    std::cout << "Result: " << member_func(&calc, 6, 7) << std::endl; // 输出: Result: 42
}
```

不过，通常情况下，更简洁的做法是结合 `std::bind` 或者直接使用Lambda表达式来简化这个过程。

#### 4. 结合 `std::bind` 使用

使用 `std::bind` 可以方便地将成员函数绑定到特定的对象实例上。

```cpp
#include <iostream>
#include <functional>

class Calculator {
public:
    int multiply(int a, int b) const {
        return a * b;
    }
};

int main() {
    Calculator calc;
    auto bound_member_func = std::bind(&Calculator::multiply, &calc, std::placeholders::_1, std::placeholders::_2);
    std::function<int(int, int)> func = bound_member_func;

    std::cout << "Result: " << func(8, 9) << std::endl; // 输出: Result: 72
}
```

### 应用场景

- **回调机制**：`std::function` 提供了一种统一的方式来处理各种类型的回调函数。
  
  ```cpp
  void execute(std::function<void()> callback) {
      callback();
  }
  
  int main() {
      execute([](){ std::cout << "Callback executed!" << std::endl; });
  }
  ```

- **异步编程**：在异步操作完成后执行某些代码块时非常有用。
  
- **策略模式**：可以在运行时动态改变算法的行为。

### 注意事项

- **性能**：虽然 `std::function` 提供了极大的灵活性，但它也带来了一些额外的开销。对于性能敏感的应用程序，应当谨慎使用，并考虑是否可以通过其他方式（如模板）达到相同的目的。
  
- **空检查**：`std::function` 对象可以为空（即没有绑定任何可调用对象）。在调用之前，最好检查它是否有效。

  ```cpp
  if (func) {
      func(); // 确保 func 不为空
  }
  ```

通过灵活运用 `std::function`，你可以编写出更加模块化和易于维护的代码，特别是在需要实现回调机制或多态行为的情况下。