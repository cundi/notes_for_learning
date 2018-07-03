# 上下文管理器综述

定义：上下文管理器是一个可以包裹任意代码块的对象。

实现过程：

使用上下文管理器的理由：

- 不使用上下文管理器：

```python
try:
    f = open('/path/to/file', 'r')
    content = f.read()
finally:
    f.close()
```

- 使用了上下文管理器

```python
with open('/path/to/file', 'r') as f:
    content = f.read()
```