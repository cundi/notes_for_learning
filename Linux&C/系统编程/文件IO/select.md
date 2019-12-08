# Select

系统调用`select()`提供了一种同步方式的多路复用实现。

```c
#include <sys/select.h>

int select (int n,
            fd_set *readfds,
            fd_set *writefds,
            fd_set *exceptfds,
            struct timeval *timeout);

FD_CLR(int fd, fd_set *set);
FD_ISSET(int fd, fd_set *set);
FD_SET(int fd, fd_set *set);
FD_ZERO(fd_set *set);
```

`timeout`参数是一个指向`timeval`结构体的指针，其定义如下：

```c
#include <sys/time.h>

struct timeval {
        long tv_sec;         /* seconds */
        long tv_usec;        /* microseconds */
};
```

`FD_ZERO`可以移除指定集合中的全部文件描述符，它应该在调用`select()`之前使用。

```c
fd_set writefds;

FD_ZERO(&writefds);
```

`FD_SET`向指定集合中添加一个文件描述符，而`FD_CLR`则从给定集合中移除一个文件描述符。

```c
FD_SET(fd, &writefds);    /* add 'fd' to the set */
FD_CLR(fd, &writefds);    /* oops, remove 'fd' from the set */
```

## 返回值和错误代码

## sample

本例中仅处理一个文件描述符，所以并不是真正的多路复用，只是让为清晰的说明使用方法。

```c
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

#define TIMEOUT 5       /* select timeout in seconds */
#define BUF_LEN 1024    /* read buffer in bytes */

int main (void)
{
        struct timeval tv;
        fd_set readfds;
        int ret;

        /* Wait on stdin for input. */
    FD_ZERO(&readfds);
        FD_SET(STDIN_FILENO, &readfds);

        /* Wait up to five seconds. */
        tv.tv_sec = TIMEOUT;
        tv.tv_usec = 0;

        /* All right, now block! */
        ret = select (STDIN_FILENO + 1,
                      &readfds,
                      NULL,
                      NULL,
                      &tv);
        if (ret == −1) {
                perror ("select");
                return 1;
        } else if (!ret) {
                printf ("%d seconds elapsed.\n", TIMEOUT);
                return 0;
        }

        /*
         * Is our file descriptor ready to read?
         * (It must be, as it was the only fd that
         * we provided and the call returned
         * nonzero, but we will humor ourselves.)
         */
        if (FD_ISSET(STDIN_FILENO, &readfds)) {
                char buf[BUF_LEN+1];
                int len;

                /* guaranteed to not block */
                len = read (STDIN_FILENO, buf, BUF_LEN);
                if (len == −1) {
                        perror ("read");
                        return 1;
                }

                if (len) {
                        buf[len] = '\0';
                        printf ("read: %s\n", buf);
                }

                return 0;
        }

        fprintf (stderr, "This should not happen!\n");
        return 1;
}
```