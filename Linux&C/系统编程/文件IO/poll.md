# Poll

系统调用`poll()`是System V中的一种多路复用解决方案。

```c
#include <poll.h>

int poll (struct pollfd *fds, nfds_t nfds, int timeout);
```

```c
#include <poll.h>

struct pollfd {
        int fd;           /* file descriptor */
        short events;     /* requested events to watch　需要关注的请求事件 */
        short revents;    /* returned events witnessed 　返回所观察的时间　*/
};
```

每个pollfd结构体都关注一个文件描述符。

有效的事件类型：

POLLIN
There is data to read.
POLLRDNORM
There is normal data to read.
POLLRDBAND
There is priority data to read.
POLLPRI
There is urgent data to read.
POLLOUT
Writing will not block.
POLLWRNORM
Writing normal data will not block.

## sample

在真实应用中不需要在每次调用poll时，对pollfd结构体进行重构。

```c
#include <stdio.h>
#include <unistd.h>
#include <poll.h>

#define TIMEOUT 5       /* poll timeout, in seconds */

int main (void)
{
        struct pollfd fds[2];
        int ret;

        /* watch stdin for input */
        fds[0].fd = STDIN_FILENO;
        fds[0].events = POLLIN;

        /* watch stdout for ability to write (almost always true) */
        fds[1].fd = STDOUT_FILENO;
        fds[1].events = POLLOUT;

        /* All set, block! */
        ret = poll (fds, 2, TIMEOUT * 1000);
        if (ret == −1) {
                perror ("poll");
                return 1;
        }

        if (!ret) {
                printf ("%d seconds elapsed.\n", TIMEOUT);
                return 0;
        }

        if (fds[0].revents & POLLIN)
                printf ("stdin is readable\n");

        if (fds[1].revents & POLLOUT)
                printf ("stdout is writable\n");

        return 0;
}
```

运行

```bash
$ ./poll
stdout is writabl
```

再次运行：

```bash
$ ./poll < ode_to_my_parrot.txt
stdin is readable
stdout is writable
```
