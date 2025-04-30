# struct timeval

`struct timeval` 是C语言标准库中定义的一个结构体，主要用于获取高精度时间信息。它通常与系统调用和某些库函数一起使用，以提供比 `time_t` 更精确的时间测量（微秒级精度）。`struct timeval` 定义在 `<sys/time.h>` 头文件中。

### 结构体定义

```c
struct timeval {
    time_t      tv_sec;  // 秒数，通常是time_t类型，表示自Epoch以来的秒数
    suseconds_t tv_usec; // 微秒数，类型为suseconds_t，范围从0到999,999
};
```

- **`tv_sec`**: 表示自纪元（Epoch，即1970年1月1日00:00:00 UTC）以来的秒数。
- **`tv_usec`**: 表示微秒部分，范围是从0到999,999。这意味着它可以表示一秒内的微秒精度。

### 使用场景

`struct timeval` 常用于需要精确计时的场合，例如：

- 高精度计时器。
- 计算两个事件之间经过的时间差。
- 在网络编程中设置超时值等。

### 示例代码

以下是一个简单的例子，展示如何使用 `gettimeofday()` 函数来获取当前时间，并将其存储在一个 `struct timeval` 变量中：

```c
#include <stdio.h>
#include <sys/time.h>

int main() {
    struct timeval now;
    
    // 获取当前时间
    if (gettimeofday(&now, NULL) == -1) {
        perror("gettimeofday");
        return 1;
    }

    printf("Current time: %ld seconds and %ld microseconds since the Epoch\n",
           (long)now.tv_sec, (long)now.tv_usec);

    return 0;
}
```

在这个例子中，我们使用了 `gettimeofday()` 函数来获取当前的时间，并将结果存储在 `now` 变量中。然后，我们打印出自纪元以来的秒数和微秒数。

### 注意事项

- `gettimeofday()` 函数在POSIX兼容系统上可用，但在较新的POSIX标准中已被标记为废弃(deprecated)，推荐使用 `clock_gettime()` 函数代替，尤其是在需要更高精度的情况下。
  
- 尽管 `struct timeval` 提供了微秒级别的精度，但它并不保证所有系统都能达到这样的精度。实际可获得的精度取决于系统的硬件和操作系统配置。

通过理解 `struct timeval` 的结构及其用途，可以更准确地处理涉及时间的操作，特别是在需要较高时间分辨率的应用程序开发中。