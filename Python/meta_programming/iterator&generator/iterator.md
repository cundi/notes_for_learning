# Iterator

定义：实现了迭代器协议的容器对象，该协议包含了`__next__`,`__iter__`两个方法。

`__next__`: 

`__iter__`:

## 解析式

通过迭代器进行迭代更有效率，因为它们用原生C实现，比for循环更快。

三种解析式：

- 列表解析式
- 字典解析式
- 集合解析式

示例：

```shell
>>> l = [ i+1 for i in range(10)]
>>> print(type(l), l)
<class 'list'> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

## itertools 模块

Python内建函数map,reduce,filter,zip的生成器版本分别是itertools模块中但imap,ireduce,ifilter,izip。

- itertools.islice():

对潜在但无限生成器进行切片。

- itertools.takewhile():

对生成器添加一个结束条件。

- itertools.chain(*iterable)：

从可迭代对象列表中返回单个可迭代对象

```bash

```

- itertools.cycle：

创建一个迭代器的副本，然后循环输出结果。

- itertools.tee(iterable,number)：

从单个可迭代中返回n个独立的可迭代。

- functools.lru_cache：

该装饰器用来记忆映射为参数的结果。

```bash

```

- functools.wraps：

