# logging 模块

两个目的：

- 调试：记录应用的相关操作事件。例如，
- 审计：为业务分析而记录事件。

最佳实践：初始化日志记录器时只使用`__name__`全局变量创建。理由是，the logging module creates a hierarchy of loggers using dot notation, so using __name__ ensures no name collisions.

```python
# __init__.py
import logging
logging.getLogger(__name__).addHandler(logging.NullHandler())
```


示例：通过字典实现配置

```python
import logging
from logging.config import dictConfig

logging_config = dict(
    version = 1,
    formatters = {
        'f': {'format':
              '%(asctime)s %(name)-12s %(levelname)-8s %(message)s'}
        },
    handlers = {
        'h': {'class': 'logging.StreamHandler',
              'formatter': 'f',
              'level': logging.DEBUG}
        },
    root = {
        'handlers': ['h'],
        'level': logging.DEBUG,
        },
)

dictConfig(logging_config)

logger = logging.getLogger()
logger.debug('often makes a very good meal of %s', 'visiting tourists')
```

示例：在代码中直接配置

```python
import logging

logger = logging.getLogger()
handler = logging.StreamHandler()
formatter = logging.Formatter(
        '%(asctime)s %(name)-12s %(levelname)-8s %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.debug('often makes a very good meal of %s', 'visiting tourists')
```