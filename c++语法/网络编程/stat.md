## 基本操作

`stat` 函数是C标准库的一部分，用于获取文件的状态信息。它在 `<sys/stat.h>` 头文件中声明，并且可以用来检索关于文件的各种属性，如文件大小、权限、最后修改时间等。`stat` 函数适用于多种操作系统，包括Unix和类Unix系统（如Linux）。

### 函数原型

```c
#include <sys/stat.h>

int stat(const char *pathname, struct stat *statbuf);
```

- **参数**：
  - `pathname`: 要查询的文件或目录的路径名。
  - `statbuf`: 指向一个 `struct stat` 结构体的指针，用于存储该文件的信息。

- **返回值**：
  - 成功时返回 `0`。
  - 如果发生错误，则返回 `-1` 并设置 `errno` 来指示错误类型。

### `struct stat` 结构体

`struct stat` 包含了大量描述文件状态的字段。以下是其中一些常用的字段：

- **`dev_t st_dev;`**: 文件所在设备的ID。
- **`ino_t st_ino;`**: inode编号。
- **`mode_t st_mode;`**: 文件类型和模式（权限）。
- **`nlink_t st_nlink;`**: 硬链接数。
- **`uid_t st_uid;`**: 文件所有者的用户ID。
- **`gid_t st_gid;`**: 文件所有者的组ID。
- **`dev_t st_rdev;`**: 设备ID（如果是一个特殊文件）。
- **`off_t st_size;`**: 文件大小（字节数），对于常规文件有效。
- **`struct timespec st_atim;`**: 最后访问时间。
- **`struct timespec st_mtim;`**: 最后修改时间。
- **`struct timespec st_ctim;`**: 最后状态改变时间（如权限更改）。
- **`blksize_t st_blksize;`**: 文件系统的I/O块大小。
- **`blkcnt_t st_blocks;`**: 分配给文件的512B块的数量。

### 示例代码

以下是一个简单的示例，展示如何使用 `stat` 函数来获取文件的基本信息：

```c
#include <stdio.h>
#include <sys/stat.h>
#include <errno.h>
#include <time.h>

void print_stat_info(struct stat *sb) {
    printf("File size:\t\t%lld bytes\n", (long long)sb->st_size);
    printf("Blocks allocated:\t%lld\n", (long long)sb->st_blocks);
    printf("Last status change:\t%s", ctime(&sb->st_ctime));
    printf("Last file access:\t%s", ctime(&sb->st_atime));
    printf("Last file modification:\t%s", ctime(&sb->st_mtime));
}

int main() {
    struct stat sb;

    if (stat("example.txt", &sb) == -1) {
        perror("stat");
        return 1;
    }

    printf("File type: ");

    switch (sb.st_mode & S_IFMT) {
        case S_IFBLK:  printf("block device\n");            break;
        case S_IFCHR:  printf("character device\n");         break;
        case S_IFDIR:  printf("directory\n");                break;
        case S_IFIFO:  printf("FIFO/pipe\n");                break;
        case S_IFLNK:  printf("symlink\n");                  break;
        case S_IFREG:  printf("regular file\n");             break;
        case S_IFSOCK: printf("socket\n");                   break;
        default:       printf("unknown?\n");                 break;
    }

    print_stat_info(&sb);

    return 0;
}
```

在这个例子中，我们首先调用 `stat` 函数获取文件 `example.txt` 的状态信息，并将其存储在 `sb` 变量中。然后，通过检查 `st_mode` 字段中的文件类型位，我们可以确定文件的类型。最后，我们打印了一些基本的文件信息，包括文件大小、分配的块数以及各种时间戳。

### 错误处理

当 `stat` 函数失败时，它会返回 `-1` 并设置全局变量 `errno` 来指示错误原因。你可以使用 `perror` 或者 `strerror` 函数来获取并显示相应的错误消息。例如：

```c
if (stat("nonexistentfile.txt", &sb) == -1) {
    perror("stat"); // 输出类似于 "stat: No such file or directory"
}
```

这有助于调试和理解为什么 `stat` 调用失败。常见的错误包括文件不存在 (`ENOENT`)、权限不足 (`EACCES`) 等。

总之，`stat` 函数是一个强大的工具，用于从程序内部获取文件或目录的详细信息。通过解析 `struct stat` 中的不同字段，开发者可以获得丰富的文件元数据，从而实现更加智能和灵活的文件操作。