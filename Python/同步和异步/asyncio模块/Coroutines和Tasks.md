# Coroutines and Tasks

## Coroutines

示例：

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

## Task

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


