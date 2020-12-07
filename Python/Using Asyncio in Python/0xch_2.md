# CHAPTER 2 The Truth About Threads

Let’s be frank for a moment—you really don’t want to use Curio. All things equal, you
should probably be programming with threads. Yes, threads. THOSE threads. Seri‐
ously. I’m not kidding.
—Dave Beazley, “Developing with Curio”

If you’ve never heard of threads before, here’s a basic description: threads are a feature
provided by an operating system (OS), made available to software developers so that
they may indicate to the OS which parts of their program may be run in parallel. The
OS decides how to share CPU resources with each of the parts, much as the OS
decides to share CPU resources with all the other different programs (processes) run‐
ning at the same time.

Since you’re reading an Asyncio book, this must be the part where I tell you,
“Threads are terrible, and you should never use them,” right? Unfortunately, the sit‐
uation is not so simple. We need to weigh the benefits and risks of using threads, just
like with any technology choice.

This book is not supposed to be about threads at all. But there are two problems here:
Asyncio is offered as an alternative to threading, so it’s hard to understand the value
proposition without some comparison; and even when using Asyncio, you will still
likely have to deal with threads and processes, so you need to know something about
threading.

>The context of this discussion is exclusively concurrency in net‐
>work programming applications. Preemptive multithreading is also
>used in other domains, where the trade-offs are entirely different.

## Benefits of Threading
These are the main benefits of threading:

`Ease of reading code`

Your code can run concurrently, but still be set out in a very simple, top-down
linear sequence of commands to the point where—and this is key—you can pre‐
tend, within the body of your functions, that no concurrency is happening.

`Parallelism with shared memory`

Your code can exploit multiple CPUs while still having threads share memory.
This is important in many workloads where it would be too costly to move large
amounts of data between the separate memory spaces of different processes, for
example.

`Know-how and existing code`

There is a large body of knowledge and best practices available for writing threa‐
ded applications. There is also a huge amount of existing “blocking” code that
depends on multithreading for concurrent operation.

Now, with Python, the point about parallelism is questionable because the Python
interpreter uses a global lock, called the global interpreter lock (GIL), to protect the
internal state of the interpreter itself. That is, it provides protection from the potential
catastrophic effects of race conditions between multiple threads. A side effect of the
lock is that it ends up pinning all threads in your program to a single CPU. As you
might imagine, this negates any parallelism performance benefits (unless you use
tools like Cython or Numba to maneuver around the limitation).

The first point regarding perceived simplicity, however, is significant: threading in
Python feels exceptionally simple, and if you haven’t been burned before by impossi‐
bly hard race condition bugs, threading offers a very attractive concurrency model.
Even if you have been burned in the past, threading remains a compelling option
because you will likely have learned (the hard way) how to keep your code both sim‐
ple and safe.

I don’t have space to get into safer threaded programming here, but generally speak‐
ing, the best practice for using threads is to use the ThreadPoolExecutor class from
the concurrent.futures module, passing all required data in through the submit()
method. Example 2-1 shows a basic example.

Example 2-1. Best practice for threading

```python
from concurrent.futures import ThreadPoolExecutor as Executor

def worker(data):
    <process the data>

with Executor(max_workers=10) as exe:
    future = exe.submit(worker, data)
```

The `ThreadPoolExecutor` offers an extremely simple interface for running functions
in a thread—and the best part is that, if needed, you can convert the pool of threads
into a pool of subprocesses simply by using ProcessPoolExecutor instead. It has the
same API as ThreadPoolExecutor, which means that your code will be little affected
by the change. The executor API is also used in asyncio and is described in the next
chapter (see Example 3-3).

In general, you’ll prefer your tasks to be somewhat short-lived, so that when your
program needs to shut down, you can simply call Executor.shutdown(wait=True)
and wait a second or two to allow the executor to complete.

Most importantly: if at all possible, you should try to prevent your threaded code (in
the preceding example, the worker() function) from accessing or writing to any
global variables!

## Drawbacks of Threading
[N]ontrivial multithreaded programs are incomprehensible to humans. It is true that
the programming model can be improved through the use of design patterns, better
granularity of atomicity (e.g., transactions), improved languages, and formal methods.
However, these techniques merely chip away at the unnecessarily enormous nondeterminism of the threading model. The model remains intrinsically intractable.

—Edward A. Lee “The Problem with Threads”

The drawbacks of threading have been mentioned in a few other places already, but
for completeness let’s collect them here anyway:

`Threading is difficult`

Threading bugs and race conditions in threaded programs are the hardest kinds
of bugs to fix. With experience, it is possible to design new software that is less
prone to these problems, but in nontrivial, naively designed software, they can be
nearly impossible to fix, even by experts. Really!

`Threads are resource-intensive`

Threads require extra operating system resources to create, such as preallocated,
per-thread stack space that consumes process virtual memory up front. This is a
big problem with 32-bit operating systems, because the address space per process
is limited to 3 GB.1 Nowadays, with the widespread availability of 64-bit operat‐
ing systems, virtual memory isn’t as precious as it used to be (addressable space
for virtual memory is typically 48 bits; i.e., 256 TiB). On modern desktop operat‐
ing systems, the physical memory required for stack space for each thread isn’t
even allocated by the OS until it is required, including stack space per thread. For
example, on a modern, 64-bit Fedora 29 Linux with 8 GB memory, creating
10,000 do-nothing threads with this short snippet:

```python
# threadmem.py
import os
from time import sleep
from threading import Thread
threads = [
Thread(target=lambda: sleep(60)) for i in range(10000)]
[t.start() for t in threads]
print(f'PID = {os.getpid()}')
[t.join() for t in threads]
```

leads to the following information in top:

```shell
MiB Mem : 7858.199 total, 1063.844 free, 4900.477 used
MiB Swap: 7935.996 total, 4780.934 free, 3155.062 used

PID USER PR NI VIRT RES SHR COMMAND
15166 caleb 20 0 80.291g 131.1m 4.8m python3
```

Preallocated virtual memory is a staggering ~80 GB (due to 8 MB stack space per
thread!), but resident memory is only ~130 MB. On a 32-bit Linux system, I
would be unable to create this many because of the 3 GB user-space addressspace limit, 
regardless of actual consumption of physical memory. To get around
this problem on 32-bit systems, it is sometimes necessary to decrease the
preconfigured stack size, which you can still do in Python today, with
threading.stack_size([size]). Obviously, decreasing stack size has implica‐
tions for runtime safety with respect to the degree to which function calls may be
nested, including recursion. Single-threaded coroutines have none of these prob‐
lems and are a far superior alternative for concurrent I/O.

`Threading can aect throughput`

At very high concurrency levels (say, >5,000 threads), there can also be an impact
on throughput due to context-switching costs, assuming you can figure out how
to configure your operating system to even allow you to create that many
threads! It has become so tedious on recent macOS versions, for example, to test
the preceding 10,000 do-nothing-threads example, that I gave up trying to raise
the limits at all.

`Threading is inflexible`

The operating system will continually share CPU time with all threads regardless
of whether a thread is ready to do work or not. For instance, a thread may be
waiting for data on a socket, but the OS scheduler may still switch to and from
that thread thousands of times before any actual work needs to be done. (In the
async world, the select() system call is used to check whether a socket-awaiting
coroutine needs a turn; if not, that coroutine isn’t even woken up, avoiding any
switching costs completely.)

None of this information is new, and the problems with threading as a programming
model are not platform-specific either. For example, this is what the Microsoft Visual
C++ documentation says about threading:

None of this information is new, and the problems with threading as a programming
model are not platform-specific either. For example, this is what the Microsoft Visual
C++ documentation says about threading:

    The central concurrency mechanism in the Windows API is the thread. You typically
    use the CreateThread function to create threads. Although threads are relatively easy to
    create and use, the operating system allocates a significant amount of time and other
    resources to manage them. Additionally, although each thread is guaranteed to receive
    the same execution time as any other thread at the same priority level, the associated
    overhead requires that you create sufficiently large tasks. For smaller or more finegrained tasks, 
    the overhead that is associated with concurrency can outweigh the bene‐
    fit of running the tasks in parallel.

But—I hear you protest—this is Windows, right? Surely a Unix system doesn’t have
these problems? Here follows a similar recommendation from the Mac Developer
Library’s Threading Programming Guide:

    Threading has a real cost to your program (and the system) in terms of memory use
    and performance. Each thread requires the allocation of memory in both the kernel
    memory space and your program’s memory space. The core structures needed to man‐
    age your thread and coordinate its scheduling are stored in the kernel using wired
    memory. Your thread’s stack space and per-thread data is stored in your program’s
    memory space. Most of these structures are created and initialized when you first cre‐
    ate the thread—a process that can be relatively expensive because of the required inter‐
    actions with the kernel.

They go even further in the Concurrency Programming Guide (emphasis mine):

    In the past, introducing concurrency to an application required the creation of one or
    more additional threads. Unfortunately, writing threaded code is challenging. Threads
    are a low-level tool that must be managed manually. Given that the optimal number of
    threads for an application can change dynamically based on the current system load
    and the underlying hardware, implementing a correct threading solution becomes
    extremely difficult, if not impossible to achieve. In addition, the synchronization
    mechanisms typically used with threads add complexity and risk to software designs
    without any guarantees of improved performance.

These themes repeat throughout:

• Threading makes code hard to reason about.
• Threading is an inefficient model for large-scale concurrency (thousands of con‐
current tasks).

Next, let’s look at a case study involving threads that highlights the first and most
important point.

## Case Study: Robots and Cutlery
Second, and more important, we did not (and still do not) believe in the standard mul‐
tithreading model, which is preemptive concurrency with shared memory: we still
think that no one can write correct programs in a language where “a = a + 1” is not
deterministic.
—Roberto Ierusalimschy et al., “The Evolution of Lua”

At the start of this book, I told the story of a restaurant in which humanoid robots—
ThreadBots—did all the work. In that analogy, each worker was a thread. In the case
study in Example 2-2, we’re going to look at why threading is considered unsafe.

Example 2-2. ThreadBot programming for table service

```python
import threading
from queue import Queue

class ThreadBot(threading.Thread):

def __init__(self):
    super().__init__(target=self.manage_table)
    self.cutlery = Cutlery(knives=0, forks=0)
    self.tasks = Queue()

def manage_table(self):
    while True:
        task = self.tasks.get()
        if task == 'prepare table':
            kitchen.give(to=self.cutlery, knives=4, forks=4)
        elif task == 'clear table':
            self.cutlery.give(to=kitchen, knives=4, forks=4)
        elif task == 'shutdown':
            return
```

