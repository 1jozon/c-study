# int main(int argc,char *argv[])

`int main(int argc, char *argv[])` 是C语言程序的标准入口点，用于定义主函数。这个形式的 `main` 函数允许程序接收命令行参数。下面是对这两个参数的解释：

### 参数解释

- **`argc`** (Argument Count)：这是一个整数，表示传递给程序的命令行参数的数量。请注意，程序名本身也被算作一个参数，因此 `argc` 的最小值是1。

- **`argv`** (Argument Vector)：这是一个指向字符串数组的指针，每个元素都是一个命令行参数。这些参数按照它们在命令行中出现的顺序存储为字符串。`argv[0]` 通常是程序的名称，而后续的 `argv[1]` 到 `argv[argc-1]` 则是实际的命令行参数。

### 示例代码

这里有一个简单的例子来说明如何使用 `argc` 和 `argv` 来处理命令行参数：

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("程序名: %s\n", argv[0]);
    
    if (argc > 1) {
        printf("收到的参数数量: %d\n", argc - 1);
        for (int i = 1; i < argc; ++i) {
            printf("参数 %d: %s\n", i, argv[i]);
        }
    } else {
        printf("没有提供额外的参数。\n");
    }

    return 0;
}
```

如果你编译并运行这段程序如下：

```bash
./myprogram arg1 arg2 arg3
```

输出将是：

```
程序名: ./myprogram
收到的参数数量: 3
参数 1: arg1
参数 2: arg2
参数 3: arg3
```

### 在你的上下文中的应用

基于你之前提供的HTTP服务器示例，`main` 函数通过检查 `argc` 来确保用户提供了正确的端口号作为命令行参数，并将其用作服务器监听的端口：

```c
if (argc != 2)
{
    printf("Usage : %s <port>\n", argv[0]);
    exit(1);
}

// 使用 atoi 将命令行参数转换为整数以获取端口号
serv_adr.sin_port = htons(atoi(argv[1]));
```

在这个例子中，程序期望用户提供一个参数（即监听的端口号）。如果用户没有提供正确数量的参数，程序将打印一条消息指示如何使用该程序，并退出。

### 总结

- `argc` 表示传入程序的参数个数。
- `argv` 是一个字符指针数组，包含了所有的命令行参数。
- 理解和正确处理 `argc` 和 `argv` 对于编写需要接受外部输入或配置的程序非常重要，比如指定文件路径、端口号等。这使得程序更加灵活和可配置。