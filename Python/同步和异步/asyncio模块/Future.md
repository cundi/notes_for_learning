# Future

用途：Future对象用来桥接 `低级的基于回调代码`和高级async/await代码。

## Future函数

```python
# New in version 3.5.
asyncio.isfuture(obj)
```

```python
asyncio.ensure_future(obj, *, loop=None)

"""
返回三种： 

1. 如果obj是Future，返回Future；Task对象，则返回`类Future` 对象
2. 如果obj是协程（通过asyncio.iscoroutine判断），则包装为
3. 如果obj是awaitable（通过inspect.isawaitable()判断），则

4. 如果obj不是以上三种类型，则抛出 TypeError。
"""
```

```python
asyncio.wrap_future(future, *, loop=None

"""
将concurrent.futures.Future对象包装为 asyncio.Future 对象
"""
```

## Future对象

```python
asyncio.Future(*, loop=None)

```

- Future表示最终的异步操作结果
- 该类几乎完全兼容于　`concurrent.futures.Future`
- 该类不是线程安全的
- Future是awaitable对象。协程可以等待Future对象
- 通常Future被用于低级的回调风格代码（比如，）

asyncio的Future对象设计模仿了`concurrent.futures.Future`，其区别在于：

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

### Future对象的方法

```python
result()

"""
"""
```

set_result(result)

set_exception(exception)

```python
done()
```

```python
cancelled()

"""
"""
```

```python
add_done_callbakc(callback, *, context=None)

"""

"""
```