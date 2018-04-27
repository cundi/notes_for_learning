# Redis综述

特点：

- key-value存储系统

value类型：

- string
- list
- set
- zset（有序集合）
- hash



## 安装

```shell
pip install redis
```

## 使用

API分类：

连接方式

- 直接连接
- 连接池

操作

- String 操作
- Hash 操作
- List 操作
- Set 操作
- Sort Set 操作
- 管道
- 发布订阅

## 连接方式

### 直接连接

```python
import redis

r = redis.Redis(host='localhost', port=6379, decode_responses=True)
r.set('foo', 'bar')  #"将键值对存入redis缓存
print(r['foo'])
print(r.get('foo'))  # 取出键name对应的值
print(type(r.get('foo')))
```

### 连接池

```python
# -*- coding:utf-8 -*-
import redis


pool = redis.ConnectionPool(host="127.0.0.1", port=6379)
r = redis.Redis(connection_pool=pool)
r.set('foo', 'bar')
print(r.get('foo'))
```

## 操作

### String操作

语法格式：

set(name, value, ex=None, px=None, nx=False, xx=False)

参数说明：

ex，过期时间（秒）
px，过期时间（毫秒）
nx，如果设置为True，则只有name不存在时，当前set操作才执行
xx，如果设置为True，则只有name存在时，当前set操作才执行

```python
# foo1的值，3秒之后为None
r.set('foo1', 'bar1', ex=3)
# 等同于
r.setex('foo1', 'bar1')

time.sleep(3)
print(r.get('foo1'))

# foo1的值，3000毫秒之后为None
r.set('foo2', 'bar2', px=3000)
# 等同于
r.psetex("foo2", 3000, "bar2")

# 键不存在返回True，否则为None
r.set('foo3', 'bar3', nx=True)
# 等同于.
r.setnx('foo3', 'bar3' )

# 如果键名存在返回True，否则为None
r.set('foo3', 'bar3', xx=True)

# 批量存取值
r.mset()
r.mget({'foo4': 'bar4', 'foo5': 'bar5'})

# 设置新值并获取旧值
r.getset('foo3', 'bar-3')
```

