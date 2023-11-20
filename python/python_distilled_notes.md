## Python Basics

When you use Python in REPL the results of the last statement are available in the `_` variable.  This is only available in REPL.

`divmod(x,y)` Returns (x // y, x % y)

Immediately adjacent string literals are concatenated into a single string.  The two examles below are quivalent.

```python
print('''Content-type: text/html

<h1> Hello World </h1>
Click <a href="http://www.python.org">here</a>.
''')

print(
'Content-type: text/html\n'
'\n'
'<h1> Hello World </h1>\n'
'Clock <a href="http://www.python.org">here</a>\n'
)


String methods just to add to your reference you know this stuff mostly...

Method Description

s.endswith(prefix [,start [,end]]) Checks whether a string ends with prefix.

s.find(sub [, start [,end]]) Finds the first occurrence of the specified substring sub or -1 if not found.

s.lower() Converts to lowercase.

s.replace(old, new [,maxreplace]) Replaces a substring.

s.split([sep [,maxsplit]]) Splits a string using sep as a delimiter. maxsplit is the maximum number of splits to perform.

s.startswith(prefix [,start [,end]]) Checks whether a string starts with prefix.

s.strip([chrs]) Removes leading and trailing whitespace or characters supplied in chrs.

s.upper() Converts a string to uppercase.

Although str() and repr() both create strings, their output is often different. str() produces the output that you get when you use the print() function, whereas repr() creates a string that you type into a program to exactly represent the value of an object. For example:

The format() function is used to convert a single value to a string with a specific formatting applied. For example

```python
>>> x = 12.34567
>>> format(x, '0.2f')
'12.35'
>>>
```

Sometimes you might want to read data typed interactively in the console. To do that, use the input() function. For example

The input() function returns all of the typed text up to the terminating newline, which is not included

```python
name = input('Enter your name : ')
print('Hello', name)
```

The elements of a set are typically restricted to immutable objects. For example, you can make a set of numbers, strings, or tuples. However, you can’t make a set containing lists. Most common objects will probably work with a set, however—when in doubt, try it.

If working with existing data, you can also create a set using a set comprehension. For example, this statement turns all of the stock names from the data in the previous section into a set: `names = { s[0] for s in portfolio }`

To create an empty set `r = set()`

New items can be added to a set sing `add()` or `update()`

```python
t.add('DIS')                   # Add a single item
s.update({'JJ', 'GE', 'ACME'}) # Adds multiple items to s
```

An item can be removed using `remove()` or `discard()`

```python
t.remove('IBM')    # Remove 'IBM' or raise KeyError if absent.
s.discard('SCOX')  # Remove 'SCOX' if it exists.
```

The difference between `remove()` and `discard()` is that `discard()` doesn’t raise an exception if the item isn’t present.



```python
portfolio = [
   ('ACME', 50, 92.34),
   ('IBM', 75, 102.25),
   ('PHP', 40, 74.50),
   ('IBM', 50, 124.75)
]

total_shares = { s[0]: 0 for s in portfolio }
for name, shares, _ in portfolio:
    total_shares[name] += shares

# total_shares = {'IBM': 125, 'ACME': 50, 'PHP': 40}
```

In this example, `{ s[0]: 0 for s in portfolio }` is an example of a dictionary comprehension. It creates a dictionary of key-value pairs from another collection of data. In this case, it’s making an initial dictionary mapping stock names to 0. The for loop that follows iterates over the dictionary and adds up all of the held shares for each stock symbol.

Many common data processing tasks such as this one have already been implemented by library modules. For example, the collections module has a Counter object that can be used for this task:

```python
from collections import Counter

total_shares = Counter()
for name, shares, _ in portfolio:
    total_shares[name] += shares

# total_shares = Counter({'IBM': 125, 'ACME': 50, 'PHP': 40})
```

