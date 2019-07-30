# queue模块

- 所属数据结构：FIFO

- 特性：

无法从空队列中移除元素，超出队列最大容量限制时无法添加元素
queque模块是队列这种数据结构的简单实现

- 用途：多线程编程

- 场景分析：

在生产者和消费者之间传递数据，

- 使用提示：注意`Queue`实例的大小，不要超过进程和内存调优的定义范围

## 五个高级API

1. get(): 从queue对象中抛出一个元素
2. put(): 向队列中添加新元素
3. qsize(): 返回队列中的元素数量
4. empty(): 返回一个标记队列是否为空的布尔值
5. full(): 返回布尔值，表示队列是否已满

## 基本FIFO Queue

```python
import queue

q = queue.Queue()
for i in range(5):
    q.put(i)
while not q.empty():
    print(q.get(), end=' ')
print()
```

## LIFO Queue

```python
```

## 优先Queue

```python
import heapq


class PriorityQueue:

    def __init__(self):
        self._queue = []
        self._index = 0

    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, self._index, item))
        self._index += 1

    def pop(self):
        return heapq.heappop(self._queue)[-1]
```

```python
>>> class Item:
...     def __init__(self, name):
...         self.name = name
...     def __repr__(self):
...         return 'Item({!r})'.format(self.name)
...
>>> q = PriorityQueue()
>>> q.push(Item('foo'), 1)
>>> q.push(Item('bar'), 5)
>>> q.push(Item('spam'), 4)
>>> q.push(Item('grok'), 1)
>>> q.pop()
Item('bar')
>>> q.pop()
Item('spam')
>>> q.pop()
Item('foo')
>>> q.pop()
Item('grok')
>>>
```

## 线程安全的优先Queue

参见：线程部分示例