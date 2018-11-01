# 列表解析式

语法格式：

- 单个可迭代对象 [ expression for target in iterable ]
- 多个可迭代对象 [ expression for target1 in iterable1 if condition1
             for target2 in iterable2 if condition2 ...
             for targetN in iterableN if conditionN ]

## 解析式中的for循环

- 单层for循环

```shell
>>> [x ** 2 for x in range(10)]
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

>>> list(map((lambda x: x ** 2), range(10)))
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

- 两层for循环

```shell
>>> res = [x + y for x in [0, 1, 2] for y in [100, 200, 300]]
>>> res
[100, 200, 300, 101, 201, 301, 102, 202, 302]
```

## 解析式中使用 if

```shell
>>> [x for x in range(5) if x % 2 == 0]
[0, 2, 4]

>>> list(filter((lambda x: x % 2 == 0), range(5)))
[0, 2, 4]
```

### 等价于filter和map联合使用的例子

```shell
>>> [x ** 2 for x in range(10) if x % 2 == 0]
[0, 4, 16, 36, 64]
>>> list( map((lambda x: x**2), filter((lambda x: x % 2 == 0), range(10))) )
[0, 4, 16, 36, 64]
```

## 嵌套匿名函数

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