---
layout: page
title: "Threading in Python"
permalink: /python/concurrency/threading
---

## Python threading overview

Due to the GIL getting multiple tasks running simultaneously in Python requires a non-standard implementation of Python, writing some of your code in a different language or use of [multiprocessing]({% link python/concurrency/multiprocessing.md %}) which has extra overhead.

Because of the CPython implementation of Python works, threading may not speed up all tasks. Tasks that spend much of their time waiting for external events are generally good candidates for threading.  Problems that require heavy CPU computation and spend little time waiting for external events likely won't run faster at all and likely will run slower.

### Daemon threads

While daemon is typically referred to a process that runs in the background in Python threading a daemon is a thread that will shut down immediately when the program exits.  If your program is running threads that are not daemons, the program will wait for those threads to complete before exiting.

## Starting threads

There are a few things to note about the example below.

* `ThreadPoolExecutor` is the preferred way of calling threads.  Its a context manager that handles the starting and joining on your behalf.

* You can use the map function to call the thread function multiple times and pass it an argument.  You can also use `executor.submit` if you want to be able to get result or have more granular arguments

* ***Note:*** `ThreadPoolExecutor` will hide the errors thrown in the functions it is calling.  This can make it hard to troubleshoot/debug.

```python
import time
import concurrent.futures
import random

def thread_function(name):
    print(f"Thread {name}: starting")
    time.sleep(2)
    print(f"Thread {name}: finishing")

def thread_function2(num: int) -> int:
    time.sleep(1)
    return num + random.randint(1,2)

if __name__ == "__main__":
    with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
        # short hand way
        executor.map(thread_function, ['apple','banana', 'kiwi'])

        for i in range(3):
            f1 = executor.submit(thread_function2, i)
            # getting result is optional you can just let the thread run
            print(f1.result())
```

### Starting multiple threads without a context manager

While this is not recommended here is how you would run multiple threads without using the `ThreadPoolExecutor`.  Note that you need to call `join()` to have one thread (including your main program) wait for another thread to finish.

```python
import threading
import time

def thread_function(name):
    print(f"Thread {name}: starting")
    time.sleep(2)
    print(f"Thread {name}: finishing")

if __name__ == "__main__":
    threads = list()
    for index in range(3):
        print(f"Main    : create and start thread {index}.")
        x = threading.Thread(target=thread_function, args=(index,))
        threads.append(x)
        x.start()

    for index, thread in enumerate(threads):
        print(f"Main    : before joining thread {index}")
        thread.join()
        print(f"Main    : thread {index} done")
```

## Protecting threaded programs from race conditions

Race conditions can occur when two or more thread access a shared piece of data or resource and modify it simultaneously leading to lost or corrupt data.

### Basic synchronization using Lock

The example below uses `threading.RLock` to implement a basic thread safe counter.  A few things to note.

* While this is thread safe we are not necessarily protecting from a deadlock.  If one thread grabs the lock and does not give it back your program will get stuck.  For this reason you want to use `RLock` via a context manager as in the example.

* Note the use of `local_copy` in the update code is not needed, its just used to set up a situation where without using `RLock` a race conditions would occur.

* The reason to use `RLock` and not `Lock` is if you use `Lock` and call `acquire()` the program will wait as long as it takes to get the lock.  Its easy to have a bug where same thread calls `acquire()` when it already has the lock.  This will lead to your code getting stuck.  `RLock` protects from this by allowing `acquire()` to be called multiple times by the same thead as long as it releases the lock as many times as it acquired it.

```python
import threading
import time

class Counter:
    def __init__(self):
        self.value = 0
        self._lock = threading.RLock()

    def update(self, name):
        with self._lock:
            local_copy = self.value
            local_copy += 1
            time.sleep(0.1)
            self.value = local_copy

if __name__ == "__main__":
    database = Counter()
    print(f"Testing update. Starting value is {database.value}.")
    with concurrent.futures.ThreadPoolExecutor(max_workers=25) as executor:
        for index in range(25):
            executor.submit(database.update, index)
    print(f"Testing update. Ending value is {database.value}")
```

## Producer consumer using queue

The producer consumer problem where you have one process producing (writing) and another consumer the writes.  You need to be able to synchronize the two as you can have producer produce things faster than the consumer can consume them.

Python's standard library has a `queue` module with a thread safe `Queue` class.

The other primitive from Python threading `Event`. `threading.Event` allows one thread to signal an event while many other threads can be waiting for that event to happen. The threads that are waiting for the event do not necessarily need to stop what they are doing, they can check the status of the event every once in a while.

***Note:*** in the producer code we don't just wait for the event to be set.  We also check for the queue to be empty before exiting.  This for the event that producer exists, but we still have data in the queue that needs to be processes.

```python
import concurrent.futures
import queue
import random
import threading
import time

def producer(queue, event):
    while not event.is_set():
        message = random.randint(1, 101)
        print(f"Producer writing message: {message}")
        queue.put(message)

    print("Producer received event. Exiting")

def consumer(queue, event):
    while not event.is_set() or not queue.empty():
        message = queue.get()
        print(f"Consumer storing message: {message} size={queue.qsize()}")

    print("Consumer received event. Exiting")

if __name__ == "__main__":
    pipeline = queue.Queue(maxsize=10)
    event = threading.Event()
    with concurrent.futures.ThreadPoolExecutor(max_workers=2) as executor:
        executor.submit(producer, pipeline, event)
        executor.submit(consumer, pipeline, event)
        time.sleep(0.1)
        print("Main: about to set event")
        event.set()
```

## Other Python threading objects

### Semaphore

`threading.Semaphore` is a counter that is atomic (guaranteed that OS won't swap threads when you are incrementing or decrementing). The counter is incremented when you call `.release()` and decremented when you call `.acquire()`.  If `acquire()` is called when the counter is zero, that thread will block until a different thread calls `.release()` and increments the counter to one.  Semaphores are frequently used to protect a resource that has a limited capacity such as a pool of connections that needs to be limited to a specific number.

### Timer

`threading.Timer` is a way to schedule a function to be called after a certain amount of time has passed.  You create a `Timer` by passing in a number of seconds to wait and a function to call.

`t = threading.Timer(30.0, my_functions)`

Your start the `Timer` by calling `.start()`.  The function will be called on a new thread at some point after the specified time (there is not promise that it will be called at exactly the time you want).

To stop a `Timer` that is already started you can call `.cancel()`.  If `cancel()` is called after timer has executed the cancel does nothing and does not produce an exception.

Timers can be used to timeout actions such as giving user some time to make selection.

### Barrier

`threading.Barrier` can be used to keep a fixed number of threads in sync. When calling a `Barrier`, the caller must specify how many threads will be synchronizing on it. Each thread calls `.wait()` on the `Barrier`.  They all will remain blocked until the specified number of threads are waiting, and then they are all released at the same time.

Because threads are scheduled by the operating system even though they are released simultaneously they will be scheduled to run at the same time.

An example use of `Barrier` is to allow a pool of threads to initialize themselves.  Having them wait of a `Barrier` ensures that none of the threads start running before all of the threads are finished with their initialization.