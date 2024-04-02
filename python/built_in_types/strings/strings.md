---
layout: page
title: "Python Strings"
permalink: /python/general_syntax
---

## Python strings general information

[Python strings are immutable]({% link python/built_in_types/strings/string_immutability.md %})

## Formatting Python Strings

[f strings]({% link python/built_in_types/strings/f_strings.md %})

[String formatters]({% link python/built_in_types/strings/string_formatters.md %})

[Raw strings]({% link python/built_in_types/strings/raw_strings.md %})

## String methods

capitalize()	Converts the first character to upper case
casefold()	Converts string into lower case
center()	Returns a centered string
count()	Returns the number of times a specified value occurs in a string
encode()	Returns an encoded version of the string
endswith()	Returns true if the string ends with the specified value
expandtabs()	Sets the tab size of the string
find()	Searches the string for a specified value and returns the position of where it was found
format()	Formats specified values in a string
format_map()	Formats specified values in a string
index()	Searches the string for a specified value and returns the position of where it was found
isalnum()	Returns True if all characters in the string are alphanumeric
isalpha()	Returns True if all characters in the string are in the alphabet
isascii()	Returns True if all characters in the string are ascii characters
isdecimal()	Returns True if all characters in the string are decimals
isdigit()	Returns True if all characters in the string are digits
isidentifier()	Returns True if the string is an identifier
islower()	Returns True if all characters in the string are lower case
isnumeric()	Returns True if all characters in the string are numeric
isprintable()	Returns True if all characters in the string are printable
isspace()	Returns True if all characters in the string are whitespaces
istitle()	Returns True if the string follows the rules of a title
isupper()	Returns True if all characters in the string are upper case
join()	Converts the elements of an iterable into a string
ljust()	Returns a left justified version of the string
lower()	Converts a string into lower case
lstrip()	Returns a left trim version of the string
maketrans()	Returns a translation table to be used in translations
partition()	Returns a tuple where the string is parted into three parts
replace()	Returns a string where a specified value is replaced with a specified value
rfind()	Searches the string for a specified value and returns the last position of where it was found
rindex()	Searches the string for a specified value and returns the last position of where it was found
rjust()	Returns a right justified version of the string
rpartition()	Returns a tuple where the string is parted into three parts
rsplit()	Splits the string at the specified separator, and returns a list
rstrip()	Returns a right trim version of the string
split()	Splits the string at the specified separator, and returns a list
splitlines()	Splits the string at line breaks and returns a list
startswith()	Returns true if the string starts with the specified value
strip()	Returns a trimmed version of the string
swapcase()	Swaps cases, lower case becomes upper case and vice versa
title()	Converts the first character of each word to upper case
translate()	Returns a translated string
upper()	Converts a string into upper case
zfill()	Fills the string with a specified number of 0 values at the beginning


'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'removeprefix', 'removesuffix', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']




##### Using `vars()` For Variable Access

You can use the build in `vars()` method to access all in scope variables to utilize in your strings.  `vars()` will return a dictionary of all in scope variables

```python
>>> food = 'spam'
>>> qty = 10
>>> vars()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'food': 'spam', 'qty': 10}
>>> '%(qty)d more %(food)s' % vars()
'10 more spam'
```

## String Operations

### Repetition

```python
>>> 'Ni!' * 4
'Ni!Ni!Ni!Ni!'
```

### Character Operations

```python
>>> ord('s') # Get the ASCII binary value
115
>>> chr(115) # Get the character of binary value
's'
```

### Joining

```python
>>> my_list = ['a', 'b', 'c', 'd']
>>> my_string = ','.join(my_list) # yields 'a,b,c,d'
>>> my_string
'a,b,c,d'
```

## String Slicing

The general form `s[i:j]` means "give me everything in s from offset, but not including offset j"

```python
>>> s = 'yeti'
>>> s[1:3]
'et'
>>> s[1:]
'eti'
>>> s[0:3]
'yet'
>>> s[:-1]
'yet'
>>> s[:]
'yeti'
>>> s = 'abcdefghijklmnopqrstuvwxyz'
>>> s[1:10:2] # Grab every other (2nd) item from offset 1-9
'bdfhj'
>>> 'hello'[::-1] # Use negative stride to reverse a string
'olleh'
>>> s[5:1:-1] # If using negative stride the first two arguments are essentially reversed
'fedc'
```

## Encoding

Most non-UTF codecs handle only a small subset of Unicode characters. When converting test to bytes, if a character is not defined in the target encoding, `UnicodeEncodeError` will be raised, unless special handling is provided by passing an `errors` argument to the encoding method.

```python
>>> city = 'São Paulo'
>>> city.encode('utf_8')
b'S\xc3\xa3o Paulo'
>>> city.encode('iso8859_1')
b'S\xe3o Paulo'
>>> city.encode('cp437', errors='ignore')
b'So Paulo'
>>> city.encode('cp437', errors='replace')
b'S?o Paulo'
>>> city.encode('cp437', errors='xmlcharrefreplace')
b'S&#227;o Paulo'
```

`error='ignore'` Skips characters that cannot be encoded.  Usually bad as it leads to lost data.

`error='replace'` Substitutes character that cannot be encoded with `?`; data is also lost but user is given a hint that something changed.

`error='xmlcharrefreplace` Replaces un-encodable characters with an XML entity.  If you can't use UTF, and you can't afford to loose data this is the only option.
