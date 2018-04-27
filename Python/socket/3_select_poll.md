# 进程通信底层

`http://www.cnblogs.com/coser/archive/2012/01/06/2315216.html`

`http://www.cnblogs.com/alex3714/p/4372426.html`

- 水平触发（Level Triggered）：select()和poll()将就绪的文件描述符告诉进程后，如果进程没有对其进行IO操作，那么下次调用select()和poll()的时候将再次报告这些文件描述符，所以它们一般不会丢失就绪的消息。

- 边缘触发（Edge Triggered）：只告诉进程哪些文件描述符刚刚变为就绪状态，它只说一遍，如果我们没有采取行动，那么它将不会再次告知。

## select

优点：良好跨平台支持
缺点：单个进程能所监视的文件描述符的数量存在最大限制，在Linux上一般为1024。

## poll

特点：没有最大文件描述符数量限制。

缺点：包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。

## epoll

特点：epoll可以同时支持水平触发和边缘触发。

和select/poll的区别：

- elect/poll进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描
- 而epoll事先通过epoll_ctl()来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait()时便得到通知

## Python中使用select

select函数特性：

- 对底层操作系统的直接访问的接口。

select函数用途：

- 监控sockets、files和pipes，等待IO完成（Waiting for I/O completion）。
- 监控可读、可写或是异常事件的产生。

- Server端：

```python
import select
import socket
import Queue

#create a socket
server = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
server.setblocking(False)
#set option reused
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR  , 1)

server_address= ('127.0.0.1',10001)
server.bind(server_address)

server.listen(10)

#sockets from which we except to read
inputs = [server]

#sockets from which we expect to write
outputs = []
 
#Outgoing message queues (socket:Queue)
message_queues = {}
 
#A optional parameter for select is TIMEOUT
timeout = 20

while inputs:
    print "waiting for next event"
    readable , writable , exceptional = select.select(inputs, outputs, inputs, timeout)

    # When timeout reached , select return three empty lists
    if not (readable or writable or exceptional) :
        print "Time out ! "
        break;
    for s in readable :
        if s is server:
            # A "readable" socket is ready to accept a connection
            connection, client_address = s.accept()
            print "    connection from ", client_address
            connection.setblocking(0)
            inputs.append(connection)
            message_queues[connection] = Queue.Queue()
        else:
            data = s.recv(1024)
            if data :
                print " received " , data , "from ",s.getpeername()
                message_queues[s].put(data)
                # Add output channel for response
                if s not in outputs:
                    outputs.append(s)
            else:
                #Interpret empty result as closed connection
                print "  closing", client_address
                if s in outputs :
                    outputs.remove(s)
                inputs.remove(s)
                s.close()
                #remove message queue 
                del message_queues[s]
    for s in writable:
        try:
            next_msg = message_queues[s].get_nowait()
        except Queue.Empty:
            print " " , s.getpeername() , 'queue empty'
            outputs.remove(s)
        else:
            print " sending " , next_msg , " to ", s.getpeername()
            s.send(next_msg)
    for s in exceptional:
        print " exception condition on ", s.getpeername()
        #stop listening for input on the connection
        inputs.remove(s)
        if s in outputs:
            outputs.remove(s)
        s.close()
        #Remove message queue
        del message_queues[s]
```

- Client端：

```python
import socket

messages = ["This is the message" ,
            "It will be sent" ,
            "in parts "]

print("Connect to the server")

server_address = ("192.168.1.102",10001)
#Create a TCP/IP sock
socks = []
for i in range(10):
    socks.append(socket.socket(socket.AF_INET,socket.SOCK_STREAM))
for s in socks:
    s.connect(server_address)
counter = 0
for message in messages :
    #Sending message from different sockets
    for s in socks:
        counter+=1
        print "  %s sending %s" % (s.getpeername(),message+" version "+str(counter))
        s.send(message+" version "+str(counter))
    #Read responses on both sockets
    for s in socks:
        data = s.recv(1024)
        print " %s received %s" % (s.getpeername(),data)
        if not data:
            print "closing socket ",s.getpeername()
            s.close()
```