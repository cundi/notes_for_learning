# Future

## Future函数

## Future对象

```python
asyncio.Future(*, loop=None)
```

- Future表示最终的异步操作结果
- 该类几乎完全兼容于　`concurrent.futures.Future`
- 该类不是线程安全的
- Future是awaitable对象。协程可以等待Future对象
- 通常Future被用于低级的回调风格代码（比如，）

Ｆuture对象的设计模仿了`concurrent.futures.Future`，其区别在于：

1. 和asyncio的Future不同，`concurrent.futures.Future`的实例不可以用在`await`表达式上。
2. `asyncio.Future.result()`和`asyncio.Future.exception()`不接受`timeout`参数。
3. 在Future没有完成时，`asyncio.Future.result()`和`asyncio.Future.exception()`抛出一个`InvalidStateError`的异常。
4. 使用了`asyncio.Future.add_done_callback()`进行注册的回调不会立即调用。相反，注册的回调会在`loop.call_soon()`中按计划执行。
5. asyncio的Future不兼容于`concurrent.futures.wait()`和`concurrent.futures.as_completed()`。

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