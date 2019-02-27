# 常见错误

## RuntimeError: Cannot run the event loop while another loop is running

The kernel itself runs on an event loop, and as of Tornado 5.0, it's using the asyncio event loop. So the asyncio event loop is always running in the kernel. As far as I know, we haven't figured out a way to deal with this yet.

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