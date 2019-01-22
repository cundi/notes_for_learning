# Event loop

1. asyncio应用的核心
2. 应用开发者通常使用高级asyncio函数，比如asyncio.run().
3. 本小节的函数目的在于编写底层库和代码，以及框架。或者是需要更加精细地控制事件循环行为。

实现功能：

- 注册、执行和取消调用
- 运行子进程，使用外部程序实现关联传输的通信
- 委派高损耗性能的函数到线程池

软件实例：

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

## 获取事件循环

获取、设置和创建事件循环的三个底层函数

获取：

- asyncio.get_running_loop()

- asyncio.get_event.loop()

设置：

- asyncio.set_event_loop(loop)

为当前的系统线程设置一个以`loop`作为参数的当前事件循环。

创建：

- asyncio.new_event_loop()

## 事件循环的API

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

## gevent

使用两种机制实现异步编程：

- 对标准库使用异步I/O函数。
- Greenlet对象实现并行执行。

Greenlet：一种可以当作线程使用但协程。

```python
```

