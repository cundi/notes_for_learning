# Thread类

```python
class threading.Thread(group=None,
                        target=None,
                        name=None,
                        args=(),
                        kwargs={}
                    )

        """
        :param: group: 用于未来，设置为空，
        :param: target: 目标函数
        :param: name: 线程名称，默认格式为Thread-N格式
        :param: args: 需要传递到目标函数的参数元组
        :param: kwargs: 用于目标函数的关键字参数字典
        """
```

该类的实例方法：

- start()方法让线程开始运行
- join()方法让线程进入等待状态，等待其他线程的完成

示例：

```python
import threading


def function(i):
    print("function called by thread %i\n" %i)
    return


threads = []

for i in range(5):
    t = threading.Thread(target=function , args=(i,))
    threads.append(t)
    t.start()
    t.join()
```

## 获取线程名称

多线程分别用于不同用途时，可以设置并获取线程名称

```python
threading.currentThread().getName()
```

示例：

```python
import threading
import time

def first_function():
    print (threading.currentThread().getName()+\
    str(' is Starting \n'))
    time.sleep(2)
    print (threading.currentThread().getName()+\
    str( ' is Exiting \n'))
    return


def second_function():
    print (threading.currentThread().getName()+\
    str(' is Starting \n'))
    time.sleep(2)
    print (threading.currentThread().getName()+\
    str( ' is Exiting \n'))
    return
    def third_function():
    print (threading.currentThread().getName()+\
    str(' is Starting \n'))
    time.sleep(2)
    print (threading.currentThread().getName()+\
    str( ' is Exiting \n'))
    return


if __name__ == "__main__":
    t1 = threading.Thread\
    (name='first_function', target=first_function)
    t2 = threading.Thread\
    (name='second_function', target=second_function)
    t3 = threading.Thread\
    (name='third_function',target=third_function)
    t1.start()
    t2.start()
    t3.start()
    t1.join()
    t2.join()
    t3.join()
```

## 子类化 Thread

```python
import threading
import time

exitFlag = 0

class myThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
```
