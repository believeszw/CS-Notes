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

这是另一个示例程序。 它是一个简单的服务器，它监听端口 40713 上的 `TCP` 连接，一次从其输入中读取一行数据，并在它到达时通过 `ROT13` 做处理并输出。 它使用 `Unix fork()` 调用为每个传入连接创建一个新进程。

```c
Example: Forking ROT13 server
/* For sockaddr_in */
#include <netinet/in.h>
/* For socket functions */
#include <sys/socket.h>

#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

#define MAX_LINE 16384

char rot13_char(char c)
{
    // We don't want to use isalpha here; setting the locale would change
    // which characters are considered alphabetical.
    if ((c >= 'a' && c <= 'm') || (c >= 'A' && c <= 'M'))
        return c + 13;
    else if ((c >= 'n' && c <= 'z') || (c >= 'N' && c <= 'Z'))
        return c - 13;
    else
        return c;
}

void child(int fd)
{
    char outbuf[MAX_LINE+1];
    size_t outbuf_used = 0;
    ssize_t result;

    while (1) {
        char ch;
        result = recv(fd, &ch, 1, 0);
        if (result == 0) {
            break;
        } else if (result == -1) {
            perror("read");
            break;
        }

        // We do this test to keep the user from overflowing the buffer.
        if (outbuf_used < sizeof(outbuf)) {
            outbuf[outbuf_used++] = rot13_char(ch);
        }

        if (ch == '\n') {
            send(fd, outbuf, outbuf_used, 0);
            outbuf_used = 0;
            continue;
        }
    }
}

void run(void)
{
    int listener;
    struct sockaddr_in sin;

    sin.sin_family = AF_INET;
    sin.sin_addr.s_addr = 0;
    sin.sin_port = htons(40713);

    listener = socket(AF_INET, SOCK_STREAM, 0);

#ifndef WIN32
    {
        int one = 1;
        setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &one, sizeof(one));
    }
#endif

    if (bind(listener, (struct sockaddr*)&sin, sizeof(sin)) < 0) {
        perror("bind");
        return;
    }

    if (listen(listener, 16)<0) {
        perror("listen");
        return;
    }



    while (1) {
        struct sockaddr_storage ss;
        socklen_t slen = sizeof(ss);
        int fd = accept(listener, (struct sockaddr*)&ss, &slen);
        if (fd < 0) {
            perror("accept");
        } else {
            if (fork() == 0) {
                child(fd);
                exit(0);
            }
        }
    }
}

int main(int c, char **v)
{
    run();
    return 0;
}
```

那我们是否拥有一次处理多个连接的完美解决方案？ 首先，在某些平台上创建进程（甚至创建线程）可能相当昂贵。 在现实生活中，您需要使用线程池而不是创建新进程。 但更重要的是，线程不会像你想的那样扩展。 如果你的程序需要同时处理数千或数万个连接，那么处理成千上万个线程就不会像每个 `CPU` 只有几个线程一样高效。

但如果线程不是多个连接的答案，那答案是什么？ 在 `Unix` 范例中，可以使非阻塞 `socket` 。 执行此操作的 `Unix` 调用是：
```c
fcntl(fd, F_SETFL, O_NONBLOCK);
```
其中 `fd` 是 `socket` 的文件描述符。\
文件描述符是内核在打开时分配给 `socket` 的编号。 你使用这个数字来指代 `socket` 的 `Unix` 调用。\
一旦你生成了非阻塞 `fd` （套接字），从那时起，每当你对 `fd` 进行网络调用时，调用将立即完成操作或者返回一个特殊的错误代码来表示 `I couldn’t make any progress now, try again.` ， 所以我们的双套接字示例可能天真地写成：
```c
Bad Example: busy-polling all sockets
// This will work, but the performance will be unforgivably bad.
int i, n;
char buf[1024];
for (i=0; i < n_sockets; ++i)
    fcntl(fd[i], F_SETFL, O_NONBLOCK);

while (i_still_want_to_read()) {
    for (i=0; i < n_sockets; ++i) {
        n = recv(fd[i], buf, sizeof(buf), 0);
        if (n == 0) {
            handle_close(fd[i]);
        } else if (n < 0) {
            if (errno == EAGAIN)
                 ;// The kernel didn't have any data for us to read.
            else
                 handle_error(fd[i], errno);
         } else {
            handle_input(fd[i], buf, n);
         }
    }
}
```

































``
