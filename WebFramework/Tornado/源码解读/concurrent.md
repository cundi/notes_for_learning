# cocurrent

源码地址：`https://github.com/tornadoweb/tornado/blob/master/tornado/concurrent.py`

模块引用:

```python
import asyncio
from concurrent import futures
import functools
import sys
import types

from tornado.log import app_log

import typing
from typing import Any, Callable, Optional, Tuple, Union
```

```python
_T = typing.TypeVar("_T")
```

处理future对象：

```python
Future = asyncio.Future

FUTURES = (futures.Future, Future)


def is_future(x: Any) -> bool:
    return isinstance(x, FUTURES)
```

```python
class DummyExecutor(futures.Executor):
    def submit(
        self, fn: Callable[..., _T], *args: Any, **kwargs: Any
    ) -> "futures.Future[_T]":
        future = futures.Future()  # type: futures.Future[_T]
        try:
            future_set_result_unless_cancelled(future, fn(*args, **kwargs))
        except Exception:
            future_set_exc_info(future, sys.exc_info())
        return future

    def shutdown(self, wait: bool = True) -> None:
        pass


dummy_executor = DummyExecutor()
```