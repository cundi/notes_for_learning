# Coroutines and Tasks

引用：`https://docs.python.org/3/library/asyncio-task.html#awaitables`

## Coroutines

协程的部分详情参考PEP492。

asyncio提供了三种运行协程的主要机制：

1. `asyncio.run()`函数运行顶层入口函数（如上）
2. 对协程对象使用await
3. 

协程声明示例：

```python
# Python 3.4 coroutine example
import asyncio

@asyncio.coroutine
def my_coro():
    yield from func()
```

```python
# 3.5
import asyncio

async def hello_world():
    print("Hello World!")

loop = asyncio.get_event_loop()
# Blocking call which returns when the hello_world() coroutine is done
loop.run_until_complete(hello_world())
loop.close()
```

```python
# 3.5
import asyncio
import datetime

async def display_date(loop):
    end_time = loop.time() + 5.0
    while True:
        print(datetime.datetime.now())
        if (loop.time() + 1.0) >= end_time:
            break
        await asyncio.sleep(1)

loop = asyncio.get_event_loop()
# Blocking call which returns when the display_date() coroutine is done
loop.run_until_complete(display_date(loop))
loop.close()
```

示例-1， 运行协程的第一种机制：

```python
# 3.7+
>>> import asyncio

>>> async def main():
...     print('hello')
...     await asyncio.sleep(1)
...     print('world')

>>> asyncio.run(main())
hello
world
```

示例-2，对协程对象使用await关键字

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    await say_after(1, 'hello')
    await say_after(2, 'world')

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

示例-3，asyncio.create_task() 函数并发执行协程：

```python
async def main():
    task1 = asyncio.create_task(
        say_after(1, 'hello'))

    task2 = asyncio.create_task(
        say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    # Wait until both tasks are completed (should take
    # around 2 seconds.)
    await task1
    await task2

    print(f"finished at {time.strftime('%X')}")
```

- Future

`Futuer`是一种特殊的`底层的`awaitable对象，它代表了一次异步执行的`事件结果`。



```python
# 链式调用协程
# print_sum()在返回结果之前，会一直处于等待状态，直到compute()完成为止。
import asyncio

async def compute(x, y):
    print("Compute %s + %s ..." % (x, y))
    await asyncio.sleep(1.0)
    return x + y

async def print_sum(x, y):
    result = await compute(x, y)
    print("%s + %s = %s" % (x, y, result))

loop = asyncio.get_event_loop()
loop.run_until_complete(print_sum(1, 2))
loop.close()
```

## 可await（awaitable）

定义：一个对象可以用于await表达式就是awaitable。

三种awaitable对象：

1. 协程（coroutine）
2. Task
3. Future

- 协程

可以在一个协程中await另外一个协程。

两个重要且相近的关联概念：

- 协程函数：使用了 `async def`的函数；
- 协程对象：调用协程函数所返回的对象。

```python
import asyncio

async def nested():
    return 42

async def main():
    # Nothing happens if we just call "nested()".
    # A coroutine object is created but not awaited,
    # so it *won't run at all*.
    nested()

    # Let's do it differently now and await it:
    print(await nested())  # will print "42".

asyncio.run(main())
```

- Task

用途：并发地按计划执行协程。

用法：将函数使用asyncio.create_task()包装为`Task`，然后协程就会按照计划执行

```python
import asyncio

async def nested():
    return 42

async def main():
    # Schedule nested() to run soon concurrently
    # with "main()".
    task = asyncio.create_task(nested())

    # "task" can now be used to cancel "nested()", or
    # can simply be awaited to wait until it is complete:
    await task

asyncio.run(main())
```

## 模块函数

### Running an asyncio Program

```python
# Python 3.7新加入
asyncio.run(coro, *, debug=False)

"""
1. 运行传递的协程
2. 该函数在同一个线程中的事件循环运行时无法调用
3. debug为True，则启用事件循环的调试模块
4. 该函数总是创建一个新的事件循环，并在最后关闭它
5. 应该用在异步程序的主入口，理论上也只应该调用一次
"""
```

### Creating Tasks

```python
asyncio.create_task(coro)

"""
1. 将协程对象包装为Task对象，并对其做计划执行。
2. 返回Task对象。
3. get_running_loop()返回事件循环中所执行的task
4. 当前线程中如果没有正在运行的循环，则抛出RuntimeError 
5. Python 3.7 新加入，之前的版本可以使用底层的asyncio.ensure_future()作为替代。
"""
```

### Sleeping

```python
```

### 并发地运行Task

```python
```

### 保护被取消

```python
```

### 超时

```python
```

### 初始化时的等待（Waiting Primitives）

```python
coroutine asyncio.wait(aws, *, loop=None, timeout=None, return_when=ALL_COMPLETED)

"""
:param: aws: awaitable对象。
:param: loop: Python 3.10版本中将会移除。

"""

```

### 从其他线程中做计划执行

```python
asyncio.run_coroutine_threadsafe(coro, loop)

"""
1. 将协程提交到指定的事件循环中。
2. 该函数是线程安全的。
3. 返回  concurrent.futures.Future 以等待来自操作系统中其他线程的结果
4. 该函数表明了，它是被操作系统中的其他线程所调用，而不是正在运行事件循环的那个线程。
"""

# 创建一个协程
coro = asyncio.sleep(1, result=3)

# Submit the coroutine to a given loop
future = asyncio.run_coroutine_threadsafe(coro, loop)

# Wait for the result with an optional timeout argument
assert future.result(timeout) == 3
```

## Task 对象

```python
asyncio.Task(coro, *, loop=None)
```

- 调度协程的执行，即将协程包装到future中。
- Task是Future的子类
- Task类不是线程安全的

1. 在事件循环中task负责执行协程对象。
2. 如果被包装的协程生成自future对象，那么task会暂停所包装协程的执行，然后等待future的完成。
3. 在future完成后，被包装协程的执行会带着future结果或者异常进行重启。

task的协作调度：

    一次事件循环中仅可以运行一个task。
    如果另外的事件循环运行在其他线程中，那么剩余的task可以并行的运行。
    在task等待future的完成时，事件循环会执行一个新的task。

task的取消：

    task的取消不同于future的取消。
    调用`cancel()`方法会抛出一个被包装协程的`CancelledError`。
    如果包装的协程没有捕获、或者抛出`CancelledError`异常，c那么`cancelled()`仅返回`True`。

task的销毁：

    如果挂起的task被销毁，那么它所包装的协程执行则不会完成。

task的创建：

    不要直接用Task类的实例，应该使用函数`ensure_future()`或者方法`AbstractEventLoop.create_task()`。

### 并行task示例

并行的执行`A、B、C`三个task

```python
import asyncio

async def factorial(name, number):
    f = 1
    for i in range(2, number+1):
        print("Task %s: Compute factorial(%s)..." % (name, i))
        await asyncio.sleep(1)
        f *= i
    print("Task %s: factorial(%s) = %s" % (name, number, f))

loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.gather(
    factorial("A", 2),
    factorial("B", 3),
    factorial("C", 4),
))
loop.close()
```

对应的输出：

```python
Task A: Compute factorial(2)...
Task B: Compute factorial(2)...
Task C: Compute factorial(2)...
Task A: factorial(2) = 2
Task B: Compute factorial(3)...
Task C: Compute factorial(3)...
Task B: factorial(3) = 6
Task C: Compute factorial(4)...
Task C: factorial(4) = 24
```



## Awaitables


