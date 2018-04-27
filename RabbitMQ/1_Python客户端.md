# 消息队列

基本概念：

- Broker：简单来说就是消息队列服务器实体
- Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列
- Queue：消息队列载体，每个消息都会被投入到一个或多个队列
- Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来
- Routing Key：路由关键字，exchange根据这个关键字进行消息投递
- vhost：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离
- producer：消息生产者，就是投递消息的程序
- consumer：消息消费者，就是接受消息的程序
- channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务

Python客户端：

- pika
- Celery(分布式任务队列)

## Python原生消息队列

两种消息队列：

- 线程 queue（同一进程下线程之间进行交互）
- 进程 Queue（父子进程进行交互 或者 同父下多个子进程进行交互）

缺陷：

两个互相独立的程序不能进行交互

示例：

1. 线程队列的实现

```python
#!/usr/bin/env python3
#coding:utf8

import queue
import threading


message = queue.Queue(10)


def producer(i):
    '''厨师,生产包子放入队列'''
    while True:
        message.put(i)
def consumer(i):
    '''消费者,从队列中取包子吃'''
    while True:
        msg = message.get()

for i in range(12): 厨师的线程包子
    t = threading.Thread(target=producer, args=(i,))
    t.start()
for i in range(10): 消费者的线程吃包子
    t = threading.Thread(target=consumer, args=(i,))
    t.start()
```

## rabbitmq的实现

### pika

#### 基本的生产消费者

当消费者接收来自生产者的数据时，消费者中断后小于10秒，再次连接的到生产者时数据依旧存在；若大于10秒之后，重新链接，数据将消失。消费者等待链接。

- 生产者（发送端）

```python
#!/usr/bin/env python
import pika
# ######################### 生产者 #########################
#链接rabbit服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
#创建频道
channel = connection.channel()
#创建一个队列名叫hello
channel.queue_declare(queue='hello')
#向队列插入数值 routing_key是队列名 body是要插入的内容
channel.basic_publish(exchange='',
                  routing_key='hello',
                  body='Hello World!')
print("开始队列")
connection.close()
```

- 消费者

```python
import pika
#链接rabbit
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
#创建频道
channel = connection.channel()
#如果生产者没有运行创建队列，那么消费者创建队列
channel.queue_declare(queue='hello')


def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
    import time
    time.sleep(10)
    print 'ok'
    ch.basic_ack(delivery_tag = method.delivery_tag) #主要使用此代码

channel.basic_consume(callback,
                      queue='hello',
                      no_ack=False)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

#### 队列持久化

特点：

1. RabbitMQ宕机时队列中的消息不丢失。
2. 应用了delivery_mode=2的消息持久化在写入磁盘磁盘时需要操作时间，如果此时宕机，消息将丢失。所以这里需要使用发布者确认模式。

- 生产者（发送端）

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
               'localhost',5672))  # 默认端口5672，可选
channel = connection.channel()
#声明queue,  durable=True表示队列持久化
channel.queue_declare(queue='hello2', durable=True)
#消息不能直接发往queue，总是需要通过exchange
# delivery_mode=2,  消息持久化
channel.basic_publish(exchange='',
                      routing_key='hello2',
                      body='Hello World!',
                      properties=pika.BasicProperties(
                          delivery_mode=2,
                          )
                      )

print(" [x] Sent 'Hello World!'")
connection.close()
```

查看当前queue数量及queue里消息数量：

```shell
rabbitmqctl list_queues
```

- 消费者（接收端）

```python
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(
               'localhost'))
channel = connection.channel()
channel.queue_declare(queue='hello2', durable=True)

def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
    time.sleep(10)
    ch.basic_ack(delivery_tag = method.delivery_tag)  # 告诉生产者，消息处理完成

channel.basic_qos(prefetch_count=1)  # 类似权重，按能力分发，如果有一个消息，就不在给你发
channel.basic_consume(  # 消费消息
                      callback,  # 如果收到消息，就调用callback
                      queue='hello2',
                      # no_ack=True  # 一般不写，处理完接收处理结果。宕机则发给其他消费者
                      )

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

消息的获取：

默认消息队列里的数据是按照顺序被消费者拿走，

### celery