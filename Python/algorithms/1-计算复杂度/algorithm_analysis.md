# 算法分析

## 问答

计算机可以执行的基本操作有哪些？

这些基本操作耗费了多少时间？

计算复杂度的定义是什么？

为什么要关心计算复杂度？

什么时候需要关心代码片段的复杂度？

如何提高一段代码的效率？

Big-O注记的定义是什么？

## 算法分析释疑

概念：

- 一个函数花费了O（1）时间，意味着该函数仅执行了1个步骤。
- 如果为O(n)表示该函数执行n个步骤才完成。

1000个项时的列表和字典复杂度示例：

```python
n = 1000
a = list(range(n))
b = dict.fromkeys(range(n))

 for i in range(100):
       i in a  # takes n=1000 steps
       i in b  # takes 1 step
```

复杂度为O(1), O(n), 和 O(n**2) 的函数示例:

```python
def o_one(items):
       return 1  # 1 operation so O(1)
   def o_n(items):
       total = 0
       # Walks through all items once so O(n)
       for item in items:
           total += item
       return total
   def o_n_squared(items):
       total = 0
       # Walks through all items n*n times so O(n**2)
       for a in items:
           for b in items:
               total += a * b
return total


   n = 10
   items = range(n)
   o_one(items)  # 1 operation
   o_n(items)  # n = 10 operations
   o_n_squared(items)  # n*n = 10*10 = 100 operations
```

## 列表元素存取

这里需要证明的理论：

- 列表的大小不影响其平均访问时间？
- 无视元素在列表中位置，对任意位置元素的平均访问时间都是相同的？

```python
# 列表访问时间
import datetime
import random
import time


def main():
    # 写一个xml文件
    file = open("ListAccessTiming.xml","w")
    file.write(’<?xml version="1.0" encoding="UTF-8" standalone="no" ?>\n’) file.write(’<Plot title="Average List Element Access Time">\n’)

    # 大小从1000到20000的测试列表
    xmin = 1000
    xmax = 20000

    # 在yList的1000次重新取回中计算xList的平均访问时间
    xList = []
    yList = []

    for x in range(xmin, xmax+1, 1000):
        xList.append(x)
        prod = 0
        # 创建一个大小为x的列表
        lst = [0] * x

        # 等待垃圾回收或者内存定位完成
        time.sleep(1)

        # 执行1000次重新取回测试之前的时间
        starttime = datetime.datetime.now()

        for v in range(1000):
            # 在列表的随机位置取回值。
            index = random.randint(0, x-1)
            val  = lst[index]
            # 执行一个假操作，以确保值真的取回来了
            prod = prod * vale
        # 执行1000次重新取回测试以后的时间
        endtime = datetime.datetime.now()

        # 具体耗时
        deltaT = endtime - starttime

        # 平均访问时间
        accessTime = deltaT.total_seconds() * 1000

        yList.append(accessTime)

    file.write('<Axes>\n')
    file.write('XAxis min="’+str(xmin)+’" max="’+str(xmax)+’">List Size</XAxis>\n')
    file.write('</Axes>\n')
    file.write('<Sequence title="Average Access Time vs List Size" color="red">\n')

    for i in range(len(xList)):
        file.write()

    file.write('</Sequence>\n')

    #
```