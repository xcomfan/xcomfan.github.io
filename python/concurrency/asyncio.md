---
layout: page
title: "Async IO"
permalink: /python/concurrency/asyncio
---

## What is asyncio

Async IO in Python is actually a single threaded single process design. It takes long waiting periods in which functions would otherwise be blocking and allows other functions to run during that downtime.

Async IO is ideas for when you have multiple IO-bound tasks such as

* Network IO
* Read/write operations where you want to write and move on, but not have to worry about managing locks.

## The asyncio package and async/await

At the hear of async IO are coroutines.  A coroutine is a specialized version of a Python generator function. A coroutine can suspend its execution before reaching return, and it can indirectly pass control to another coroutine for some time.

### Hello World of async IO

```python
import asyncio

async def count():
    print("One")
    await asyncio.sleep(1)
    print("Two")

async def main():
    await asyncio.gather(count(), count(), count())

if __name__ == "__main__":
    import time
    s = time.perf_counter()
    asyncio.run(main())
    elapsed = time.perf_counter() - s
    print(f"executed in {elapsed:0.2f} seconds.")
```

The above program will output below.  Notice that execution took one second whereas without asyncio it would take 3.

```text
One
One
One
Two
Two
Two
executed in 1.00 seconds.
```

The order of the output above demonstrates the core of async IO. Each of the calls to `count()` is driven by a single event loop. When each task reaches the `await asyncio.sleep(1)`, the function is effectively notifying the event loop that its going to sleep in the example here but making a long call such as a network request in a real scenario; and that the event loop can run something else in the meantime.

***Note*** we are using `asyncio.sleep()` here as `time.sleep()` is not compatible with async IO.  In general you need to be cautious of libraries being used with async IO as libraries not compatible with async IO will be blocking while executing.

### Rules of async IO

* The syntax `async def` introduces either a **native coroutine** or an **asynchronous generator**.  

* The expressions `with async` and `async for` are also valid.

* The keyword `await` passes function control back to the event loop.  It suspends the execution of the surrounding coroutine and allows something else to run.

* A function that you define with `async def` is a coroutine. It may use `await` `return`, or `yield` but all of these are optional.  To call a coroutine function, you must `await` it to get its results.

* You can use `yield` in in an aysnc def block.  This creates an **asynchronous generator** which you iterate over with using `async for`.

* Anything defined with `async def` cannot use `yield from`.  This will raise a `SyntaxError`.

* You cannot use `await` outside of a `async def` function.  This will raise a `SyntaxError`.  You can only use `await` in the body of a coroutine.

## The event loop and asyncio.run()

You can think of an event loop as a `while True` loop that monitors coroutines, taking feedback on what's idle, and looking for things that can be executed in the meantime. It is able to wake up a coroutine when whatever that coroutine is waiting on is becomes available.

`asyncio.run()` is responsible for getting the event loop running tasks until they are complete then closing the vent loop. A more granular option for controlling rhe vent loop is below but in most cases `asyncio.run()` should be sufficient.

```python
loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

### some important points about even loop

* Coroutines don't do much on their own until they are tied to an event loop. If you call a function declared with `async def` it won't do anything and just return the coroutine object. It needs to be called/started with `asyncio.run()`

* By default an asyncio event loop runs in a single thread on a single CPU core. Usually this is sufficient but you can mis threading and asyncio to have it run on multiple threads.

* Event loops are pluggable. If you want to you can implement your own even loop (one example of this is the `uvloop` package which in an implementation of the even loop in Cython). You can use any working implementation of even loop independent of the coroutines themselves.

### Top level asyncio functions

You can create a task with `create_task` to schedule the execution of a coroutine object, followed by `asyncio.run()`. ***Note*** in example below if you don't `await` on `t` within `main()`, it may finish before `main()` itself signals that it is complete.  Because `asyncio.run(main())` calls `loop.run_until_complete(main())` without the `await t` present the event loop only ares if `main()` is done not the tasks that get created within main.  Without the `await t` the loop's other tasks will be cancelled, possibly before they are completed.  If you need to get a list of currently pending tasks you can use `asyncio.Task.all_tasks()`

```python
>>> import asyncio

>>> async def coro(seq) -> list:
...     """'IO' wait time is proportional to the max element."""
...     await asyncio.sleep(max(seq))
...     return list(reversed(seq))
...
>>> async def main():
...     # This is a bit redundant in the case of one task
...     # We could use `await coro([3, 2, 1])` on its own
...     t = asyncio.create_task(coro([3, 2, 1]))  # Python 3.7+
...     await t
...     print(f't: type {type(t)}')
...     print(f't done: {t.done()}')
...
>>> t = asyncio.run(main())
t: type <class '_asyncio.Task'>
t done: True
```

`asyncio.gather()` is mean to put a collections of coroutines (futures) into a single future.  As a result, it returns a single future object, and, if you `await asyncio.gather()` and specify multiple tasks and coroutines you are waiting for all of them to be completed.  The result will be a list of results across the inputs.  Alternatively you can loop over `asyncio.as_completed()` to get tasks as they are completed in the order of completion.  The function returns an iterator that yields tasks as they finish.

```python
>>> import asyncio
>>> async def coro(seq) -> list:
...     """'IO' wait time is proportional to the max element."""
...     await asyncio.sleep(max(seq))
...     return list(reversed(seq))
...
>>> async def main():
...     t = asyncio.create_task(coro([3, 2, 1]))
...     t2 = asyncio.create_task(coro([10, 5, 0]))
...     print('Start:', time.strftime('%X'))
...     for res in asyncio.as_completed((t, t2)):
...         compl = await res
...         print(f'res: {compl} completed at {time.strftime("%X")}')
...     print('End:', time.strftime('%X'))
...     print(f'Both tasks done: {all((t.done(), t2.done()))}')
...
>>> a = asyncio.run(main())
Start: 09:49:07
res: [1, 2, 3] completed at 09:49:10
res: [0, 5, 10] completed at 09:49:17
End: 09:49:17
Both tasks done: True
```

## Full example program using async io

The example below is a web scraping URL collector which uses `aiohttp` as the `requests` package does not support async IO.

The program below reads a sequence of URLs from a local file `urls.txt`, sends `GET` requests for the URLs and decodes the resulting content. It then searches for URLs in the body of the HTML response and writes them to `foundurls.txt`

below is a sample of urls.txt.  The second one is intended to fail.

```bash
$ cat urls.txt
https://regex101.com/
https://docs.python.org/3/this-url-will-404.html
https://www.nytimes.com/guides/
https://www.mediamatters.org/
https://1.1.1.1/
https://www.politico.com/tipsheets/morning-money
https://www.bloomberg.com/markets/economics
https://www.ietf.org/rfc/rfc2616.txt
```

Some notes on the program.

* In the `parse()` function the second portion is blocking, but it consists fo a quick regex match and ensuring that links discovered are made into absolute paths.  Just a note here that any line in a coroutine unless that line uses `yield`, `await` or `return` will block.  If this operation was a more expensive process you should consider running it with `loop.run_in_executor()`.

* The package `aiofiles` is the async IO compatible package for writing to files.

```python
import asyncio
import logging
import re
import sys
from typing import IO
import urllib.error
import urllib.parse

import aiofiles
import aiohttp
from aiohttp import ClientSession

logging.basicConfig(
    format="%(asctime)s %(levelname)s:%(name)s: %(message)s",
    level=logging.DEBUG,
    datefmt="%H:%M:%S",
    stream=sys.stderr,
)
logger = logging.getLogger("areq")
logging.getLogger("chardet.charsetprober").disabled = True

HREF_RE = re.compile(r'href="(.*?)"')

async def fetch_html(url: str, session: ClientSession, **kwargs) -> str:
    """GET request wrapper to fetch page HTML.

    kwargs are passed to `session.request()`.
    """

    resp = await session.request(method="GET", url=url, **kwargs)
    resp.raise_for_status()
    logger.info("Got response [%s] for URL: %s", resp.status, url)
    html = await resp.text()
    return html

async def parse(url: str, session: ClientSession, **kwargs) -> set:
    """Find HREFs in the HTML of `url`."""
    found = set()
    try:
        html = await fetch_html(url=url, session=session, **kwargs)
    except (
        aiohttp.ClientError,
        aiohttp.http_exceptions.HttpProcessingError,
    ) as e:
        logger.error(
            "aiohttp exception for %s [%s]: %s",
            url,
            getattr(e, "status", None),
            getattr(e, "message", None),
        )
        return found
    except Exception as e:
        logger.exception(
            "Non-aiohttp exception occured:  %s", getattr(e, "__dict__", {})
        )
        return found
    else:
        for link in HREF_RE.findall(html):
            try:
                abslink = urllib.parse.urljoin(url, link)
            except (urllib.error.URLError, ValueError):
                logger.exception("Error parsing URL: %s", link)
                pass
            else:
                found.add(abslink)
        logger.info("Found %d links for %s", len(found), url)
        return found

async def write_one(file: IO, url: str, **kwargs) -> None:
    """Write the found HREFs from `url` to `file`."""
    res = await parse(url=url, **kwargs)
    if not res:
        return None
    async with aiofiles.open(file, "a") as f:
        for p in res:
            await f.write(f"{url}\t{p}\n")
        logger.info("Wrote results for source URL: %s", url)

async def bulk_crawl_and_write(file: IO, urls: set, **kwargs) -> None:
    """Crawl & write concurrently to `file` for multiple `urls`."""
    async with ClientSession() as session:
        tasks = []
        for url in urls:
            tasks.append(
                write_one(file=file, url=url, session=session, **kwargs)
            )
        await asyncio.gather(*tasks)

if __name__ == "__main__":
    import pathlib
    import sys

    assert sys.version_info >= (3, 7), "Script requires Python 3.7+."
    here = pathlib.Path(__file__).parent

    with open(here.joinpath("urls.txt")) as infile:
        urls = set(map(str.strip, infile))

    outpath = here.joinpath("foundurls.txt")
    with open(outpath, "w") as outfile:
        outfile.write("source_url\tparsed_url\n")

    asyncio.run(bulk_crawl_and_write(file=outpath, urls=urls))
```

## Async for and async generator comprehensions

Along with the plain `async await` Python enables `async for` to iterate over an **asynchronous iterator**. The purpose of the asynchronous iterator is for it to be able to call asynchronous code at each state its iterated over.  A natural extension of this concept is an **asynchronous generator**.  Python enables **asynchronous comprehension** with `async await`.

The key thing is that neither **asynchronous generators** nor **asynchronous comprehensions** make the iteration concurrent. All they do is provide the look and feel of their synchronous version, but with the ability for the loop in question to give up control to the event loop so that another coroutine could run. The async for and async with statements are only needed to the extent that using plain for or with would “break” the nature of await in the coroutine.

## Async IO design patterns

### Chaining coroutines

A key feature of coroutines is that they can be chained together. A coroutine is awaitable, so another coroutine can `await` it.  This allows you to break programs into smaller, manageable, recyclable coroutines.  Basic example of this pattern below.

```python
import asyncio
import random
import time

async def part1(n: int) -> str:
    i = random.randint(0, 10)
    print(f"part1({n}) sleeping for {i} seconds.")
    await asyncio.sleep(i)
    result = f"result{n}-1"
    print(f"Returning part1({n}) == {result}.")
    return result

async def part2(n: int, arg: str) -> str:
    i = random.randint(0, 10)
    print(f"part2{n, arg} sleeping for {i} seconds.")
    await asyncio.sleep(i)
    result = f"result{n}-2 derived from {arg}"
    print(f"Returning part2{n, arg} == {result}.")
    return result

async def chain(n: int) -> None:
    start = time.perf_counter()
    p1 = await part1(n)
    p2 = await part2(n, p1)
    end = time.perf_counter() - start
    print(f"-->Chained result{n} => {p2} (took {end:0.2f} seconds).")

async def main(*args):
    await asyncio.gather(*(chain(n) for n in args))

if __name__ == "__main__":
    import sys
    random.seed(444)
    args = [1, 2, 3] if len(sys.argv) == 1 else map(int, sys.argv[1:])
    start = time.perf_counter()
    asyncio.run(main(*args))
    end = time.perf_counter() - start
    print(f"Program finished in {end:0.2f} seconds.")
```

The above programs output is below.  Note how part1 sleeps for a variable amount of time, and part2 begins working with the results as they become available.

```text
$ python3 chained.py 9 6 3
part1(9) sleeping for 4 seconds.
part1(6) sleeping for 4 seconds.
part1(3) sleeping for 0 seconds.
Returning part1(3) == result3-1.
part2(3, 'result3-1') sleeping for 4 seconds.
Returning part1(9) == result9-1.
part2(9, 'result9-1') sleeping for 7 seconds.
Returning part1(6) == result6-1.
part2(6, 'result6-1') sleeping for 4 seconds.
Returning part2(3, 'result3-1') == result3-2 derived from result3-1.
-->Chained result3 => result3-2 derived from result3-1 (took 4.00 seconds).
Returning part2(6, 'result6-1') == result6-2 derived from result6-1.
-->Chained result6 => result6-2 derived from result6-1 (took 8.01 seconds).
Returning part2(9, 'result9-1') == result9-2 derived from result9-1.
-->Chained result9 => result9-2 derived from result9-1 (took 11.01 seconds).
Program finished in 11.01 seconds.
```

### Using a queue

The `asyncio` package provides [queue classes](https://docs.python.org/3/library/asyncio-queue.html) that let you implement a producer consumer pattern that is async IO compatible.  Each producer may add multiple items to the queue at random times and a group of consumers can retrieve items from the queue as they show up without the two processes needing to signal each other.  

Basic example of this pattern below where a producer puts from 1 to 5 items into the queue with a random number and the time the item was added to queue. Consumer pulls an item out, calculates the elapsed time.

Few notes about this example.

* There needs to be a signal to the consumers that production is done. Otherwise `await q.get()` will hang indefinitely, because the queue will have been fully processed, but consumers won't have any idea that production is complete.

* The `await` on `q.join()` blocks until all items in the queue have been received and processed and then cancel the consumer tasks, which would otherwise hang up and wait endlessly for additional queue items to appear.

```python
import asyncio
import itertools as it
import os
import random
import time

async def makeitem(size: int = 5) -> str:
    return os.urandom(size).hex()

async def randsleep(caller=None) -> None:
    i = random.randint(0, 10)
    if caller:
        print(f"{caller} sleeping for {i} seconds.")
    await asyncio.sleep(i)

async def produce(name: int, q: asyncio.Queue) -> None:
    n = random.randint(0, 10)
    for _ in it.repeat(None, n):  # Synchronous loop for each single producer
        await randsleep(caller=f"Producer {name}")
        i = await makeitem()
        t = time.perf_counter()
        await q.put((i, t))
        print(f"Producer {name} added <{i}> to queue.")

async def consume(name: int, q: asyncio.Queue) -> None:
    while True:
        await randsleep(caller=f"Consumer {name}")
        i, t = await q.get()
        now = time.perf_counter()
        print(f"Consumer {name} got element <{i}>"
              f" in {now-t:0.5f} seconds.")
        q.task_done()

async def main(nprod: int, ncon: int):
    q = asyncio.Queue()
    producers = [asyncio.create_task(produce(n, q)) for n in range(nprod)]
    consumers = [asyncio.create_task(consume(n, q)) for n in range(ncon)]
    await asyncio.gather(*producers)
    await q.join()  # Implicitly awaits consumers, too
    for c in consumers:
        c.cancel()

if __name__ == "__main__":
    import argparse
    random.seed(444)
    parser = argparse.ArgumentParser()
    parser.add_argument("-p", "--nprod", type=int, default=5)
    parser.add_argument("-c", "--ncon", type=int, default=10)
    ns = parser.parse_args()
    start = time.perf_counter()
    asyncio.run(main(**ns.__dict__))
    elapsed = time.perf_counter() - start
    print(f"Program completed in {elapsed:0.5f} seconds.")
```

## References

A lot of the content I made notes on here is from the [Real Python async IO tutorial](https://realpython.com/async-io-python/#libraries-that-work-with-asyncawait)
