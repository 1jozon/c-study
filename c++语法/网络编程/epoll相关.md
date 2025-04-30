## event.events = EPOLLIN | EPOLLET | EPOLLRDHUP;

在Linux环境下，`epoll` 是一种高效的I/O事件通知机制，常用于处理大量并发连接的场景。通过 `epoll_ctl` 函数可以向 `epoll` 实例添加、修改或删除感兴趣的文件描述符及其关联的事件。你提到的代码片段：

```c
event.events = EPOLLIN | EPOLLET | EPOLLRDHUP;
```

这行代码设置了 `struct epoll_event` 结构体中 `events` 字段的值，指定了对该文件描述符感兴趣的事件类型。下面是对每个标志的详细解释：

### 事件标志

1. **`EPOLLIN`**:
   - **含义**: 当前文件描述符上有数据可读（即输入缓冲区中有数据可供读取）。
   - **用途**: 监听读事件，通常用于服务器端等待客户端发送数据。

2. **`EPOLLET`**:
   - **含义**: 边缘触发模式（Edge Triggered, ET）。与默认的水平触发模式（Level Triggered, LT）不同，边缘触发模式仅在状态变化时（如从不可读变为可读）通知一次。
   - **用途**: 提供更高的性能和更精细的控制，但同时也增加了编程复杂度，因为需要确保完全处理完所有可用的数据，否则可能会错过后续的通知。

3. **`EPOLLRDHUP`**:
   - **含义**: 对端关闭连接（Remote Disconnection Hang Up）。当对端关闭连接或者半关闭连接时，会产生这个事件。
   - **用途**: 允许检测到对端关闭连接的情况，从而及时清理资源或进行其他必要的操作。

### 示例代码

以下是一个简单的示例，展示了如何使用这些标志来设置 `epoll` 监听的事件：

```c
#include <sys/epoll.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

#define MAX_EVENTS 10
#define SERVER_FD 0 // 假设这是服务器监听套接字的文件描述符

int main() {
    int epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        return 1;
    }

    struct epoll_event event;
    event.events = EPOLLIN | EPOLLET | EPOLLRDHUP; // 设置感兴趣的事件
    event.data.fd = SERVER_FD; // 将服务器监听套接字绑定到event结构体

    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, SERVER_FD, &event) == -1) {
        perror("epoll_ctl: add");
        close(epoll_fd);
        return 1;
    }

    struct epoll_event events[MAX_EVENTS];
    int n_fds;

    while (1) {
        n_fds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (n_fds == -1) {
            perror("epoll_wait");
            close(epoll_fd);
            return 1;
        }

        for (int i = 0; i < n_fds; i++) {
            if (events[i].events & EPOLLIN) {
                printf("Data available on fd %d\n", events[i].data.fd);
                // 处理读事件...
            }
            if (events[i].events & EPOLLRDHUP) {
                printf("Peer closed connection on fd %d\n", events[i].data.fd);
                // 处理对端关闭连接事件...
            }
            // 可以根据需要处理其他事件...
        }
    }

    close(epoll_fd);
    return 0;
}
```

### 注意事项

- **边缘触发 vs 水平触发**：选择边缘触发模式 (`EPOLLET`) 需要特别注意，因为它只会在状态改变时触发一次通知。这意味着你需要确保一次性读取所有可用的数据，否则可能会丢失通知。相比之下，水平触发模式 (`EPOLL_LT`, 默认模式) 会持续通知只要有数据未被读取。

- **非阻塞模式**：当使用边缘触发模式时，通常需要将相关的文件描述符设置为非阻塞模式，以便能够在循环中尽可能多地读取数据，直到没有更多数据为止。

- **处理 `EPOLLRDHUP`**：当检测到 `EPOLLRDHUP` 事件时，应该检查并清理相应的资源，比如关闭对应的文件描述符，避免资源泄露。

通过正确配置和使用这些标志，可以构建出高效且响应迅速的网络服务器或其他需要处理大量并发I/O操作的应用程序。理解每种事件标志的作用及其对程序逻辑的影响对于开发稳定可靠的系统至关重要。

## EPOLLRDHUP命名规则

`EPOLLRDHUP` 是 Linux `epoll` 机制中的一个事件标志，用于通知程序对端已关闭连接或进行了半关闭操作。这个名称遵循一定的命名规则和约定，理解这些可以帮助更好地记忆和使用相关的标志。

### 名称解析

- **`EPOLL`**: 表示这是与 `epoll` 相关的事件标志。`epoll` 是 Linux 特有的 I/O 事件通知接口，提供了一种高效的方式来管理多个文件描述符。
  
- **`RDHUP`**: 这是 "Read Hang Up" 的缩写，意味着读端挂起（即对端关闭了连接）。具体来说：
  - **`RD`**: 来自 "Read"，表示这是一个关于读操作的事件。
  - **`HUP`**: 来自 "Hang Up"，传统上用来表示电话线路断开，在这里指的是网络连接的一端关闭。

因此，`EPOLLRDHUP` 整体的意思是：当检测到对端关闭连接或进行了半关闭（shutdown）操作时触发的事件。

### 命名规则

在 `epoll` 和其他类似的 POSIX API 中，事件标志通常遵循以下命名惯例：

1. **前缀**:
   - 使用特定于该接口的前缀来标识其所属的系统调用或库。对于 `epoll`，所有事件标志都以 `EPOLL` 开头。
   
2. **核心功能描述**:
   - 紧随前缀之后的部分描述了事件的核心功能。例如，`IN` 对应可读事件 (`EPOLLIN`)，`OUT` 对应可写事件 (`EPOLLOUT`)。
   
3. **特殊情况描述**:
   - 对于一些特殊的情况或条件，可能会添加额外的后缀。比如 `RDHUP` 用来表示远程挂起（对端关闭），而 `ET` 表示边缘触发模式 (`EPOLLET`)。

### 其他常见标志及其含义

为了更全面地理解这种命名方式，下面列出了一些其他的 `epoll` 事件标志及其含义：

- **`EPOLLIN`**: 可读事件。当文件描述符上有数据可读时触发。
- **`EPOLLOUT`**: 可写事件。当文件描述符准备好接受输出数据时触发。
- **`EPOLLET`**: 边缘触发模式。与默认的水平触发模式不同，它只在状态变化时通知一次。
- **`EPOLLERR`**: 错误事件。当文件描述符发生错误时触发。
- **`EPOLLHUP`**: 挂起事件。当文件描述符被挂起（通常是连接被关闭）时触发。
- **`EPOLLRDHUP`**: 如前所述，当对端关闭连接或进行了半关闭操作时触发。

### 总结

`EPOLLRDHUP` 的命名规则反映了其功能特性——即监测到对端关闭连接或执行了半关闭操作。通过这种方式命名，开发者可以快速识别出每个标志的作用，从而正确配置 `epoll` 监听感兴趣的事件。了解这些命名惯例有助于更有效地使用 `epoll` 及其他类似的异步I/O接口，并能提高代码的可读性和维护性。

## EPOLLONESHOT 标志

在 `epoll` 的上下文中，"one-shot" 通常指的是事件监听的一种模式，具体通过设置 `EPOLLONESHOT` 标志来实现。这种模式确保了某个文件描述符上的事件仅触发一次，直到再次明确地重新武装（re-arm）该文件描述符。

### EPOLLONESHOT 标志

- **含义**: 当为某个文件描述符设置了 `EPOLLONESHOT` 标志后，该文件描述符上发生的事件只会通知一次。即使该文件描述符的状态再次满足条件（例如，可读或可写），也不会再触发相应的事件，除非通过调用 `epoll_ctl` 函数并指定 `EPOLL_CTL_MOD` 来重新设置该文件描述符的事件监听状态。
  
- **用途**: 这种机制可以用于减少不必要的事件通知，从而提高性能和简化并发处理逻辑。它特别适用于那些需要对每个事件进行复杂处理且不希望在同一事件上多次触发的应用场景。

### 使用示例

以下是一个简单的例子，展示了如何使用 `EPOLLONESHOT` 标志：

```c
#include <sys/epoll.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

#define MAX_EVENTS 10
#define SERVER_FD 0 // 假设这是服务器监听套接字的文件描述符

int main() {
    int epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        return 1;
    }

    struct epoll_event event;
    event.events = EPOLLIN | EPOLLET | EPOLLRDHUP | EPOLLONESHOT; // 设置感兴趣的事件及one-shot模式
    event.data.fd = SERVER_FD; // 将服务器监听套接字绑定到event结构体

    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, SERVER_FD, &event) == -1) {
        perror("epoll_ctl: add");
        close(epoll_fd);
        return 1;
    }

    struct epoll_event events[MAX_EVENTS];
    int n_fds;

    while (1) {
        n_fds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (n_fds == -1) {
            perror("epoll_wait");
            close(epoll_fd);
            return 1;
        }

        for (int i = 0; i < n_fds; i++) {
            if (events[i].events & EPOLLIN) {
                printf("Data available on fd %d\n", events[i].data.fd);
                // 处理读事件...
                
                // 由于使用了one-shot模式，需要重新设置以接收下一次事件
                if (epoll_ctl(epoll_fd, EPOLL_CTL_MOD, events[i].data.fd, &event) == -1) {
                    perror("epoll_ctl: mod");
                    close(epoll_fd);
                    return 1;
                }
            }
            if (events[i].events & EPOLLRDHUP) {
                printf("Peer closed connection on fd %d\n", events[i].data.fd);
                // 处理对端关闭连接事件...
            }
            // 可以根据需要处理其他事件...
        }
    }

    close(epoll_fd);
    return 0;
}
```

### 关键点

- **一次性通知**: 使用 `EPOLLONESHOT` 后，当某个文件描述符上的事件被触发一次之后，除非重新设置，否则不会再收到关于该文件描述符的通知。
  
- **重新武装**: 在处理完一个事件后，如果还想继续监听该文件描述符的事件，必须显式地调用 `epoll_ctl` 并使用 `EPOLL_CTL_MOD` 操作来重新设置该文件描述符的监听状态。

### 优点

- **降低负载**: 对于高并发场景下的大量文件描述符，使用 one-shot 模式可以避免频繁触发相同的事件，从而减少系统开销。
  
- **简化逻辑**: 它使得开发者能够更精确地控制何时以及如何处理特定的I/O事件，有助于编写更加清晰和易于维护的代码。

总之，`one-shot` 模式提供了一种有效的方式来管理 `epoll` 监听的文件描述符及其事件，特别适合需要精细控制和高效处理的网络应用程序。理解并正确应用这种模式可以显著提升程序的性能和稳定性。

## 为什么要设置文件描述符为非阻塞状态

在使用 `epoll` 或其他异步I/O机制时，将文件描述符设置为非阻塞状态是一个常见的做法。这样做有几个关键的好处和原因，尤其是在处理高并发连接的服务器应用中。

### 1. 避免阻塞操作

**非阻塞模式**允许文件描述符上的操作（如读写）立即返回，而不是挂起当前线程或进程等待操作完成。这对于提高应用程序的整体响应性和效率至关重要。

- **避免阻塞读写**: 当在一个非阻塞套接字上调用 `recv` 或 `send` 时，如果暂时没有数据可读或发送缓冲区已满，函数会立即返回 `-1` 并设置 `errno` 为 `EAGAIN` 或 `EWOULDBLOCK`，而不是等待直到条件满足。这使得程序可以继续执行其他任务，而不是被单一的操作阻塞。
  
- **边缘触发模式下的必要性**: 在 `epoll` 的边缘触发 (`EPOLLET`) 模式下，一旦某个事件被触发（例如，有数据可读），如果没有完全处理完所有可用的数据就返回，那么后续对该事件的通知将会丢失。因此，在这种情况下，通常需要以非阻塞方式尽可能多地读取数据，直到无法再读取为止。

### 2. 提高并发处理能力

对于需要处理大量并发连接的应用程序（如Web服务器），非阻塞I/O结合事件驱动模型（如 `epoll`）能够显著提高性能。

- **多路复用**: 使用 `epoll` 可以同时监视多个文件描述符的状态变化。当某个文件描述符变为可读或可写时，`epoll_wait` 函数返回，并告知哪个文件描述符准备好了进行I/O操作。由于这些操作是非阻塞的，可以在不阻塞主线程的情况下高效地处理每个就绪的文件描述符。

- **减少上下文切换开销**: 相比于传统的多线程或多进程模型，非阻塞I/O减少了因频繁的线程/进程切换带来的开销。因为每个连接不需要单独的线程或进程来维持，而是通过事件循环统一管理。

### 3. 更好的资源利用

非阻塞I/O有助于更有效地利用系统资源，特别是CPU和内存。

- **提高CPU利用率**: 由于非阻塞操作不会导致线程或进程长时间处于等待状态，因此CPU可以更多地用于实际计算而非空闲等待，从而提高了整体系统的吞吐量。
  
- **节省内存**: 不需要为每个连接分配独立的栈空间（如在每连接一个线程的模型中），从而降低了内存消耗。

### 示例：设置非阻塞模式

要将一个文件描述符设置为非阻塞模式，可以使用 `fcntl` 函数：

```c
#include <fcntl.h>
#include <unistd.h>

void set_nonblocking(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) {
        perror("fcntl get failed");
        return;
    }
    if (fcntl(fd, F_SETFL, flags | O_NONBLOCK) == -1) {
        perror("fcntl set failed");
    }
}
```

然后，在你的代码中调用这个函数：

```c
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
if (sockfd < 0) {
    perror("socket creation failed");
    exit(EXIT_FAILURE);
}

set_nonblocking(sockfd); // 设置为非阻塞模式
```

### 总结

将文件描述符设置为非阻塞状态的主要目的是为了防止I/O操作阻塞程序的执行流程，从而使得应用能够在等待某些操作完成的同时处理其他任务。这对于构建高性能、响应迅速且能处理大量并发连接的网络服务特别重要。通过与 `epoll` 等高效的事件通知机制结合使用，非阻塞I/O成为了现代网络编程中的核心技术之一。

## epoll与select和poll的区别是什么？

`epoll`、`select` 和 `poll` 都是用于I/O多路复用的机制，它们允许单个进程监视多个文件描述符（如套接字），并等待其中任何一个变为可读或可写。然而，它们在实现细节、性能和使用方式上存在显著差异。

### 1. 基本概念与工作机制

- **`select`**:
  - 是最早的I/O多路复用机制之一，适用于POSIX兼容系统。
  - 它需要你提供三个文件描述符集（可读、可写、异常条件），以及一个超时值。`select`会阻塞直到这些集合中的至少一个文件描述符准备好进行I/O操作。
  - 每次调用`select`都需要重新构建文件描述符集合，并且它有固定的大小限制（通常为1024）。

- **`poll`**:
  - 解决了`select`的一些限制，比如不再有最大文件描述符数量的限制。
  - 使用一个单独的结构体数组来表示感兴趣的文件描述符及其事件类型。虽然克服了`select`的文件描述符数限制问题，但在大规模并发连接的情况下，效率依然不高，因为每次调用都需要遍历整个列表检查每个文件描述符的状态。

- **`epoll`**:
  - 是Linux特有的高性能I/O事件通知接口，特别适合处理大量并发连接。
  - `epoll`通过创建一个epoll实例（文件描述符），然后通过`epoll_ctl`函数向这个实例添加、修改或删除感兴趣的文件描述符及对应的事件。
  - 当某个或某些文件描述符准备就绪时，`epoll_wait`函数返回这些文件描述符的信息。`epoll`支持两种工作模式：水平触发（LT，默认）和边缘触发（ET），提供了更高的灵活性和性能优化空间。

### 2. 性能比较

- **`select`**:
  - 性能较差，尤其是在处理大量文件描述符时。每次调用`select`都需要从用户态复制文件描述符集合到内核态，并且随着文件描述符数量增加，性能下降明显。
  
- **`poll`**:
  - 虽然解决了`select`的文件描述符数量限制问题，但对于每个文件描述符的状态检查仍然需要线性扫描整个列表，因此在高并发场景下效率较低。

- **`epoll`**:
  - 在处理大量并发连接时表现出色。由于采用了更高效的内部数据结构（红黑树和双向链表），使得即使在成千上万个文件描述符中查找准备好的文件描述符也非常高效。
  - 另外，`epoll`还支持边缘触发模式，可以进一步减少不必要的回调次数，提高性能。

### 3. 编程复杂度

- **`select` & `poll`**:
  - 相对简单直接，但随着文件描述符数量的增加，管理和维护变得越来越复杂。

- **`epoll`**:
  - 提供了更多的控制选项（例如，不同的触发模式），这增加了编程的复杂性。但是，对于需要处理大量并发连接的应用来说，这种额外的复杂性换来的是更好的性能和更低的资源消耗。

### 4. 其他区别

- **跨平台兼容性**:
  - `select` 和 `poll` 更加通用，几乎所有Unix-like系统都支持。
  - `epoll` 是Linux特有的，不具有跨平台性。

- **扩展性和伸缩性**:
  - `epoll` 设计之初就是为了应对大规模网络服务器的需求，因此在扩展性和伸缩性方面优于`select` 和 `poll`。

### 示例代码对比

#### `select` 示例

```c
#include <sys/select.h>

fd_set readfds;
FD_ZERO(&readfds);
FD_SET(sockfd, &readfds);

struct timeval timeout;
timeout.tv_sec = 5;
timeout.tv_usec = 0;

int activity = select(sockfd + 1, &readfds, NULL, NULL, &timeout);
if (activity < 0) {
    perror("select failed");
}
```

#### `poll` 示例

```c
#include <poll.h>

struct pollfd fds[1];
fds[0].fd = sockfd;
fds[0].events = POLLIN;

int activity = poll(fds, 1, 5000); // 等待5秒
if (activity < 0) {
    perror("poll failed");
}
```

#### `epoll` 示例

```c
#include <sys/epoll.h>

int epoll_fd = epoll_create1(0);
if (epoll_fd == -1) {
    perror("epoll_create1");
}

struct epoll_event event;
event.events = EPOLLIN;
event.data.fd = sockfd;

if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, sockfd, &event) == -1) {
    perror("epoll_ctl: add");
}

struct epoll_event events[10];
int n_fds = epoll_wait(epoll_fd, events, 10, 5000); // 等待5秒
if (n_fds == -1) {
    perror("epoll_wait");
}
```

综上所述，`epoll` 在处理大量并发连接时相比`select` 和 `poll` 提供了显著的性能优势，尽管它的API相对复杂一些。选择哪种机制取决于具体的应用需求，包括预期的并发连接数、跨平台需求等因素。

## epoll_wait  

`epoll_wait` 是 Linux 系统中 `epoll` 机制的核心函数之一，用于等待事件的发生。它会阻塞当前进程直到指定的文件描述符集合中有事件就绪（例如可读、可写），或者超时发生。

### 函数原型

```c
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

- **`epfd`**: `epoll_create` 创建的 `epoll` 实例的文件描述符。
- **`events`**: 指向一个 `struct epoll_event` 数组的指针，用于存储就绪的事件信息。
- **`maxevents`**: 指定 `events` 数组的最大大小，必须大于0。
- **`timeout`**: 超时时间，单位为毫秒：
  - `-1`: 阻塞直到有事件发生。
  - `0`: 立即返回，无论是否有事件发生。
  - `>0`: 等待指定的毫秒数，如果在此期间内没有事件发生，则超时返回。

### 返回值

- 成功时返回就绪的文件描述符数量，可能为0（表示超时）。
- 失败时返回 `-1` 并设置相应的 `errno`。

### 结构体 `epoll_event`

`epoll_event` 结构体用于定义感兴趣的事件和接收事件通知：

```c
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      // epoll事件类型
    epoll_data_t data;        // 用户数据
};
```

- **`events`**: 事件类型，可以是多个事件的按位或组合，如 `EPOLLIN`, `EPOLLOUT`, `EPOLLRDHUP`, `EPOLLERR`, `EPOLLHUP` 等。
- **`data`**: 可以用来关联用户数据，通常用于存储文件描述符 (`fd`) 或其他相关信息。

### 示例代码

以下是一个简单的示例，展示了如何使用 `epoll_wait` 来监听文件描述符上的事件：

```c
#include <sys/epoll.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <errno.h>

#define MAX_EVENTS 10
#define SERVER_FD 0 // 假设这是服务器监听套接字的文件描述符

void set_nonblocking(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) {
        perror("fcntl get failed");
        return;
    }
    if (fcntl(fd, F_SETFL, flags | O_NONBLOCK) == -1) {
        perror("fcntl set failed");
    }
}

int main() {
    int epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        return 1;
    }

    struct epoll_event event;
    event.events = EPOLLIN; // 监听读事件
    event.data.fd = SERVER_FD; // 将服务器监听套接字绑定到event结构体

    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, SERVER_FD, &event) == -1) {
        perror("epoll_ctl: add");
        close(epoll_fd);
        return 1;
    }

    set_nonblocking(SERVER_FD); // 设置为非阻塞模式

    struct epoll_event events[MAX_EVENTS];
    int n_fds;

    while (1) {
        n_fds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1); // 阻塞直到有事件发生
        if (n_fds == -1) {
            if (errno == EINTR) {
                continue; // 被信号中断，继续等待
            } else {
                perror("epoll_wait");
                close(epoll_fd);
                return 1;
            }
        }

        for (int i = 0; i < n_fds; i++) {
            if (events[i].events & EPOLLIN) {
                printf("Data available on fd %d\n", events[i].data.fd);
                // 处理读事件...
            }
            if (events[i].events & EPOLLRDHUP) {
                printf("Peer closed connection on fd %d\n", events[i].data.fd);
                // 处理对端关闭连接事件...
            }
            // 可以根据需要处理其他事件...
        }
    }

    close(epoll_fd);
    return 0;
}
```

### 注意事项

1. **超时处理**：当 `timeout` 参数为正数时，`epoll_wait` 会在指定的时间后返回，即使没有任何事件发生。这允许应用程序在等待I/O事件的同时执行定时任务。

2. **错误处理**：`epoll_wait` 可能因为被信号中断而返回 `-1` 并设置 `errno` 为 `EINTR`。在这种情况下，通常建议重新调用 `epoll_wait` 继续等待。

3. **非阻塞模式**：虽然 `epoll` 本身是非阻塞的（通过 `timeout` 参数控制），但为了更好地与边缘触发模式配合使用，通常也需要将相关的文件描述符设置为非阻塞模式。

4. **One-shot 模式**：如果启用了 `EPOLLONESHOT` 标志，则每次事件发生后需要重新设置文件描述符以再次监听该事件。

通过正确使用 `epoll_wait` 和相关机制，可以构建出高效且响应迅速的网络服务器或其他需要处理大量并发I/O操作的应用程序。理解并掌握这些技术对于开发高性能系统至关重要。

## errno

在 Unix 和类 Unix 操作系统（如 Linux）中，`errno` 是一个线程局部存储的整型变量，用于报告系统调用或库函数错误的原因。当一个系统调用或库函数遇到错误时，它通常会返回一个特定的错误值（例如 `-1` 或 `NULL`），同时设置 `errno` 为一个表示具体错误类型的数值。

### 常见的 `errno` 错误码

以下是一些常见的 `errno` 错误码及其含义：

- **`EACCES` (13)**: 权限不足。尝试访问需要更高权限的对象。
- **`EAGAIN` (11) / `EWOULDBLOCK`**: 资源暂时不可用。通常发生在非阻塞操作中，表示操作无法立即完成。或者你尝试读取或写入一个设置为非阻塞模式的文件描述符（如套接字、管道等），而此时没有数据可读或写缓冲区已满时，系统调用会返回 `-1` 并设置 `errno` 为 `EAGAIN` 或 `EWOULDBLOCK`（这两个错误码在许多系统上是相同的）。
- **`EBADF` (9)**: 非法文件描述符。传入了一个无效的文件描述符参数。
- **`ECONNREFUSED` (111)**: 连接被拒绝。目标机器积极拒绝了建立连接的尝试。
- **`EEXIST` (17)**: 文件已存在。尝试创建一个已经存在的文件。
- **`EFAULT` (14)**: 坏地址。传递给系统调用的指针指向了非法地址。
- **`EINVAL` (22)**: 非法参数。传递给函数或系统调用的参数无效。
- **`EINTR` (4)**: 系统调用被中断。由于接收到了信号导致系统调用中断。
- **`EIO` (5)**: 输入输出错误。发生了某种形式的输入输出错误。
- **`ENOMEM` (12)**: 内存不足。请求分配内存失败。
- **`ENFILE` (23)**: 系统打开文件过多。达到了系统范围内允许的最大文件数。
- **`ENOENT` (2)**: 无此文件或目录。指定的路径名不存在。
- **`ENOSPC` (28)**: 设备上没有剩余空间。磁盘或设备上没有足够的空间来执行操作。
- **`ENOTDIR` (20)**: 不是目录。期望的是目录，但实际上不是。
- **`EPERM` (1)**: 操作不允许。用户不具备执行该操作的权限。
- **`ETIMEDOUT` (110)**: 连接超时。与某个远程主机的连接尝试超时。

### 使用 `errno`

为了使用 `errno`，你需要包含 `<errno.h>` 头文件，并且在检测到错误后检查其值。例如：

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main() {
    FILE *file = fopen("nonexistent_file.txt", "r");
    if (file == NULL) {
        fprintf(stderr, "Error opening file: %s\n", strerror(errno));
    } else {
        fclose(file);
    }
    return 0;
}
```

在这个例子中，如果 `fopen` 调用失败，则会打印出详细的错误信息，`strerror` 函数用于将 `errno` 转换为对应的错误消息字符串。

### 注意事项

- **多线程环境**：在多线程程序中，每个线程都有自己的 `errno` 实例，因此无需担心线程之间的相互干扰。
- **错误处理**：正确地处理错误代码对于构建健壮的应用程序至关重要。忽略错误可能导致未定义行为或安全漏洞。
- **避免依赖具体的错误码值**：最好使用 `<errno.h>` 中定义的符号常量（如 `EACCES`, `EAGAIN` 等）而不是直接使用数字值，以提高代码的可读性和可维护性。

通过理解和适当利用 `errno`，可以更有效地调试和处理程序中的错误情况，确保应用程序更加稳定可靠。