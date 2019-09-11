# 对异步IO的介绍

大多数初级程序员开始阻塞IO调用。如果一个IO调用是同步的，当你调用它时，它在操作完成之前不会返回，或者直到过了很长时间您的网络堆栈放弃了。例如，当在TCP连接上调用 `connect()` 时，操作系统会将 `SYN` 数据包发送到TCP连接另一端的主机。它不会将控制权返回给您的应用程序，直到它从相对的主机收到 `SYN ACK` 数据包，或者直到它已经过了足够的时间才决定放弃它为止。

这是一个使用阻塞网络调用的非常简单的客户端的示例。它打开与 `www.google.com` 的连接，向其发送简单的 `HTTP` 请求，并将响应打印到 `stdout` 。

一个简单的阻塞的 `HTTP` 客户端例子：
```c
/* For sockaddr_in */
#include <netinet/in.h>
/* For socket functions */
#include <sys/socket.h>
/* For gethostbyname */
#include <netdb.h>

#include <unistd.h>
#include <string.h>
#include <stdio.h>

int main(int c, char **v)
{
    const char query[] =
        "GET / HTTP/1.0\r\n"
        "Host: www.google.com\r\n"
        "\r\n";
    const char hostname[] = "www.google.com";
    struct sockaddr_in sin;
    struct hostent * h;
    const char * cp;
    int fd;
    ssize_t n_written, remaining;
    char buf[1024];

    // Look up the IP address for the hostname.
    // Watch out; this isn't threadsafe on most platforms.
    h = gethostbyname(hostname);
    if (!h) {
        fprintf(stderr, "Couldn't lookup %s: %s", hostname, hstrerror(h_errno));
        return 1;
    }
    if (h->h_addrtype != AF_INET) {
        fprintf(stderr, "No ipv6 support, sorry.");
        return 1;
    }

    // Allocate a new socket
    fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        perror("socket");
        return 1;
    }

    // Connect to the remote host.
    sin.sin_family = AF_INET;
    sin.sin_port = htons(80);
    sin.sin_addr = * (struct in_addr*)h->h_addr;
    if (connect(fd, (struct sockaddr*) &sin, sizeof(sin))) {
        perror("connect");
        close(fd);
        return 1;
    }

    // Write the query.
    // XXX Can send succeed partially?
    cp = query;
    remaining = strlen(query);
    while (remaining) {
      n_written = send(fd, cp, remaining, 0);
      if (n_written <= 0) {
        perror("send");
        return 1;
      }
      remaining -= n_written;
      cp += n_written;
    }

    // Get an answer back.
    while (1) {
        ssize_t result = recv(fd, buf, sizeof(buf), 0);
        if (result == 0) {
            break;
        } else if (result < 0) {
            perror("recv");
            close(fd);
            return 1;
        }
        fwrite(buf, 1, result, stdout);
    }

    close(fd);
    return 0;
}
```

上面代码中的所有网络调用都是阻塞的： `gethostbyname()` 在解析 `www.google.com` 成功或失败之前不会返回; 在连上之前连接不会返回; `recv` 呼叫在收到数据或关闭之前不会返回; 并且 `send` 调用直到它至少将其输出刷新到内核的写缓冲区才会返回。

现在，阻塞 `IO` 并不一定是不合适的。 如果没有其他任何你希望你的程序在此期间做什么，阻塞 `IO` 是个不错的选择。但是假设你需要编写一个程序来同时处理多个连接。 为了使我们的示例具体：假设您想要从两个连接读取输入，并且您不知道哪个连接将首先获得输入，那就不能这样说了。

反面教材：
```c
/* This won't work. */
char buf[1024];
int i, n;
while (i_still_want_to_read()) {
    for (i=0; i<n_sockets; ++i) {
        n = recv(fd[i], buf, sizeof(buf), 0);
        if (n==0)
            handle_close(fd[i]);
        else if (n<0)
            handle_error(fd[i], errno);
        else
            handle_input(fd[i], buf, n);
    }
}
```

因为如果数据首先到达 `fd[2]` ，你的程序不会尝试从 `fd[2]` 读取，直到 `fd[0]` 和 `fd[1]` 收到数据并结束操作。

有时人们使用多线程或多进程来解决这个问题。进行多线程处理的最简单方法之一是使用单独的进程（或线程）来处理每个连接。 由于每个连接都有自己的进程，因此等待一个连接的阻塞 `IO` 调用不会使任何其他连接的进程阻塞。





























``
