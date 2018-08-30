# celery

主要组件: 
producer: 任务发布者, 通过调用API向celery发布任务的程序 
celery beat: 任务调度, 根据配置文件发布定时任务 
worker: 实际执行任务的程序,接收任务生产者发来的消息（即任务），将任务存入队列。Celery 本身不提供队列服务，官方推荐使用 RabbitMQ 和 Redis 等。
broker: 接受任务消息,存入队列再按顺序分发给worker执行, 执行任务的处理单元，它实时监控消息队列，获取队列中调度的任务，并执行它。
backend: 存储结果的服务器, 存储任务的执行结果，以供查询。同消息中间件一样，存储也可使用 RabbitMQ, Redis 和 MongoDB 等。

![](Python/thirdLibrary/images/celery_data_flow.jpg)

使用 Celery 实现异步任务主要包含三个步骤：

- 创建一个 Celery 实例

- 启动 Celery Worker

- 应用程序调用异步任务


## 安装

``shell
pip install -U Celery
``

安装所有功能包

```shell
pip install "celery[librabbitmq,redis,auth,msgpack]"
```

## 生产与消费

### 相同主机

### 不同主机

On Machine A:

Install Celery & RabbitMQ.
Configure rabbitmq so that Machine B can connect to it.
Create my_tasks.py with some tasks and put some tasks in queue.

On Machine B:

Install Celery.
Copy my_tasks.py file from machine A to this machine.
Run a worker to consume the tasks


## call remote task

a key point is invoke `send_task` method.

A --> B

Host A:

```python
# file name
from celery import Celery
import time

broker = 'redis://1.1.1.1:6379'
backend = 'redis://1.1.1.1:6379'


app = Celery('tasks', broker=broker, backend=backend, task_serializer='pickle')
app.conf.update(CELERY_ACCEPT_CONTENT = ['pickle'], CELERY_RESULT_SERIALIZER = 'pickle')

@app.task(serializer="pickle")
def add(x, y):
    return x + y


result = app.send_task('task.add', [1,2])
time.sleep(1)
print(result.ready())
print(result.result)
```

Host B:

```python
# file name task.py
from celery import Celery
import time

broker = 'redis://1.1.1.1:6379'
backend = 'redis://1.1.1.1:6379'


app = Celery('tasks', broker=broker, backend=backend, task_serializer='pickle')
app.conf.update(CELERY_ACCEPT_CONTENT = ['pickle'], CELERY_RESULT_SERIALIZER = 'pickle')

@app.task(serializer="pickle")
def add(x, y):
    return x + y
```

start woker on Host B


```shell
celery -A task worker  --loglevel=info
```

run task.py at Host A, ouput as following

```
True
3
```