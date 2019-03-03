# compare standard with async


- 示例-1，异步代码中使用同步sleep

```python
import asyncio  
import time  
from datetime import datetime

async def custom_sleep():  
    print('SLEEP', datetime.now())
    time.sleep(1)
async def factorial(name, number):  
    f = 1
    for i in range(2, number+1):
        print('Task {}: Compute factorial({})'.format(name, i))
        await custom_sleep()
        f *= i
    print('Task {}: factorial({}) is {}\n'.format(name, number, f))

start = time.time()  
loop = asyncio.get_event_loop()
tasks = [  
    asyncio.ensure_future(factorial("A", 3)),
    asyncio.ensure_future(factorial("B", 4)),
]
loop.run_until_complete(asyncio.wait(tasks))  
loop.close()
end = time.time()  
print("Total time: {}".format(end - start))
```

输出:

```shell
Task A: Compute factorial(2)  
SLEEP 2017-04-06 13:39:56.207479  
Task A: Compute factorial(3)  
SLEEP 2017-04-06 13:39:57.210128  
Task A: factorial(3) is 6
Task B: Compute factorial(2)  
SLEEP 2017-04-06 13:39:58.210778  
Task B: Compute factorial(3)  
SLEEP 2017-04-06 13:39:59.212510  
Task B: Compute factorial(4)  
SLEEP 2017-04-06 13:40:00.217308  
Task B: factorial(4) is 24
Total time: 5.016386032104492
```

- 示例-2，异步sleep

```python
import asyncio  
import time  
from datetime import datetime

async def custom_sleep():  
    print('SLEEP {}\n'.format(datetime.now()))
    await asyncio.sleep(1)
async def factorial(name, number):  
    f = 1
    for i in range(2, number+1):
        print('Task {}: Compute factorial({})'.format(name, i))
        await custom_sleep()
        f *= i
    print('Task {}: factorial({}) is {}\n'.format(name, number, f))

start = time.time()  
loop = asyncio.get_event_loop()
tasks = [  
    asyncio.ensure_future(factorial("A", 3)),
    asyncio.ensure_future(factorial("B", 4)),
]
loop.run_until_complete(asyncio.wait(tasks))  
loop.close()
end = time.time()  
print("Total time: {}".format(end - start))
```

```shell
Output:

Task A: Compute factorial(2)  
SLEEP 2017-04-06 13:44:40.648665
Task B: Compute factorial(2)  
SLEEP 2017-04-06 13:44:40.648859
Task A: Compute factorial(3)  
SLEEP 2017-04-06 13:44:41.649564
Task B: Compute factorial(3)  
SLEEP 2017-04-06 13:44:41.649943
Task A: factorial(3) is 6
Task B: Compute factorial(4)  
SLEEP 2017-04-06 13:44:42.651755
Task B: factorial(4) is 24
Total time: 3.008226156234741
```
