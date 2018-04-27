# TCP Server and Client

地址族（family）: AF_INET (this is IP version 4 or IPv4)
类型（type）: SOCK_STREAM (this means connection oriented TCP protocol)

- 客户端:

```python
import sys
import socket

RECV_BYTES = 1024

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
except socket.error as e:
    print("Socket client create failed! Error: {0}".format(e))
    sys.exit()

print('Socket Created!')

s.connect(('127.0.0.1', 8080)) # 参数为元组
print(s.recv(RECV_BYTES).decode('utf-8'))
for data in range(0, 100, 10)
    s.send(data)
    s.sendall("GET / HTTP/1.1\r\n\r\n") # 发送整个字符串
    print(s.recv(RECV_BYTES).decode('utf-8'))
s.send(b'exit')
s.close()
```

- 服务端(使用线程)

```python
import socket
import threading

RECV_BYTES = 1024

try:
    s = socket.socket(family=socket.AF_INET, type=socket.SOCK_STREAM, proto=0)
except socket.error as msg:
    print("Failed to create socket, Error code: " + str(msg[0]) + \
    ', Error message: ' + msg[1])
    sys.exit()
print(s)
print('Socket created')
# socket服务终止时，确保快速重启,开发和测试时使用
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind(('127.0.0.1', 8080))
s.listen(5) # 最大连接数
print('Waiting for connection...')

def tcp_server(sock, addr):
    print("Accept new connection from {0}:{1}".format((addr[0], addr[1]))
    sock.send(b'Welcome !')
    try:
        while 1:
            data = sock.recv(RECV_BYTES)
            time.sleep(1)
            if not data or data.decode('uft-8') == 'exit':
                break
            sock.send("Hello Client, {0}".format(data.decode('uft-8').encode('uft-8')))
    finally:
        sock.close()

while 1:
    sock, addr = s.accept()
    t = threading.Thread(target=tcp_server, args=(sock, addr)) # 创建新线程，以接受多个客户端连接
    t.start()
```

输出结果为：

```shell
<socket.socket fd=53, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('0.0.0.0', 0)>
Socket created
```

- 服务端（面向对象写法）：

```python
import SocketServer

class MyHandler(SocketServer.BaseRequestHandler):
    def handle(self):
        while 1:
            dataReceived = self.request.recv(1024)
            if not dataReceived: break
            self.request.send(dataReceived)

myServer = SocketServer.TCPServer(('',8881), MyHandler)
myServer.serve_forever()
```

- 服务端（select）：

```python
import socket, select


ONNECTION_LIST = [] # socket客户端列表
RECV_BUFFER = 4096 # 使用2的指数值是个明智的选择
PORT = 5000

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# socket服务终止时，确保快速重启,开发和测试时使用
s.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind("0.0.0.0", PORT)
s.listen(10)

ONNECTION_LIST.append(s)
print("Chat server started on port "+ str(PORT))

while 1:
    # 从select中取得需要读取的socket列表
    read_sockets,write_sockets,error_sockets = select.select(ONNECTION_LIST, [], [])

    for sock in read_sockets:
        # 新连接
        if sock == s:
            # 处理通过s进入的连接
            sockfd, addr = s.accept()
            ONNECTION_LIST.append(sockfd)
            print("Client (%s, %s) connected" % addr)
        else:
            # 处理来自client数据
            try:
                data = sock.recv(RECV_BUFFER)
                # 给client返回响应消息
                if data:
                    sock.send("Ok..." + data)
            except:
                select.broadcast_data(sock, "Client (%s, %s) is offline" % addr)
                print "Client (%s, %s) is offline" % addr
                sock.close()
                CONNECTION_LIST.remove(sock)
                continue
server_socket.close()
```