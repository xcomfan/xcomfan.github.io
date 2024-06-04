---
layout: page
title: "Python Generator Functions"
permalink: /python/generator_functions
---

## Generator functions

You use a generator function when you need to return a value from a big set of data and you only need the next value, not the entire list. This can be very memory efficient as you are not computing and returning an entire set of data, but just the next value you need.

The `yield` keyword is used to return the values in this manner. `yield` produces a value but suspends the function until the next call to `__next__()`.

One important note is that generator functions and generator expressions are their own iterators and thus you can't have multiple iterators at multiple positions like you can in some built in structures. If you want to scan a generator's values multiple times you must either create a new generator for each scan or build a re-scannable list out of the generator's values.

```python
>>> def gensquares(N):
...   for i in range(N):
...     yield i ** 2
...
>>> for i in gensquares(5):
...   print(i, end=' : ')
...
0 : 1 : 4 : 9 : 16 : >>>
>>> x = gensquares(4)
>>> x
<generator object gensquares at 0xffffa1464040>
>>> next(x)
0
>>> next(x)
1
>>> next(x)
4
>>> next(x)
9
>>> next(x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

A generator function can be a much more convenient way to write an iterator as you don't have to worry about the iterator protocol (`__next__`, `__iter__`, etc).

## Generator expressions

A generator expression has some important differences from list comprehension.

* Generator expressions do not construct a list, they construct a generator object
* Only useful purpose for generator comprehensions is iteration.
* Once generator expression is consumed it cannot be re-used.

below is an example of a generator comprehension vs a list.  The general syntax for a generator expression is `(expression for i in s if condition)`

```python
>>> a = [1,2,3,4]
>>> b = [2*x for x in a]
>>> b
[2, 4, 6, 8]
>>> c = (2*x for x in a)
>>> c
<generator object <genexpr> at 0xffff8e824120>
```

Both generator expressions and generator functions give us an object which can be iterated over with a `for` loop

## Yield from expression

`yield from` is a way to delegate iteration.  Small example below for explanation.

```python
>>> def count_down(n):
...   while n > 0:
...     yield n
...     n -= 1
... 
>>> def count_up(stop):
...   n = 1
...   while n < stop:
...     yield n
...     n += 1
... 
>>> def count_up_and_down(n):
...   yield from count_up(n)
...   yield from count_down(n)
... 
>>> for x in count_up_and_down(3):
...   print(x)
... 
1
2
3
2
1
>>>
```

## Shutting down generators

Generators can be shut down using `.close()` below is an example

```python
 import time

def follow(the_file):
  the_file.seek(0, os.SEEK_END) # End of file
  while True:
    line = the_file.readline()
    if not line:
      time.sleep(0.1)    # Sleep briefly
      continue
    yield line

lines = follow(open("access-log"))
for i, line in enumerate(lines):
  print(line, end='')
  if i == 10:
    lines.close()
```

When `.close()` is used inside the generator a `GeneratorExit` exception is raised.  This is a good opportunity to clean up resources. Note that you cannot ignore `GeneratorExit` to continue the generator. You will get an error.

```python
import time
def follow(the_file):
  the_file.seek(0, os.SEEK_END)
  try:
    while True:
      line = the_file.readline()
      if not line:
        time.sleep(0.1)
        continue
      yield line
  except GeneratorExit:
    print("Follow: Shutting down")
```

## Generators can receive values using .send()

This form of a generator is a co-routine.  Its sometimes called a reverse-generator. Coroutines can be though of as receivers or consumers. They receive values sent to them. To get a co-routine to run properly, you have to ping it with a `.send(None)` operation first. This advances to the first yield where it will receive its first value. The having to call `.send(None)` can be more elegantly handled by using a decorator which is shown in the example below.

```python
def consumer(func):
    def start(*args, **kwargs):
        c = func(*args, **kwargs)
        c.send(None)
        return c
    return start

# this function receives values rather than generating them
@consumer
def recv_count():
    try:
        while True:
            n = yield # Yield expression
            print("T-minus", n)
    except GeneratorExit:
        print("Kaboom!")

r = recv_count()
for i in range(5, 0, -1):
    r.send(i)
r.close()

'''
Yields the output
T-minus 5
T-minus 4
T-minus 3
T-minus 2
T-minus 1
Kaboom!
'''
```

## Processing data files with generators

The following contents is based on the excellent talk by Based on the [excellent talk](https://www.dabeaz.com/generators) by David M. Beazley. I just worked though the examples in his talk and summarized notes for myself here.

### Problem statement

The code in the following section is a way of using generators to create a data pipeline which can process hundreds of httpd access logs in a directory. Below is a sample of the log file content being processed by the program for reference. This is not a real log just an example.

```text
81.107.39.38 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 7587
81.107.39.39 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 133
81.107.39.38 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 23903
81.107.39.40 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 9732
81.107.39.38 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 2934
81.107.39.41 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 124
81.107.39.38 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 9923
81.107.39.41 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 485667
81.107.39.38 - - [21/Apr/2021:00:08:59 -0600] "GET ..." 200 9875860
81.107.39.42 - - [21/Apt/2021:00:08:59 -0600] "GET ..." 200 1048957856
```

### Implementation

```python
import gzip, bz2, re
from pathlib import Path


# NOTE: I am pretty sure this will not close the files. In context of this example this makes sense
# but in real implementation the file handles probably should be kept out of the pipeline.
# This is also a good example of handling multiple file type options
def open_file_generator(paths):

    """Open a sequence of paths returning open file objects"""

    for path in paths:
        if path.suffix == ".gz":
            yield gzip.open(path, "rt")
        if path.suffix == ".bz2":
            yield bz2.open(path, "rt")
        else:
            yield open(path, 'rt')

def concat_files_generator(sources):

    """concatenate items from one or more sources into a single sequence of items"""

    for src in sources:
        # See yield from explanation above if this does not make sense
        yield from src

def field_map(dict_seq, name, func):

  """helper function for running a function on a property of a dictionay"""

    for d in dict_seq:
        d[name] = func(d[name])
        yield d

# Just example values form my system.
log_dir = "/home/ubuntu/python_practice/generator_pipeline"
# Path.rglob is using a generator under the hood thus fits neatly in our pipeline of generators model
file_names = Path(log_dir).rglob("sample*")

log_files = open_file_generator(file_names)
log_lines = concat_files_generator(log_files)

log_pattern = r"(\S+) (\S+) (\S+) \[(.*?)\] (\S+) (\S+) (\S+) (\S+)"
compiled_pattern = re.compile(log_pattern)

# groups here is the groups returned by re.match
groups = (compiled_pattern.match(line) for line in log_lines)
tuples = (g.groups() for g in groups if g)

# convert the tuples and col_names to dictionaries to make working with data easier
col_names = ('host','referrer','user','datetime', 'method','request','status', 'bytes')
log_contents = (dict(zip(col_names, t)) for t in tuples)

log = field_map(log_contents, "status", int)
log = field_map(log_contents, "bytes", lambda s: int(s if s != "_" else 0))

# Now that we have the one or more log files as a generator of dictionaries we can do some fun queries

# find which requests are throwing 404s
stat404 = {r['request'] for r in log if r['status'] == 404}

# print all requests that transferred more than a megabyte
large = (r for r in log if r['bytes'] > 1000000)

# find the largest data transfer
print("%d %s" % max((r['bytes'], r['request']) for r in log))

# collect all unique host IP addresses
hosts = { r['host'] for r in log }

# find out who has been hitting robots.txt
addresses = { r['host'] for r in log if 'robots.txt' in r['request'] }
import socket
for addr in addresses:
  try:
    print(socket.gethostbyaddr(addr)[0])
  except socket.herror:
    print(addr)
```

### Explanation and notes on above solution

Think of the solution above as a processing pipeline where each step is defined by iteration/generation

{% mermaid %}
 flowchart LR

   access-log-->wwwlog-->bytecolumn-->bytes_sent-->sum-->total

{% endmermaid %}

There are some benefits to this approach over a regular for loop implementation.

* Can be a little bit faster than the for loop implementation.
* Less code
* relatively easy to read
* does not require creating large lists in memory thus can be used on very large data sets
* Can plug in various "pipeline" components with this approach
* Similar to Unix approach of performing complex tests by piping data
* Generators decouple iteration from the code that uses the results of the iteration.  In the last line of the program we are performing a calculation on a sequence of lines. It does not matter where or how those lines are generated.  Thus we can plug any number of components together up front as long as they eventually produce a line sequence.
* This paradigm is not limited to files. You can use this with any objects such as sockets to send data across network or just about any other uses.

#### packaging

Below is an example of multiple pipeline stages inside a function which becomes a general purpose component that can be used as a single element in other pipelines.

```python
from pathlib import Path

def lines_from_dir(filepat, dirname):
 names = Path(dirname).rglob(filepat)
 files = gen_open(names)
 lines = gen_cat(files)
 return lines
```

Below is another example of parsing an Apache log into dicts.

```python
def apache_log(lines):
 groups = (logpat.match(line) for line in lines)
 tuples = (g.groups() for g in groups if g)
 colnames = ('host','referrer','user','datetime','method', 'request','proto','status','bytes')
 log = (dict(zip(colnames, t)) for t in tuples)
 log = field_map(log, "bytes",
 lambda s: int(s) if s != '-' else 0)
 log = field_map(log, "status", int)
 return log

# example use
lines = lines_from_dir("access-log*", "www")
log = apache_log(lines)
for r in log:
  print(r)
```

#### Things to consider

* When creating pipeline components it's critical to focus on the inputs and outputs.
* You will get the most flexibility when you use a standard set of data types.
* It is simpler to have a bunch of components that all operate on dictionaries or to have components that require inputs/outputs to be different kinds of user-defined instances.
* Error handling with this program style is tricky because you have lots of components chained together.
* Need to pay careful attention to debugging, reliability and other issues.

### Processing infinite data

The code below uses a generator to reproduce the functionality fo `tail -f`

```python
import time
import os

def follow(the_file):
    the_file.seek(0, os.SEEK_END) # End-of-file
    while True:
        line = the_file.readline()
        if not line:
            time.sleep(0.1) # Sleep briefly
            continue
        yield line

if __name__ == "__main__":
    log_file = open("sample.log")
    log_lines = follow(log_file)

    for line in log_lines:
        print(line, end="")
```

We can use this generator in our log processing pipeline utilities with a caveat that functions that consume the entire iterable such as min, max, sum etc. will not complete.  This is an example that we can write processing steps that operate on an infinite data stream.

```python
log_file = open("access-log")
log_lines = follow(log_file)
log = apache_log(loglines) # recall that apache_log consumes a generator

# print out all 404 requests as they happen
r404 = (r for r in log if r['status'] == 404)
for r in r404:
  print(r['host'], r['datetime'], r['request'])
```

## Debugging generators

Any single argument function is easy to turn into a generator function. For example...

```python
def generate(func):
  def gen_func(s):
    for item in s:
      yield func(item)
  return gen_func

# example
gen_sqrt = generate(math.sqrt)
for x in gen_sqrt(range(100)):
  print(x)
```

### Debug tracing

A debugging function that will print items going through a generator

```python
def trace(source):
  for item in source:
    # can you logging module here
    print(item)
    yield item

# This can be placed around any generator
lines = follow(open("access-log"))
log = trace(apache_log(lines))
```

### Recording the last item generated in a generator

```python
class storeLast(object):
  def __init__(self, source):
    self.source = sources
  
  def __next__(self):
    item = self.source.__next__()
    self.last = item
    return item
  
  def __iter__(self:
    return self

# this can be wrapped around a generator
lines = storeLast(follow(open("access-log")))
log = apache_log(lines)

for r in log:
  print(r)
  print(lines.last)
```

## Yield as print

Generator functions can use `yield` like a print statement. This is useful if you are producing I/O output but you want flexibility in how it gets handled.

This technique of producing output leaves the exact output method unspecified so the code is not hardwired to use files, sockets or any other specific kind of output.

```python
def print_count(n):
  yield "Hello World\n"
  yield "\n"
  yield "Look at me count to %d\n" % n
  for i in range(n):
    yield "   %d\n" % i
  yield "I'm done!\n"

# generate the output
out = print_count(10)

# Turn it into a string
out_str = "".join(out)

# Write it to a file
f = open("out.txt", "w")
for chunk in out:
  f.write(chunk)

# Send it across a network socket
for chunk in out:
  s.sendall(chunk)
```
