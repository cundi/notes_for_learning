# 常见错误

## RuntimeError: Cannot run the event loop while another loop is running

回溯信息如下：

```shell
File "/home/ml/Documents/dev/dp_admin/awx/client.py", line 22, in __init__
    self.client = httpclient.HTTPClient()
  File "/home/ml/.local/share/virtualenvs/dp_admin-b0v6X0T1/lib/python3.6/site-packages/tornado/httpclient.py", line 95, in __init__
    gen.coroutine(lambda: async_client_class(**kwargs)))
  File "/home/ml/.local/share/virtualenvs/dp_admin-b0v6X0T1/lib/python3.6/site-packages/tornado/ioloop.py", line 571, in run_sync
    self.start()
  File "/home/ml/.local/share/virtualenvs/dp_admin-b0v6X0T1/lib/python3.6/site-packages/tornado/platform/asyncio.py", line 132, in start
    self.asyncio_loop.run_forever()
  File "/usr/lib/python3.6/asyncio/base_events.py", line 412, in run_forever
    'Cannot run the event loop while another loop is running')
```

The kernel itself runs on an event loop, and as of Tornado 5.0, it's using the asyncio event loop. So the asyncio event loop is always running in the kernel. As far as I know, we haven't figured out a way to deal with this yet.

"On Python 3, IOLoop is always a wrapper around the asyncio event loop."

as listed in "Backwards-compatibility notes" of tornado 5.0: `http://www.tornadoweb.org/en/stable/releases/v5.0.0.html#backwards-compatibility-notes`

从`tornado.httpclient.HTTPClient`类的注释从可以看到，有如下变更。

```ｐython
    .. versionchanged:: 5.0

       Due to limitations in `asyncio`, it is no longer possible to
       use the synchronous ``HTTPClient`` while an `.IOLoop` is running.
       Use `AsyncHTTPClient` instead.
```

自5.0版本后，事件循环用的便是标准库的asyncio模块中的Loop对象。
且`HTTPClient`类初始化时默认不指定`async_client_class`时，使用的就是另外一个异步HTTP客户端类`AsyncHTTPClient`。
所以`HTTPClient`不再兼容于Python3.5的环境下使用。

```python
    def __init__(self, async_client_class=None, **kwargs):
        # Initialize self._closed at the beginning of the constructor
        # so that an exception raised here doesn't lead to confusing
        # failures in __del__.
        self._closed = True
        self._io_loop = IOLoop(make_current=False)
        if async_client_class is None:
            async_client_class = AsyncHTTPClient
        # Create the client while our IOLoop is "current", without
        # clobbering the thread's real current IOLoop (if any).
        self._async_client = self._io_loop.run_sync(
            gen.coroutine(lambda: async_client_class(**kwargs)))
        self._closed = False
```

## RuntimeError: This event loop is already running

引用：`https://github.com/ipython/ipython/issues/11030`

错误大抵如此：

```shell
 File "/home/ml/Documents/dev/dp_admin/awx/client.py", line 148, in get_result
    rt = json.loads(loop.run_until_complete(func))
  File "/usr/lib/python3.6/asyncio/base_events.py", line 455, in run_until_complete
    self.run_forever()
  File "/usr/lib/python3.6/asyncio/base_events.py", line 409, in run_forever
    raise RuntimeError('This event loop is already running')
RuntimeError: This event loop is already running
```

其中一个解决办法是将事件循环放到另外一个线程中运行， 代码示例如下：

```python
aio_pool = ThreadPoolExecutor(1)

def init_loop():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    return loop

aio_loop = aio_pool.submit(init_loop).result()

async def mycoro():
    await asyncio.sleep(1)
    return 5

result = aio_pool.submit(lambda : aio_loop.run_until_complete(mycoro())).result()
```

第二个解决方案：

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor


LOOP = asyncio.new_event_loop()
ThreadPoolExecutor().submit(LOOP.run_forever)

asyncio.run_coroutine_threadsafe(coro, LOOP).result()
```