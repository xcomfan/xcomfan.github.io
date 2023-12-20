# Terms and concepts

*Concurrency* is single CPU hopping between tasks to make it look like they are all running at once.

*Parallelism* is multiple CPUs/Cores doing multiple computations at the same time.

*Execution Unit* is a general term for objects that execute code concurrently each with independent state and call stack.  Python natively supports three kinds of execution units with its core packages  `threading`, `multiprocessing` and `asyncio`.

*Process* OS processes as you know them.

*Thread* An execution unit within a single process.

*Coroutine* A function that can suspend itself and resume later.  

Threading is a good option if you are dealing with IO stuff if you need compute you should be doing `multiprocessing` or `concurrent.futures.ProcessPoolExecutor`

## Concepts above as they apply to Python Programming

1. Each instance of the Python interpreter is a process.  You can start additional Python processes using the `multiprocessing` of `concurrent.futures` libraries.  Python `subprocess` library is designed to launch processes to run external programs, regardless of teh languages used to write them.

2. The Python interpreter uses a single thread to run the user's program and the memory garbage collector.  You can start additional Python threads using the `threading` or `concurrent.futures` libraries.

3. Access to object reference counts and other internal interpreter state is controlled by a lock, (the Global Interpreter Lock (GIL)).  Only one python thread can hold the GIL at any time, which means that only one thread can execute Python code at any time, regardless of the number of CPU cores.

4. To prevent a Python thread from holding the GIL indefinitely, Python’s bytecode interpreter pauses the current Python thread every 5ms by default,4 releasing the GIL. The thread can then try to reacquire the GIL, but if there are other threads waiting for it, the OS scheduler may pick one of them to proceed.

5. When we write Python code, we have no control over the GIL. But a built-in function or an extension written in C—or any language that interfaces at the Python/C API level—can release the GIL while running time-consuming tasks.

6. Every Python standard library function that makes a syscall5 releases the GIL. This includes all functions that perform disk I/O, network I/O, and time.sleep().

7. Extensions that integrate at the Python/C API level can also launch other non-Python threads that are not affected by the GIL. Such GIL-free threads generally cannot change Python objects, but they can read from and write to the memory underlying objects that support the buffer protocol, such as bytearray, array.array, and NumPy arrays.

8. The effect of the GIL on network programming with Python threads is relatively small, because the I/O functions release the GIL, and reading or writing to the network always implies high latency—compared to reading and writing to memory.

9. Contention over the GIL slows down compute-intensive Python threads. Sequential, single-threaded code is simpler and faster for such tasks

10. To run CPU-intensive Python code on multiple cores, you must use multiple Python processes.

Summary of above from documentation...

```text
 In CPython, due to the Global Interpreter Lock, only one thread can execute Python code at once (even though certain performance-oriented libraries might overcome this limitation). If you want your application to make better use of the computational resources of multicore machines, you are advised to use multiprocessing or concurrent.futures.ProcessPoolExecutor. However, threading is still an appropriate model if you want to run multiple I/O-bound tasks simultaneously.
```

## Spinner program thread implementation

```python
import itertools
import time
from threading import Thread, Event

# This function runs in a separate thread.  The done argument is an instance of threading.Event,
# a simple object to synchronize threads.
def spin(msg: str, done: Event) -> None:  
    for char in itertools.cycle(r'\|/-'):
        # Trick for text mode animation: move the cursor back to the start fo the line with 
        # \r carriage return.  
        status = f'\r{char} {msg}'
        print(status, end='', flush=True)
        # The Event.wait(timeout=None) method returns True when the event is set by another thread;
        # if the timeout elapses, it returns False.  The .1s timeout sets the "frame rate" of the animation
        # to 10 FPS.
        if done.wait(.1):  
            break  
    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='')  

def slow() -> int:
    # Imagine this function as being some sort of long call over a network.
    time.sleep(3)  
    return 42

# supervisor returns the result of slow function
def supervisor() -> int:
    # The threading.Event instances is the key to coordinate the activities of the main thread and the spinner
    # thread.
    done = Event() 
    # To create a new thread, provide a function s the target keyword argument, and positional arguments to the 
    # target as a tuple passed via args. 
    spinner = Thread(target=spin, args=('thinking!', done))  
    print(f'spinner object: {spinner}')  
    # start the spinner thread
    spinner.start()  
    # call slow, which blocks the main thread.  Meanwhile, the secondary thread is running the spinner animation.
    result = slow()  
    # Set the Event flag to True; this will terminate the for loop inside the spin function.
    # When the main thread sets the done event, the spinner thread will eventually notice and exit cleanly.
    done.set()
    # Wait until the spinner thread finishes
    spinner.join()
    return result

def main() -> None:
    result = supervisor()  
    print(f'Answer: {result}')

if __name__ == '__main__':
    main()
```

## Spinner program implemented with Processes

The `multiprocessing` package supports running concurrent tasks in separate Python processes instead of threads.  When you create a `multiprocessing.Process` instance, a whole new Python interpreter is started as a child process.  Since each Python process has its own GIL, this allows your program to use all available CPU cores but ultimates relies on OS for scheduling.

The basic API of threading and multiprocessing are similar, but their implementation is very different, and multiprocessing has a much larger API to handle the added complexity of multi-process programming. For example, one challenge when converting from threads to processes is how to communicate between processes that are isolated by the operating system and can’t share Python objects. This means that objects crossing process boundaries have to be serialized and deserialized, which creates overhead. In example below, the only data that crosses the process boundary is the Event state, which is implemented with a low-level OS semaphore in the C code underlying the multiprocessing module

```python
import itertools
import time
from multiprocessing import Process, Event  
from multiprocessing import synchronize 

def spin(msg: str, done: synchronize.Event) -> None: 
    for char in itertools.cycle(r'\|/-'):  
        status = f'\r{char} {msg}'  
        print(status, end='', flush=True)
        if done.wait(.1):
            break  
    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='')  

def slow() -> int:
    time.sleep(3)  
    return 42

def supervisor() -> int:
    done = Event()
    spinner = Process(target=spin,               
                      args=('thinking!', done))
    # The spinner object will display the pid of the child process in this implementation.
    print(f'spinner object: {spinner}')          
    spinner.start()
    result = slow()
    done.set()
    spinner.join()
    return result

def main() -> None:
    result = supervisor()  
    print(f'Answer: {result}')

if __name__ == '__main__':
    main()
```

## Spinner implemented with Coroutines

It is the job of OS schedulers to allocate CPU time to drive threads and processes. In contrast, coroutines are driven by an application-level event loop that manages a queue of pending coroutines, drives them one by one, monitors events triggered by I/O operations initiated by coroutines, and passes control back to the corresponding coroutine when each event happens. The event loop and the library coroutines and the user coroutines all execute in a single thread. Therefore, any time spent in a coroutine slows down the event loop—and all other coroutines.

```python
import asyncio
import itertools

async def spin(msg: str) -> None:  
    for char in itertools.cycle(r'\|/-'):
        status = f'\r{char} {msg}'
        print(status, flush=True, end='')
        try:
            # Use asyncio.sleep instead of time.sleep to pause without blocking other coroutines.
            await asyncio.sleep(.1)
        # asyncio.CancelledError is raised when the cancel method is called on
        # the Task controlling this coroutine indicating its time to exit the loop.
        except asyncio.CancelledError:  
            break
    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='')

async def slow() -> int:
    # using await asyncio.sleep instead of time.sleep here as well.
    await asyncio.sleep(3)  
    return 42

# main is the only regular function defined in this program.  All others are coroutines
def main() -> None:
    # The asyncio.run function starts the event loop to drive the coroutine that will eventually set the other
    # coroutines in motion.  The main function will stay blocked until supervisor returns. 
    # The return value of supervisor will bet the return value of asyncio.run. 
    result = asyncio.run(supervisor())  
    print(f'Answer: {result}')

# Native coroutines are defined with async def.
async def supervisor() -> int:
    # asyncio.create_task schedules the eventual execution of spin, immediately returning
    # an instance of asyncio.Task.
    spinner = asyncio.create_task(spin('thinking!'))  
    print(f'spinner object: {spinner}')
    # The await keyword calls slow, blocking supervisor until slow returns.  The return value of slow will be assigned to result.
    result = await slow()
    # The Task.cancel method raises a CancelledError exception inside the spin coroutine.   
    spinner.cancel()  
    return result

if __name__ == '__main__':
    main()
```

The above example demonstrates the three main ways to run a coroutine...

*asyncio.run(coro())* - Called from a regular function to drive a coroutine object that usually is the entry point for all the asynchronous code in the program.  The call blocks until the body fo `coro` returns.  The return value of the `run()` call is whatever the body of the coro returns.

*asyncio.create_task(coro())* - Called from a coroutine to schedule another coroutine to execute eventually.  This call does not suspend the current coroutine.  It returns a `Task` instance, an object that wraps the coroutine object and provides methods to control and query its state.

*await coro()* - Called from a coroutine to transfer control to the coroutine object returned by `coro`.  The value of the await expression is whatever the `coro` returns.

***Note***: invoking a coroutine as `coro()` immediately returns a coroutine object, but does nto run the body of the `coro` function.  Driving the body of the coroutines is the job of the event loop.

## Summary of the differences and similarities to note between threads and coroutines

An asyncio.Task is roughly the equivalent of a threading.Thread.

A Task drives a coroutine object, and a Thread invokes a callable.

A coroutine yields control explicitly with the await keyword.

You don’t instantiate Task objects yourself, you get them by passing a coroutine to asyncio.create_task(…).

When asyncio.create_task(…) returns a Task object, it is already scheduled to run, but a Thread instance must be explicitly told to run by calling its start method.

In the threaded supervisor, slow is a plain function and is directly invoked by the main thread. In the asynchronous supervisor, slow is a coroutine driven by await.

There’s no API to terminate a thread from the outside; instead, you must send a signal—like setting the done Event object. For tasks, there is the Task.cancel() instance method, which raises CancelledError at the await expression where the coroutine body is currently suspended.

The supervisor coroutine must be started with asyncio.run in the main function.

With coroutines you don't have to worry about your thread being interrupted.  You must explicitly `await` to let the rest of the program run.  Instead of holding locks coroutines are synchronized by definition because only one of them can run at any given time.

## Concurrent Executors

The `concurrent.futures.Executor` classes encapsulate the pattern of spawning a bunch of independent threads and collecting the results in a queue.

`futures` objects represent the asynchronous execution of an operation.  Similar to JavaScript promises.

Here we will compare two script that download flag images.  One implemented with `concurrent.futures` and one with `asyncio`

Two notes from these examples..

1. Regardless of the concurrency constructs you use—threads or coroutines—you’ll see vastly improved throughput over sequential code in network I/O operations, if you code it properly.

2. For HTTP clients that can control how many requests they make, there is no significant difference in performance between threads and coroutines

The main features of the `concurrent.futures` package are the `ThreadPoolExecutor` and `ProcessPoolExecutor` classes which implement an API to submit callables for execution in different threads or processes.  The classes transparently manage a pool of worker threads or processes and queues to distribute jobs and collect results.

Note that the example below is using Threads.

```python
import time
import httpx

from pathlib import Path
from typing import Callable
from concurrent import futures

POP20_CC = ('CN IN US ID BR PK NG BD RU JP '
            'MX PH VN ET EG DE IR TR CD FR').split()  

BASE_URL = 'https://www.fluentpython.com/data/flags'  
DEST_DIR = Path('downloaded')                         

def save_flag(img: bytes, filename: str) -> None:     
    (DEST_DIR / filename).write_bytes(img)

def get_flag(cc: str) -> bytes:  
    url = f'{BASE_URL}/{cc}/{cc}.gif'.lower()
    resp = httpx.get(url, timeout=6.1,       
                     follow_redirects=True)  
    resp.raise_for_status()  
    return resp.content

# Function to download a single image; this is what each worker will execute.
def download_one(cc: str):  
    image = get_flag(cc)
    save_flag(image, f'{cc}.gif')
    print(cc, end=' ', flush=True)
    return cc

def download_many(cc_list: list[str]) -> int:
    # Instantiate the ThreadPoolExecutor as a context manager; the executor.__exit__ method will
    # call executor.shutdown(wait=True)
    with futures.ThreadPoolExecutor() as executor:
        # The map method is similar to the map build-in, except that the download_one function will be 
        # called concurrently from multiple threads.  It returns a generator that you can iterate to 
        # retrieve the value returned by each function call.     
        res = executor.map(download_one, sorted(cc_list))  

    return len(list(res))                                 

def main(downloader: Callable[[list[str]], int]) -> None:  
    DEST_DIR.mkdir(exist_ok=True)                          
    t0 = time.perf_counter()                               
    count = downloader(POP20_CC)
    elapsed = time.perf_counter() - t0
    print(f'\n{count} downloads in {elapsed:.2f}s')

if __name__ == '__main__':
    main(download_many)
```

## Launching Processes with concurrent.futures

The `concurrent.futures` package enabled parallel computation on multi-core machines because it supports distributing work among multiple Python processes using the `ProcessPoolExecutor` class.

Both `ProcessPoolExecutor` and `ThreadPoolExecutor` implement the `Executor` interface, making it easy to switch from thread based to process based.

Essentially to use ProcessPool instead of a Thread Pool you change the line `with futures.ThreadPoolExecutor() as executor:` in example above to `with futures.ProcessPoolExecutor() as executor:` Everything else remains the same.

In my WSL environment processes ran much faster than threads.  Will be interesting to tes this on Native Linux system as book says for I/O bound stuff Threads vs process should not make any difference and process approach is more expensive in using more memory and taking longer to start.

Below is an example of using a process pool to compute prime numbers.

```python
import sys
from concurrent import futures  
from time import perf_counter
from typing import NamedTuple
import math

NUMBERS = [i for i in range(0, 50000)]

def is_prime(n: int) -> bool:
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False

    root = math.isqrt(n)
    for i in range(3, root + 1, 2):
        if n % i == 0:
            return False
    return True

class PrimeResult(NamedTuple):  
    n: int
    flag: bool
    elapsed: float

def check(n: int) -> PrimeResult:
    t0 = perf_counter()
    res = is_prime(n)
    return PrimeResult(n, res, perf_counter() - t0)

def main() -> None:
    if len(sys.argv) < 2:
        workers = None      
    else:
        workers = int(sys.argv[1])

    # Not using the with block here just so we can print the number of workers on the next line.
    executor = futures.ProcessPoolExecutor(workers)  
    # Type ignore here because _max_workers is an undocumented internal variable
    actual_workers = executor._max_workers  # type: ignore  

    print(f'Checking {len(NUMBERS)} numbers with {actual_workers} processes:')

    t0 = perf_counter()

    numbers = sorted(NUMBERS, reverse=True)  
    # use executor as context manager
    with executor:  
        # The executor.map call returns the PrimeResult instances returned by check in the same order as the 
        # numbers argument.
        for n, prime, elapsed in executor.map(check, numbers):  
            label = 'P' if prime else ' '
            print(f'{n:16}  {label} {elapsed:9.6f}s')

    time = perf_counter() - t0
    print(f'Total time: {time:.2f}s')

if __name__ == '__main__':
    main()
```

One note about the script above is that we are using map function which will return the results in same order as the numbers array.  This may not be what you want and you can get more flexibility with the combination of `executor.submit` and `futures.as_completed` because you can submit different callables and arguments while map is intended to run the same callable on different arguments.  An example of that is below...

```python
from time import sleep, strftime
from concurrent import futures

def display(*args):  
    print(strftime('[%H:%M:%S]'), end=' ')
    print(*args)

# loiter does nothing except display a message when ti starts, sleep for n seconds, 
# then display a message when it ends.  Tabs are used indent messages according to value of n
def loiter(n):  
    msg = '{}loiter({}): doing nothing for {}s...'
    display(msg.format('\t'*n, n, n))
    sleep(n)
    msg = '{}loiter({}): done.'
    display(msg.format('\t'*n, n))
    return n * 10  

def main():
    display('Script starting.')
    executor = futures.ThreadPoolExecutor(max_workers=3)  
    # submit 5 tasks to executor.  Since we have only 3 wokers 3 will start right away others wait.
    results = executor.map(loiter, range(5))  
    display('results:', results)  
    display('Waiting for individual results:')
    for i, result in enumerate(results):  
        display(f'result {i}: {result}')

if __name__ == '__main__':
    main()
```

The examples above have a lot of error handling omitted for brevity.  Lets cover the full versions.

The repo before has full examples which you can find here and should do a read though here https://github.com/fluentpython/example-code-2e (you also cloned it to your machine)

When reding though pay attention to...

* The common file has the main function and arg handling.
* There is a progress bar implemented with the `tqdm package`
* There is a help cli style screen
* In order to make the tqdm package work we need to use the `futures.as_completed` function so there is another example of that there. 

## Chapter 21. Asynchronous Programming

Python has 3 kinds of coroutines

**Native coroutine** - A coroutine function defined with `async def`.  You can delegate from a native coroutine to another native coroutine using the `await` keyword, similar to how classic coroutines use `yield from`.  The `async def` statement always defines a native coroutine, even if the `await` keyword is not used in its body.  The `await` keyword cannot be used outside of a native coroutine.  

**Classic coroutine** - A generator function that consumes data sent to it via `my_coro.send(data)` calls, and reads that data by using `yield` in an expression.  Classic coroutines can delegate to other classic coroutines using `yield from`.  Classic coroutines cannot be driven by `await`, and are no longer supported by *asyncio*.

**Generator-based coroutine** - A generator function decorated with `@types.coroutine` introduced in Python 3.5.  That decorator makes the generator compatible with the new `await` keyword.

**Asynchronous generator** - A generator function defined with `async def` and using `yield` in its body.  It returns an asynchronous generator object that provides `__anext__`, a coroutine method to retrieve the next item.

The example below will use DNS to lookup domain if they exist.  We are looking up keyword in python as potential domains.  If they do exist a + is prepended when they are printed out.

The `asyncio.get_running_loop` function was added in Python 3.7 for use in coroutines as in the example below.  If there is no running loop, `asyncio.get_running_loop` raised `RuntimeError`.  Its implementation is simpler and faster than `asyncio.get_event_loop`, which may start an event loop if necessary.  Since python 3.10 `asyncio.get_event_loop` is deprecated and will eventually become an alias to `asyncio.get_running_loop`  

In example below we start the event loop with `asyncio.run(main())`

```python
#!/usr/bin/env python3
import asyncio
import socket
from keyword import kwlist

MAX_KEYWORD_LEN = 4  

# probe returns a tuple with domain name and a boolean; True means domain resolved
async def probe(domain: str) -> tuple[str, bool]: 
    #  Get a reference to the asyncio event loop so that we can use it.
    loop = asyncio.get_running_loop()
    try:
        # the loop.getaddrinfo coroutine method returns a five part tuple of parameters to connect to
        # the given address using a socket.  In this example we don't need the result.  If we got it domain resolved
        await loop.getaddrinfo(domain, None)  
    except socket.gaierror:
        return (domain, False)
    return (domain, True)


# main must be a coroutine so we can use await in it.
async def main() -> None:
    # generator to yield python keywords with length up to max len
    names = (kw for kw in kwlist if len(kw) <= MAX_KEYWORD_LEN)  
    # generator to yield domain names with .dev suffix
    domains = (f'{name}.dev'.lower() for name in names)
    # build a list of coroutine objects by invoking the probe coroutine with each domain argument.
    coros = [probe(domain) for domain in domains]  
    # async_io as_completed is a generator that yields coroutines that return the results of the coroutines
    # passed to it in the order they are completed (not the order they were submitted)
    for coro in asyncio.as_completed(coros):  
        # At this point we know the coroutine is done because that' how as_completed works.  Therefore
        # the await expression will not blcok but we need it to get the result from coro.  
        # If coro raised an unhandled exception it would be re raised here.
        domain, found = await coro  
        mark = '+' if found else ' '
        print(f'{mark} {domain}')


if __name__ == '__main__':
    # asyncio.run starts the event loop and returns only when the event loop exists.
    # this is a common patter for scripts that use asyncio: implement main as a coroutine, and drive it with
    # asyncio.run inside the if __main__ == '__main__': block.
    asyncio.run(main()) 
```

You don't have to wait for an awaitable (the `for` keyword works with iterables the `await` with awaitables) you can use `asyncio.create_task(one_coro())` to schedule `one_coro` for concurrent execution without awaiting its return.  If you don't need to be able to cancel a task or you don't need to wait for it there is no need to track it.  

In contrast you use `await other_coro` to run `other_coro` right now and wait for its completion because you need the result before we can proceed.  In the spinner_async.py example earlier we called `res = await slow()` to execute slow and get its result.

Below is the asyncio implementation of the flag download example we have been working with.

```python
import asyncio
import time

from pathlib import Path
from httpx import AsyncClient  

from typing import Callable

BASE_URL = 'https://www.fluentpython.com/data/flags'
DEST_DIR = Path('downloaded')
POP20_CC = ('CN IN US ID BR PK NG BD RU JP '
            'MX PH VN ET EG DE IR TR CD FR').split()

def save_flag(img: bytes, filename: str) -> None:     
    (DEST_DIR / filename).write_bytes(img)

# This needs to be a plain function not a coroutine so it can be passed to and called by the main function.
def download_many(cc_list: list[str]) -> int:    
    # Execute the event loop driving the supervisor(cc_list) coroutine object until it returns.
    # This will block while the event loop runs.  The result of this line is whatever supervisor returns.
    return asyncio.run(supervisor(cc_list))      

async def supervisor(cc_list: list[str]) -> int:
    # Asynchronous HTTP client operations in httpx are methods of AsyncClient, which is also an asynchronous
    # context manager.
    async with AsyncClient() as client:          
        to_do = [download_one(client, cc)
                 for cc in sorted(cc_list)]
        # Wait for the asyncio.gather coroutine, which accepts one or more awaitable arguments and waits for all
        # of them to complete, returning a list of results for the given awaitable in the order they wre submitted.
        res = await asyncio.gather(*to_do)       

    return len(res)                          

# download_one needs to be a native coroutine, so it can await on get_flag which does the HTTP request.
# Then it displays the code of the downloaded flag and saves the image.
async def download_one(client: AsyncClient, cc: str):  
    image = await get_flag(client, cc)
    save_flag(image, f'{cc}.gif')
    print(cc, end=' ', flush=True)
    return cc

# get_flag needs to receive the AsyncClient to make the request
async def get_flag(client: AsyncClient, cc: str) -> bytes:  
    url = f'{BASE_URL}/{cc}/{cc}.gif'.lower()
    
    #The get method of an httpx.AsyncClient instance returns a ClientResponse object that is also an 
    # asynchronous context manager.
    resp = await client.get(url, timeout=6.1, follow_redirects=True)  
    # Network I/O operations are implemented as coroutine methods, so they are driven asynchronously by the 
    # asyncio setup loop.
    return resp.read() 

def main(downloader: Callable[[list[str]], int]) -> None:  
    DEST_DIR.mkdir(exist_ok=True)                          
    t0 = time.perf_counter()                               
    count = downloader(POP20_CC)
    elapsed = time.perf_counter() - t0
    print(f'\n{count} downloads in {elapsed:.2f}s')

if __name__ == '__main__':
    main(download_many)
```

Note that `get_flag` was rewritten here to be a coroutine to use the asynchronous API of HTTPX.  For peak performance with asyncio, we must replace every function that does I/O with an asynchronous version that is activate with `await` or `asyncio.create_task`, so that control is given back to the vent loop while the function waits for I/O.  If you can't rewrite the blocking function as a coroutine, you should run it in a separate thread or process.

### Asynchronous Context Managers.

The regular `with` statement does not support coroutines.  that is why PEP 492 added an `async with` statement which does work with asynchronous context managers.  An asynchronous context manager is an object which implements the `__aenter__` and `__aexit__` methods as coroutines.

There is an asyncio equivalent of the `as_completed` generator function we used in the thread pool example with progress bar.  As a reminder `as_completed` would run a set of threads and return as they are done.

The `asyncio.to_thread` coroutine makes it easy to delegate file I/O to a thread pool provided by *asyncio*.

A **semaphore** is a synchronization primative, more flexible than a lock.  A semaphore can be held by multiple coroutines, with a configurable maximum number.  This makes it ideal to throttle the number of active concurrent coroutines.  There are three `Semaphore` classes in Python's standard library; one in `threading` one in `multiprocessing` and one in `asyncio`.

An `asyncio.Semaphore` has an internal counter that is decremented whenever we `await` on the `.acquire()` coroutine method, and incremented when we call the `.release()` method which is not a coroutine because it never blocks.  The initial value of the `Semaphore` is set when the `Semaphore` is instantiated.  Awaiting on the `.acquire()` causes no delay when the counter is greater than zero, but if the counter is zero, `.acquire()` suspends the awaiting coroutine until some other coroutine call `.release()` on the same `Semaphore`, thus incrementing the counter.  Instead of using these methods directly you can use the `semaphore` as an asynchronous context manager.

```python
semaphore = asyncio.Semaphore(concur_req)
async with semaphore:
    image = await get_flag(client, base_url, cc)
```

The Semaphore.__aenter__ coroutine method awaits for .acquire(), and its __aexit__ coroutine method calls .release(). That snippet guarantees that no more than concur_req instances of get_flags coroutines will be active at any time.

Each of the Semaphore classes in the standard library has a BoundedSemaphore subclass that enforces an additional constraint: the internal counter can never become larger than the initial value when there are more .release() than .acquire() operations.

Side note... Python is not a block-scoped language.  Statements such as `try/except` don't create a local scope in the blocks they manage, but if an `except` clause binds an exception to a variable, like the exception variable the binding only exists within that particular `except` clause.

### Making Multiple Requests for Each Download

The idea hers is that you want to be able to call an async function such as download a flag or in the example we are working with here a json to get country codes from.  Think of how in Javascript you use await to let the code that does a call run for as long as it needs to and comes back.

Below is the new get_country coroutine we are adding

```python
async def get_country(client: httpx.AsyncClient,
                      base_url: str,
                      cc: str) -> str:    
    url = f'{base_url}/{cc}/metadata.json'.lower()
    resp = await client.get(url, timeout=3.1, follow_redirects=True)
    resp.raise_for_status()
    metadata = resp.json()  
    return metadata['country']
```

And here is the updated download_one which now makes two async calls one for the coutnry json and one for flag.

```python
async def download_one(client: httpx.AsyncClient,
                       cc: str,
                       base_url: str,
                       semaphore: asyncio.Semaphore,
                       verbose: bool) -> DownloadStatus:
    try:
        # hold the semaphore to await get_flg
        async with semaphore:  
            image = await get_flag(client, base_url, cc)
        # and again hold the semaphore for country
        # using separate semaphore becuase its good practice to hold semaphores for short time
        async with semaphore:  
            country = await get_country(client, base_url, cc)
    except httpx.HTTPStatusError as exc:
        res = exc.response
        if res.status_code == HTTPStatus.NOT_FOUND:
            status = DownloadStatus.NOT_FOUND
            msg = f'not found: {res.url}'
        else:
            raise
    else:
        # Use country name to create the file name (no spaces just for fun)
        filename = country.replace(' ', '_')  
        await asyncio.to_thread(save_flag, image, f'{filename}.gif')
        status = DownloadStatus.OK
        msg = 'OK'
    if verbose and msg:
        print(cc, msg)
    return status
```

## Delegating Tasks to Executors

In Python reading and writing to I/O blocks the main loop and can become a bottleneck.

You can use the code below to write to a file asynchronously as a coroutine (I believe need to validate)

`await asyncio.to_thread(save_flag, image, f'{cc}.gif')`

Now this is only available with Python 3.9 and later prior you would need to replace the above with the following lines...

```python
# get reference to the event loop.
loop = asyncio.get_running_loop() 
# first argument is the executor to use.  Passing None select the default
# ThreadPoolExecutor that is always available in the asyncio event loop.
# You can pass positional arguments to the function to run.  KW is possibl
# check functool.partial docs.        
loop.run_in_executor(None, save_flag,image, f'{cc}.gif')
```

The main reason to pass an explicit `Executor` to `loop.run_in_executor` is to employ a `ProcessPoolExecutor` if the function to execute is CPU intensive so that it will run in a different process avoiding contention for the GIL.  Because of the high startup costs its better to start the `ProcessPoolExecutor` and pass it to coroutines that need to use it.  Generally the stuff that runs in `ProcessPoolExecutor` cannot be canceled.

### Asynchronous Generator Functions

You can implement an asynchronous iterator by writing a class with __anext__ and __aiter__, but there is a simpler way: write a function declared with async def and use yield in its body. This parallels how generator functions simplify the classic Iterator pattern.

### Experimenting with Python's async console

You can start the python REPL with async features enabled with the command `python -m asyncio`

### Implementing an asynchronous generator

```python
import asyncio
import socket
from collections.abc import Iterable, AsyncIterator
from typing import NamedTuple, Optional


class Result(NamedTuple):  
    domain: str
    found: bool


OptionalLoop = Optional[asyncio.AbstractEventLoop]  


async def probe(domain: str, loop: OptionalLoop = None) -> Result:  
    if loop is None:
        loop = asyncio.get_running_loop()
    try:
        await loop.getaddrinfo(domain, None)
    except socket.gaierror:
        return Result(domain, False)
    return Result(domain, True)


async def multi_probe(domains: Iterable[str]) -> AsyncIterator[Result]:  
    loop = asyncio.get_running_loop()
    coros = [probe(domain, loop) for domain in domains]  
    for coro in asyncio.as_completed(coros):  
        result = await coro
        # yield the result this one line makes multi_probe an asynchronous generator.
        yield result  

```

The for loop in example above can be more consice

```python
for coro in asyncio.as_completed(coros):
        yield await coro
```

python parses that as `yield (await coro)` so it works.

below is the code that uses the above...

```python
#!/usr/bin/env python3
import asyncio
import sys
from keyword import kwlist

from domainlib import multi_probe


async def main(tld: str) -> None:
    tld = tld.strip('.')
    names = (kw for kw in kwlist if len(kw) <= 4)  
    domains = (f'{name}.{tld}'.lower() for name in names)  
    print('FOUND\t\tNOT FOUND')  
    print('=====\t\t=========')
    # Asynchronously iterate over multi_probe(domains)
    async for domain, found in multi_probe(domains):  
        indent = '' if found else '\t\t'  
        print(f'{indent}{domain}')


if __name__ == '__main__':
    if len(sys.argv) == 2:
        # Run the main coroutine with the given command line argument.
        asyncio.run(main(sys.argv[1]))  
    else:
        print('Please provide a TLD.', f'Example: {sys.argv[0]} COM.BR')
```

### Asynchronous generators as context managers

Writing our own asynchronous context managers is not a frequent programming task, but if you need to write one, consider using the @asynccontextmanager decorator added to the contextlib module in Python 3.7. That’s very similar to the @contextmanager decorator we studied in “Using @contextmanager”.


```python
from contextlib import asynccontextmanager

# The decorated function must be an asynchronous generator
@asynccontextmanager
async def web_page(url):  
    loop = asyncio.get_running_loop()
    # Suppose download_webpage is a blocking function using the requests library; we 
    # run it in a separate thread to avoid blocking the event loop. 
    data = await loop.run_in_executor(  
        None, download_webpage, url)
    # All lines before this yield expression will become the __aenter__ coroutine method
    # of the asynchronous context manager built by the decorator.  The value of data will be
    # bound to the data variable after the as clause in the async with statement below.
    yield data
    # Lines after the yield will beomce the __aexit__ coroutine method.
    # Here, another blocking call is delegated tot he thread executor.                       
    await loop.run_in_executor(None, update_stats, url)  

# use web page with async with.
async with web_page('google.com') as data:  
    process(data)
```

### Asynchronous generators versus native coroutines

* Both are declared with `async def`

* An asynchronous generator will have a `yield` expression in its body - that's what makes it a generator.  A native coroutine never contains a `yield`.

* A native coroutine may return some value other than `None`. An asynchronous generator can only use empty `return` statements.

* Native coroutine are awaitable: they can be driven by `await` expressions or passed to one of the many `asyncio` functions that take awaitable arguments, such as `create_task`.  Asynchronous generators are not awaitable.  They are asynchronous iterables, driven by `async for` or by asynchronous comprehensions.

### Async Comprehension and Async Generator Expressions

An asynchronous generator expression can be defined anywhere in your program, but it can only be consumed inside a native coroutine or asynchronous generator function.

```python
# This is the multi_probe function we wrote in earlier example
>>> from domainlib import multi_probe
>>> names = 'python.org rust-lang.org golang.org no-lang.invalid'.split()
# The use of async for makes this an asynchronous generator expression.
>>> gen_found = (name async for name, found in multi_probe(names) if found)  
>>> gen_found
# The asynchronous generator expression builds an async_generator object - exactly the same
# type of object returned by an asynchronous generator function like multi_probe.
<async_generator object <genexpr> at 0x10a8f9700>  
# The asynchronous generator object is driven by async for statement, which in turn can
# only appear inside an async def body or in the asynchronous REPL console.
>>> async for name in gen_found:  
...     print(name)
...
golang.org
python.org
rust-lang.org
```

#### Asynchronous comprehensions

```python
result = []
async for i in aiter():
    if i % 2:
        result.append(i)

# The above code can be written as...

result = [i async for i in aiter() if i % 2]
```

Given a native coroutine `fun` we should be able to write `result = [await fun() for fun in funcs]`

Using `await` in a list comprehension is similar to using `asyncio.gather` but `gather` gives you more control over exception handling thanks to the special return exceptions argument.  Its recommended to always set `return_exceptions=True` (the default is False).

You can use `async for` and `await` in list comprehension as well as for `dict` and `set`.

Below is an example of dict comprehesnion.

```python
>>> {name: found async for name, found in multi_probe(names)}
{'golang.org': True, 'python.org': True, 'no-lang.invalid': False,
'rust-lang.org': True}
```

Here is another example not that you can use `await` before the `for` or `async for` and after the `if`

```python
>>> {name for name in names if (await probe(name)).found}
{'rust-lang.org', 'python.org', 'golang.org'}
```

You may want to take a look into the `curio` library that makes some of these concepts easier to develop with.  It has features that allow it to run in a thread along with asyncio in another thread.

### Type Hinting and Asynchronous Objects

If you need to annotate a parameter that takes a coroutine object, then the generic type is:
These were added in Python 3.5.  With Python 3.9 and newer use the collections.abc equivalents of these.

```python
class typing.Coroutine(Awaitable[V_co], Generic[T_co, T_contra, V_co]):
...
class typing.AsyncContextManager(Generic[T_co]):
    ...
class typing.AsyncIterable(Generic[T_co]):
    ...
class typing.AsyncIterator(AsyncIterable[T_co]):
    ...
class typing.AsyncGenerator(AsyncIterator[T_co], Generic[T_co, T_contra]):
    ...
class typing.Awaitable(Generic[T_co]):
    ...
```

### How Async Works and How It Doesn't

