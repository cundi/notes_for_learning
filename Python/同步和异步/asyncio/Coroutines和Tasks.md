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

## Future

```python
asyncio.Future(*, loop=None)
```

- 该类几乎完全兼容于　`concurrent.futures.Future`
- 该类不是线程安全的

区别：

result() and exception() do not take a timeout argument and raise an exception when the future isn’t done yet.
Callbacks registered with add_done_callback() are always called via the event loop’s call_soon_threadsafe().
This class is not compatible with the wait() and as_completed() functions in the concurrent.futures package.

示例：

- 使用run_until_complete()方法的Future

```python
import asyncio

async def slow_operation(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')

loop = asyncio.get_event_loop()
future = asyncio.Future()
asyncio.ensure_future(slow_operation(future))
loop.run_until_complete(future)
print(future.result())
loop.close()
```

协程函数负责计算，并将结果存储在Future中，

- 使用run_forever()方法和Future

```python
import asyncio

async def slow_operation(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')

def got_result(future):
    print(future.result())
    loop.stop()

loop = asyncio.get_event_loop()
future = asyncio.Future()
asyncio.ensure_future(slow_operation(future))
future.add_done_callback(got_result)
try:
    loop.run_forever()
finally:
    loop.close()
```

## Task

```python
asyncio.Task(coro, *, loop=None)
```

- 调度协程的执行，即将协程包装到future中。
- Task是Future的子类

1. task负责执行事

## Awaitables


