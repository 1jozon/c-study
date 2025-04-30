## 基本概念

`mmap` 和 `munmap` 是Unix和类Unix操作系统（如Linux）提供的系统调用，用于将文件或设备映射到进程的地址空间。它们提供了内存映射文件的功能，使得文件内容可以像访问内存一样直接被读写。

### mmap

```c
void* mmap(void* start, size_t length, int prot, int flags, int fd, off_t offset);
```

- **参数**：
  - `start`: 指定映射区域的起始地址。通常设为 `NULL`，让操作系统自动选择合适的地址。
  - `length`: 要映射的字节长度。
  - `prot`: 内存保护标志，指定映射区域的访问权限。可以是以下值的组合（使用位或运算符 `|` 组合）：
    - `PROT_READ`: 映射区域可读。
    - `PROT_WRITE`: 映射区域可写。
    - `PROT_EXEC`: 映射区域可执行。
    - `PROT_NONE`: 映射区域不可访问。
  - `flags`: 影响映射操作的行为，常用的有：
    - `MAP_SHARED`: 共享此映射。对映射所做的修改将反映在文件中，并对其他进程可见。
    - `MAP_PRIVATE`: 私有映射。对映射所做的修改仅在此进程中可见，不会影响底层文件。
    - `MAP_ANONYMOUS` (或 `MAP_ANON`): 映射匿名页面，不对应任何文件 (`fd` 必须为 `-1`)。
  - `fd`: 文件描述符，指向要映射的文件。如果使用了 `MAP_ANONYMOUS` 标志，则该参数应设置为 `-1`。
  - `offset`: 文件中的偏移量，指定了从文件哪个位置开始映射。

- **返回值**：成功时返回一个指向映射区域的指针；失败时返回 `MAP_FAILED` (定义为 `(void *) -1`) 并设置 `errno` 表示错误原因。

### munmap

```c
int munmap(void* start, size_t length);
```

- **参数**：
  - `start`: 要解除映射的内存区域的起始地址。必须是之前由 `mmap` 返回的地址。
  - `length`: 要解除映射的内存区域大小，应该与之前 `mmap` 的 `length` 参数相同。

- **返回值**：成功时返回 `0`；失败时返回 `-1` 并设置 `errno` 表示错误原因。

### 使用示例

下面是一个简单的例子，演示如何使用 `mmap` 来映射一个文件并对其进行读取：

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>      // open
#include <sys/mman.h>   // mmap, munmap
#include <sys/stat.h>   // fstat
#include <unistd.h>     // close

int main() {
    const char *filepath = "example.txt";
    int fd = open(filepath, O_RDONLY);
    if (fd == -1) {
        perror("Failed to open file");
        return EXIT_FAILURE;
    }

    struct stat sb;
    if (fstat(fd, &sb) == -1) { // 获取文件信息
        perror("Failed to get file stats");
        close(fd);
        return EXIT_FAILURE;
    }

    char *mapped = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if (mapped == MAP_FAILED) {
        perror("Failed to map file");
        close(fd);
        return EXIT_FAILURE;
    }

    printf("File content:\n%s\n", mapped);

    if (munmap(mapped, sb.st_size) == -1) {
        perror("Failed to unmap file");
    }

    close(fd);
    return EXIT_SUCCESS;
}
```

这个程序首先打开一个文件，然后使用 `fstat` 获取文件大小，接着通过 `mmap` 将文件映射到内存中，最后读取并打印文件内容，并通过 `munmap` 解除映射。

这种方法非常适合处理大文件，因为它避免了频繁的I/O操作，允许直接在内存中高效地读写数据。此外，它还支持进程间共享内存等功能，非常灵活。