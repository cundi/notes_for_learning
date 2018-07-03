# 列表解析式

- 列表解析式中的嵌套匿名函数

```python
>>> bases = [(lambda i: lambda x: x**i)(i) for i in range(3)]
>>> print([b(5) for b in bases])
[1, 5, 25]
```

同等效果替代

```python
>>> import functools
>>> bases = [functools.partial(lambda i,x: x**i,i) for i in range(3)]
>>> print([b(5) for b in bases])
[1, 5, 25]
```

```python
In [177]: bases = [lambda x, i=i: x**i for i in range(3)]

In [178]: print([b(5) for b in bases])
[1, 5, 25]
```