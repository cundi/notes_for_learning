# Event loop

>引用
>`https://docs.python.org/3.7/library/asyncio-eventloop.html#asyncio-event-loop`

1. asyncio应用的核心
2. 应用开发者通常使用高级asyncio函数，比如asyncio.run().
3. 本小节的函数目的在于编写底层库和代码，以及框架。或者是需要更加精细地控制事件循环行为。

实现功能：

- 注册、执行和取消调用
- 运行子进程，使用外部程序实现关联传输的通信
- 委派高损耗性能的函数到线程池

## 获取事件循环

获取、设置和创建事件循环的三个底层函数

获取：

- asyncio.get_running_loop()

- asyncio.get_event_loop()

设置：

- asyncio.set_event_loop(loop)

为当前的系统线程设置一个以`loop`作为参数的当前事件循环。

创建：

- asyncio.new_event_loop()

## 事件循环的API

>这里的`loop`为调用asyncio.get_event_loop()之后获取的asyncio.AbstractEventLoop实例

- 运行和停止循环

```python
loop.run_until_complete(future)
# 1. 运行直到Future实例实现为止
# 2. 如果该函数的参数为协程对象，则隐式地计划使用asyncio.Task运行。
# 3. 返回Future的结果，或者抛出异常
```

```python
loop.run_forever()
```

- 计划回调

- 计划延迟回调

- 创建Future和Task

```python
loop.create_future()
```

- 打开网络连接

```python
# 该函数返回对象为协程

loop.create_connection(protocol_factory, host=None, port=None, *, ssl=None, family=0, proto=0, flags=0, sock=None, local_addr=None, server_hostname=None, ssl_handshake_timeout=None)

# 对指定了的主机和端口打开一个数据流传输连接
```

- 创建网络服务器

```python
# 该函数返回 Server 对象

loop.create_server(protocol_factory, host=None, port=None, *, family=socket.AF_UNSPEC, flags=socket.AI_PASSIVE, sock=None, backlog=100, ssl=None, reuse_address=None, reuse_port=None, ssl_handshake_timeout=None, start_serving=True)

# 新建一个TCP服务，

# 参数：
# protocol_factory: 必须是一个可
```

```python
loop.create_unix_server(protocol_factory, path=None, *, sock=None, backlog=100, ssl=None, ssl_handshake_timeout=None, start_serving=True)
```

```python
loop.connect_accepted_socket(protocol_factory, sock, *, ssl=None, ssl_handshake_timeout=None)
```

- 传输文件

```python
loop.sendfile(transport, file, offset=0, count=None, *, fallback=True)

# 使用transport发送｀file`。返回发送的字节总数．
# 如果可能的话，该方法使用高性能的`os.sendfile()`
# `file`参数必须是二进制模式打开的普通文件对象
```

- TLS升级

```python
loop.start_tls(transport, protocol, sslcontext, *, server_side=False, server_hostname=None, ssl_handshake_timeout=None)
```

- 照看文件描述符

```python
loop.add_reader(fd, callback, *args)
```

- 直接使用socket对象

一般情况下，使用基于transport的API `loop.create_connection`和`loop.create_server()`比直接使用socket更快。

应用场景：在性能不太重要的情况下，直接用socket对象更方便。

```python
loop.sock_recv(sock, nbytes)

"""
:sock: socket对象
:nbytes: 整数，可接受的字节数量。

１．socket.recv()的异步版本
２．将收到数据以字节对象形式返回
"""
```

- DNS

从Python 3.7开始，这两个方法返回都是协程。

```python
loop.getaddrinfo(host, port, *, family=0, type=0, proto=0, flags=0)

"""

- socket.getaddrinfo()的异步版本
"""
```

```python
loop.getnameinfo(sockaddr, flags=0)

"""
- socket.getnameinfo()的异步版本
"""
```

- 配合pipe的使用

```python
loop.connect_read_pipe(protocol_factory, pipe)

"""
:protocol_factory:
:pipe: 类文件对象
"""
```

```python
loop.connect_write_pipe(protocol_factory, pipe)
```

- Unix信号

```python
loop.add_signal_handler(signum, callback, *args)

"""

"""
```

- 在线程/进程池中执行代码

```python
loop.run_in_executor(executor, func, *args)

"""
:executor: concurrent.futures.Executor的实例
:func: 在指定executor中将要被执行的函数
:return: awaitable，asyncio.Future对象
"""
```

```python
import asyncio
import concurrent.futures

def blocking_io():
    # File operations (such as logging) can block the
    # event loop: run them in a thread pool.
    with open('/dev/urandom', 'rb') as f:
        return f.read(100)

def cpu_bound():
    # CPU-bound operations will block the event loop:
    # in general it is preferable to run them in a
    # process pool.
    return sum(i * i for i in range(10 ** 7))


async def main():
    loop = asyncio.get_running_loop()

    ## Options:

    # 1. Run in the default loop's executor:
    result = await loop.run_in_executor(
        None, blocking_io)
    print('default thread pool', result)

    # 2. 运行在自定义线程池中:
    with concurrent.futures.ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(
            pool, blocking_io)
        print('custom thread pool', result)

    # 3. 运行在自定进程池中:
    with concurrent.futures.ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(
            pool, cpu_bound)
        print('custom process pool', result)

asyncio.run(main())
```

```python
loop.set_default_executor(executor)

"""

"""
```

- 错误处理API

- 启用调试模式

- 运行子进程

本段的方法都是比较底层的。在使用了async/await的代码中请考虑使用高级的、更为方便的`asyncio.create_subprocess_shell()`和`asyncio.create_subprocess_exec()`。

>### 注释
>默认的asyncio事件循环不支持Windows系统中使用子进程

```python
loop.subprocess_exec(protocol_factory, *args, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, **kwargs)

"""

:return: 协程
"""
```

```python
loop.subprocess_shell(protocol_factory, cmd, *, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, **kwargs)

"""
"""
```

- 多个Future的组合，放到一个事件循环中

```python
futus = [asyncio.ensure_future(do_some_work(1)),
             asyncio.ensure_future(do_some_work(3))]

loop.run_until_complete(asyncio.gather(*futus))
```

- 在一个事件循环中运行多个协程

```python
loop.run_until_complete(asyncio.gather(do_some_work(1), do_some_work(3)))
```

## 回调处理

## Server对象

- Server对象可以使用函数　loop.create_server(), loop.create_unix_server(), start_server(), and start_unix_server() 创建，不要直接初始化asyncio.Server。
- Server对象是异步的上下文管理器。
- 在`async with`语句完成后，它能够保证server对象的关闭并再接收新的连接。

```python
srv = await loop.create_server(...)

async with srv:
    # some code

# At this point, srv is closed and no longer accepts new connections.
```

类`asyncio.Server`支持以下方法：

```python
close()

get_loop()

start_serving()
```

```python
serve_forever()
"""
:return: 协程对象

１．开始接受连接直到协程被结束。
２．每个Server对象仅可以运行一次`serve_forever`
"""

async def client_connected(reader, writer):
    # Communicate with the client with
    # reader/writer streams.  For example:
    await reader.readline()

async def main(host, port):
    srv = await asyncio.start_server(
        client_connected, host, port)
    await srv.serve_forever()

asyncio.run(main('127.0.0.1', 0))
```

```python
is_serving()
```

```python
wait_closed()
```

```python
sockets()
```

## 事件循环的实现

asyncio模块有两种事件循环的实现，即类`asyncio.SelectorEventLoop`和`asyncio.ProactorEventLoop`。
默认在所有操作系统上使用SelectorEventLoop。

- asyncio.SelectorEventLoop：

    基于`selectors`模块的事件循环
    可以针对不同的操作系统选择使用最适合的selector
    适用OS: Unix,Windows

    ```python
    import asyncio
    import selectors

    selector = selectors.SelectSelector()
    loop = asyncio.SelectorEventLoop(selector)
    asyncio.set_event_loop(loop)
    ```

- asyncio.ProactorEventLoop

    Windows系统上使用“I/O完成端口”的事件循环
    适用OS: Windows

    ```python
    import asyncio
    import sys

    if sys.platform == 'win32':
        loop = asyncio.ProactorEventLoop()
        asyncio.set_event_loop(loop)
    ```

- asyncio.AbstractEventLoop

    服务asyncio事件循环的抽象基类

## 示例

```python
import asyncio

def hello_world(loop):
    print('Hello World')
    loop.stop()

loop = asyncio.get_event_loop()

# Schedule a call to hello_world()
loop.call_soon(hello_world, loop)

# Blocking call interrupted by loop.stop()
loop.run_forever()
loop.close()
```

## 补充示例

- 软件实例：

web服务器等待进入的url请求，然后匹配该请求所关联的页面，这个过程就是一个独立的事件。

```python
from Queue import Queue
from functools import partial


eventloop = None


class EventLoop(Queue):
    def start(self):
        while True:
            function = self.get()
            function()


def do_hello():
    global eventloop
    print("hello")
    eventloop.put(do_world)


def do_world():
    print("World")
    eventloop.put(do_hello)


if __name__ == "__main__":
    eventloop = EventLoop()
    eventloop.put(do_hello)
    eventloop.start()
```