## recv

`recv` 是一个在C语言中用于网络编程的函数，主要用于从套接字（socket）接收数据。它定义在 `<sys/socket.h>` 头文件中，是实现TCP或UDP等协议通信的重要部分。通过 `recv` 函数，程序可以从已连接的套接字读取数据。

### 函数原型

```c
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

- **`sockfd`**: 套接字描述符，表示要从中读取数据的套接字。
- **`buf`**: 指向缓冲区的指针，用于存储接收到的数据。
- **`len`**: 缓冲区的大小，即最多可以接收多少字节的数据。
- **`flags`**: 调用行为选项标志，通常设置为0，但也可以使用不同的标志来改变函数的行为。

### 返回值

- 成功时返回实际接收到的字节数。如果对等方已经关闭连接并且没有剩余数据待读取，则返回0。
- 如果发生错误，返回值为 -1，并且 `errno` 变量会被设置以指示具体的错误类型。

### 标志（flags）

可以通过 `flags` 参数指定一些特殊的行为：

- **`MSG_PEEK`**: 查看到来的消息。数据将被复制到缓冲区中，但不会从输入队列中移除。
- **`MSG_WAITALL`**: 要求调用阻塞直到至少请求的字节数已经被接收。但如果遇到错误或者连接关闭，可能会返回更少的数据。
- **`MSG_DONTWAIT`**: 以非阻塞模式操作。如果当前没有数据可读，立即返回而不是等待。

### 使用示例

下面是一个简单的例子，展示了如何使用 `recv` 函数从服务器接收数据：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUFFER_SIZE 1024

int main() {
    int sock = 0;
    struct sockaddr_in serv_addr;
    char buffer[BUFFER_SIZE] = {0};
    
    // 创建套接字
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\n Socket creation error \n");
        return -1;
    }
    
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(8080);
    
    // 将IP地址转换为二进制形式
    if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr) <= 0) {
        printf("\nInvalid address/ Address not supported \n");
        return -1;
    }
    
    // 连接到服务器
    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        printf("\nConnection Failed \n");
        return -1;
    }

    // 接收数据
    int valread = recv(sock, buffer, BUFFER_SIZE, 0);
    if (valread < 0) {
        perror("recv failed");
        close(sock);
        return -1;
    }
    printf("%s\n", buffer);

    close(sock); // 关闭套接字
    return 0;
}
```

在这个例子中，客户端首先创建了一个套接字，然后连接到运行在同一台机器上的服务器（IP地址为`127.0.0.1`，端口号为`8080`）。一旦连接成功，客户端就调用 `recv` 来接收数据并打印出来。

### 注意事项

- **阻塞与非阻塞**：默认情况下，`recv` 是阻塞的，这意味着如果没有数据可用，它会一直等待直到有数据到达。如果你希望避免这种情况，可以使用 `MSG_DONTWAIT` 标志或将套接字设置为非阻塞模式。
  
- **处理部分接收**：即使你请求了特定数量的数据，`recv` 也可能只返回部分数据。因此，建议循环调用 `recv` 直到接收到所需的所有数据或连接关闭。

- **错误处理**：总是检查 `recv` 的返回值以确定是否发生了错误或对方是否关闭了连接（返回值为0）。

通过理解 `recv` 函数的工作原理及其参数，你可以有效地在网络应用中实现数据接收功能。



在使用 `recv` 函数时，`flags` 参数允许你指定不同的选项来控制接收行为。这些标志可以通过位或（`|`）操作组合使用。以下是几个常见的 `flags` 标志及其含义：

### 常见的 `flags` 标志

1. **`MSG_PEEK`**:
   - **值**: 通常为 `0x2`
   - **描述**: 这个标志让 `recv` 函数查看即将到来的消息而不实际从输入队列中移除数据。这意味着后续调用 `recv` 或其他类似的接收函数将能够再次访问相同的数据。
2. **`MSG_WAITALL`**:
   - **值**: 通常为 `0x100`
   - **描述**: 要求 `recv` 阻塞直到请求的所有数据都被接收到。如果设置此标志，`recv` 只有在接收到至少 `len` 字节的数据后才会返回。然而，如果有错误发生或者连接被关闭，可能会返回少于请求长度的数据。
3. **`MSG_DONTWAIT`**:
   - **值**: 通常为 `0x40`
   - **描述**: 将套接字设置为非阻塞模式仅对本次调用有效。如果此时没有可用的数据，`recv` 将立即返回 `-1` 并设置 `errno` 为 `EAGAIN` 或 `EWOULDBLOCK`，而不是等待数据到达。
4. **`MSG_OOB`**:
   - **值**: 通常为 `0x1`
   - **描述**: 处理带外数据（out-of-band data）。带外数据是一种特殊的紧急数据传输机制，在TCP协议中实现。并非所有协议都支持带外数据。
5. **`MSG_TRUNC`**:
   - **注意**: 此标志一般用于 `send` 和 `recv` 的返回值检查，并不作为输入参数传递给 `recv` 函数。它指示接收缓冲区太小而无法容纳整个消息，但不会影响实际接收到的数据量。
6. **`MSG_CTRUNC`**:
   - **注意**: 类似于 `MSG_TRUNC`，但它适用于控制信息而非数据本身。主要用于辅助数据的接收（如通过 `recvmsg`）。

##### 在 `recv` 函数的上下文中，`flags` 参数设置为 `0` 表示不使用任何特殊标志，即采用默认的行为模式。这意味着：

- **阻塞模式**：如果没有指定如 `MSG_DONTWAIT` 这样的非阻塞标志，`recv` 调用将处于阻塞模式。如果当前没有数据可读，函数会等待直到有数据到达或连接关闭。
- **正常接收数据**：函数将尝试从套接字接收最多 `len` 字节的数据，并且一旦数据到达就会将其从输入队列中移除（与 `MSG_PEEK` 不同）。
- **无特殊处理**：不会对带外数据 (`MSG_OOB`) 或者其他需要特殊处理的数据进行操作。