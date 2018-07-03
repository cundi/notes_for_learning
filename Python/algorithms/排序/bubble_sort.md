# Bubble Sort

关键点：项的交换应该由内部循环的次数决定

```python
In [142]: for i in list(range(9, 0, -1)):
     ...:     print('outter: ', i)
     ...:     for j  in range(i):
     ...:         print('innter: ', j)
     ...:
In [145]: for i in list(range(10)):
     ...:     print('out: ', i)
     ...:     for j in range(10 -1 -i):
     ...:         print('in: ', j)
```


需要排序的列表

```python
l = [random.randint(-50, 50) for c in range(20)]
```

实现方法有以下几种：

- 单层循环

```python
# 这里为由小到大排序
print('Before sort\n')
print(l)

print('After sort\n')
print(l)
```

```python
In [78]: l = [random.randint(-50, 50) for c in range(20)]

In [79]: i = 1

In [80]: while i<len(l):
    ...:     if l[i] < l[i-1]:
    ...:         temp = l[i]
    ...:         l[i], l[i-1] = l[i-1], l[i]
    ...:         i -=1
    ...:     else:
    ...:         i += 1
```

```python
In [132]: l = [random.randint(-50, 50) for c in range(20)]

In [133]: n = len(l)

In [134]: while n >1:
     ...:     i = 1
     ...:     while i < n:
     ...:         if l[i] < l[i-1]:
     ...:             l[i], l[i-1] = l[i-1] , l[i]
     ...:         i += 1
     ...:     n -= 1
     ...:

In [135]: print(l)
[-47, -45, -40, -26, -21, -18, -6, -4, 0, 1, 3, 8, 8, 11, 14, 20, 22, 24, 34, 45]
```

- 双层循环

```python
def bubble_sort(items):
    print('Before sort: ', items)
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            print(items[j], [items[j+1]])
            if items[j] > items[j+1]:
                items[j], items[i] = items[j+1], items[j]
    print('After sort: ', items)
```

- 递归

比较以上几种实现的复杂度