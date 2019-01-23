# 传输和协议

##　概述：

- Transports and Protocols 被用于时间循环API的底层，比如，loop.create_connection()
- 使用回调风格编程，能够实现的高性能的网络或者IPC协议（比如，HTTP）。
- 只用于库和框架，不应该出现在高层的asyncio应用中

## 介绍

- 在最高层面，transport只关心如何（how）传输字节，而protocol确定要传输多少(which)字节。
- 殊途同归：transport是对socket的抽象，而从传输的视角来看，protocol则是对应用的抽象。
- 从另外一个角度来说，transport 和 protocol 共同定义了调用网络I/O和内部进程I/O的抽象接口。
- transport 和 protocol对象之间存在这１对１的协作关系：protocol调用transport方法发送数据，而transport调用protocol方法去传递收到的数据。

## 示例

服务端：使用loop.create_server()方法创建一个　TCP　服务，发送收到数据，然后关闭连接。

```python
import asyncio


class EchoServerProtocol(asyncio.Protocol):
    def connection_made(self, transport):
        peername = transport.get_extra_info('peername')
        print('Connection from {}'.format(peername))
        self.transport = transport

    def data_received(self, data):
        message = data.decode()
        print('Data received: {!r}'.format(message))

        print('Send: {!r}'.format(message))
        self.transport.write(data)

        print('Close the client socket')
        self.transport.close()


async def main():
    # Get a reference to the event loop as we plan to use
    # low-level APIs.
    loop = asyncio.get_running_loop()

    server = await loop.create_server(
        lambda: EchoServerProtocol(),
        '127.0.0.1', 8888)

    async with server:
        await server.serve_forever()


asyncio.run(main())
```

客户端：使用`ｌoop.create_connection()`方法发送数据，然后等待回复，知道连接关闭。

```python
import asyncio


class EchoClientProtocol(asyncio.Protocol):
    def __init__(self, message, on_con_lost, loop):
        self.message = message
        self.loop = loop
        self.on_con_lost = on_con_lost

    def connection_made(self, transport):
        transport.write(self.message.encode())
        print('Data sent: {!r}'.format(self.message))

    def data_received(self, data):
        print('Data received: {!r}'.format(data.decode()))

    def connection_lost(self, exc):
        print('The server closed the connection')
        self.on_con_lost.set_result(True)


async def main():
    # Get a reference to the event loop as we plan to use
    # low-level APIs.
    loop = asyncio.get_running_loop()

    on_con_lost = loop.create_future()
    message = 'Hello World!'

    transport, protocol = await loop.create_connection(
        lambda: EchoClientProtocol(message, on_con_lost, loop),
        '127.0.0.1', 8888)

    # Wait until the protocol signals that the connection
    # is lost and close the transport.
    try:
        await on_con_lost
    finally:
        transport.close()


asyncio.run(main())
```