# \# ifndef LOCKER_H #define LOCKER_H的作用是什么，如何使用，示例一下

`#ifndef LOCKER_H`、`#define LOCKER_H` 和 `#endif` 是C/C++中用于条件编译的预处理指令，主要用于防止头文件的重复包含，这种技术通常称为“包含卫士”或“头文件卫士”。

- `#ifndef LOCKER_H`：意为“如果未定义 LOCKER_H”，检查是否已经定义了名为 LOCKER_H 的宏。如果没有定义，则执行接下来的代码直到遇到 `#endif`。
- `#define LOCKER_H`：一旦进入条件编译块，即定义 LOCKER_H 宏，以确保该头文件中的内容不会被再次编译。
- `#endif`：结束条件编译块。

使用这些指令可以避免由于多次包含同一个头文件而导致的重复定义错误。下面是一个简单的示例来说明如何在实际中使用它们：

假设我们有一个名为 `Locker.h` 的头文件，其内容可能如下所示：

```cpp
#ifndef LOCKER_H    // 如果没有定义 LOCKER_H
#define LOCKER_H    // 定义 LOCKER_H

// 这里开始是头文件的实际内容
class Locker {
public:
    Locker();  // 构造函数
    ~Locker(); // 析构函数

    void lock();   // 上锁方法
    void unlock(); // 解锁方法

private:
    bool isLocked; // 锁状态
};

#endif // 结束 LOCKER_H 的条件编译
```

在这个例子中，如果其他源文件包含了 `Locker.h` 头文件，上述代码将确保即使多次包含（直接或间接）这个头文件，其中的类定义也只会被编译一次。这样就可以避免重复定义错误，例如当两个不同的文件都尝试包含 `Locker.h` 时，或者一个文件通过多条路径间接包含了同一个头文件。

使用这个头文件的源文件只需要简单地写：

```cpp
#include "Locker.h"

// 使用 Locker 类进行编程
```

这样做即使 `Locker.h` 被多个文件包含，或是某个文件包含了它两次，也可以确保 `Locker` 类只被定义一次，从而避免编译错误。