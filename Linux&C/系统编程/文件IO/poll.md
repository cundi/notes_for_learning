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
        short events;     /* requested events to watch */
        short revents;    /* returned events witnessed */
};
```


## 