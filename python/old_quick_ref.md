

# Python Hints

## Reading input from std in.

 in = input().strip().split()

## Arrays

### Inserting

    L.append('NI') // Add to tail
    L.insert(2,'eggs') // Insert by position 
    L.extend[3,4,5] // Append multiple itemms
    L + [4,5,6] // Combines two arrays does not modify L unles you +=

    ### Remove
        L.pop() // returns the tail value and removes it
        L.pop(0) // 'pops' from specific position
        L.remove('eggs') //remove by value
  del L[0] //delete one item.
 
### Count occurences

  L.count('spam')
 
### Sort array

M.sort()
M.sort(key=str.lower, reverse=True) //exampel array ['abc', 'ABD', 'aBe']
 
### Reverse Array

M.reverse()
 
### Combine two arrays

L + [4,5,6]
 
### List comprehension
 
[ expression for target in iterable]
[ [x**2,x**3] for x in range(4)] // output [[0,0],[1,1],[4,8],[9,27]]
 
#### Get col 2 of a matrix
 
   
## Dictionaries

### Avoiding a missing key
 
if 'key' in D:
pass
  
  value = D.get('X') # weill return None by default or you can set default value
  value = D.get('X',0)
  
 ### Dictionary operations
 
  D = {} // Create empty dictionary
  D['key']='value'
  
  D.pop('muffin') //remove single item
  
  list(D.value()) // gives you array of values
  list(D.keys()) // give you array of keys
  
  D.update(D2) // combine 2 dictionaries
  
 
## Strings

 ** Strings are immutable.  Every stirng op creates new string **
 
 ### grab every second chaar from offset 1 to 9

 s[1:10:2]
 
 ### Reverse string (neg stride)

 s[::-1]
 
 ### If using neg stride first args are reversed

 s[5:1:-1]
 
## Numeric

 x // y  - Floor division
 x ** y  - poer notation
 
 sum(1,2,3,4)
 abs(-42)
 min([1,2,3,4])
 max([1,2,3,4])

## Print()
 
 ### custom seperator

 print(x,y,z, sep=", ")

 ### supress line break

 print(x,y,z, end='')

## Files

 ### Open file and don't worry about closing
 
  with open('abc.txt', 'w') as file:
   file.write('hello\n')
  
 ### Open file and don't worry about auto close and read fromit.
 
  with open('abc.txt', 'r') as file:
   for line in file:
    print(line)
   
 ### regular file open and close

  f = open('data.txt', 'w')
  f.write('Hello\n')
  f.close()
  
  
## Functions

 ### Calling main function
 
  if __name__ == '__main__':
   ...
 
 ### Named args will make a dictionary of your args.  Works with named args only.
  
  def f(**args):  # **args will make a dictionary fo your args.  Works with names args only.
   print(args)
   
    >>> f(a=1, b=2)
    {'a': 1, 'b': 2}
 
 ### positional args will collect argumetns into a tuple and assign that tupple to args.
  
  def f(*args):
   print(args)
  
  >>> f(1,2,3,4)
  (1, 2, 3, 4)
   
 ### Combining both types
  
  #### Note that order is part of the syntax keyword has to be last.

  def f(a *pargs, **kwargs):
   print(a, pargs, kwargs)
  >>> f(1,2,3, x=1, y=2)
   1 (2, 3) {'x': 1, 'y': 2}

## Classes

 ### Basic Class Exmample
 
  class Example:
   def __init__(self):
    self.data = 0
   def increment(self,val=1):
    self.data += val
   def incrementAndPrint(self,val=1):
    self.increment(val)
    print(self.data)
        
  if __name__ == '__main__':
   c = Example()
   c.incrementAndPrint(1)
 
 ### A node for linked list
  
  class ListNode:
   def __init__(self, val=0, next=None):
    self.val = val
    self.next = next

## Map
 
 Map applies a function to each element of a list and copies the function result to a new list
 >>> list(map(abs, [-1,-2,0,1,2]))
 [1,2,0,1,2]
 
## Filter
 
 Filter will select an iterable based on a test and return the result to an iterable.
 
 >>> list(filter(lambda x: x > 0), range(-5,5)))
 [1, 2, 3, 4]
 
## try/catch example

 try:
  if something:
   raise IndexError
  fetcher(x,3):
 except IndexError:
  print('got exception')
  
# Algs and Sorts

## Quick Sort

 def partition(a, lo, hi):
  i = lo + 1
  j = hi

  while True:
   while i <= j and a[i] <= a[lo]:
    i += 1
    
   while i <= j and a[j] >= a[lo]:
    j -= 1
    
   if i <= j:
    a[i], a[j] = a[j], a[i]
   else:
    break

  a[lo], a[j] = a[j], a[lo]

  return j
  
 def quickSort(a, lo, hi):
  if lo < hi:
   p = partition(a,lo,hi)
   quickSort(a,lo,p-1)
   quickSort(a,p+1,hi)

 if __name__ == "__main__":
  quickSort(a, 0, len(a)-1)
 

# Log Count Example

 import re
 from collections import defaultdict

 with open('syslog.txt') as input:
  minute_counts = defaultdict(lambda: 0)
  for line in input:
   minute = re.search(r'^[\w]+\s+\d\s\d\d:\d\d',line).group()
   if minute:
    minute_counts[minute] += 1

 for minute in sorted(minute_counts, key=minute_counts.get):
  print(f'{minute},{minute_counts[minute]}')

# API Example

 import requests
 import json

 def print_employee_tree(id, indent):
  r = requests.get(f'http://localhost:5000/employees/{str(id)}')
  if r.status_code == 200:
   resp = r.json()
   print(indent * " " + resp['name'])
   if resp.get('reports'):
    reports = resp['reports']
    for report in reports:
     print_employee_tree(report, indent+1)
  else:
   print("ERROR: Could not contact API")

 if __name__ == "__main__":
  print_employee_tree(3,0)



# Python Regex

 ## Special Charachters

  .   Any charachter except newline
  *  Zero or more repetitions
  .*  Pattern that matches 0 or more of any charachter (duh)
  +  Matches one or more repetitions of preceding regex
  ?  Matches zero or one repetitions of the preceding regex
  {m}  Matches exactly m repetitions of preceding regex
  {m,n} Matches any number of repetitions of the preceding reges from m to n inclusive
  {,n}  Any number of repetitions of <regex> less than or equal to n
  {m,}  Any number of repetitions greater than or equal to M
  {,}   Any number of repetitions or <regex> same as <regex>*
  {m,n}? The non greedy (lazy) version of {m,n}
  |  Specifies a set of alternatives to match


 ## Greedy vs NonGreedy

  *? +? ?? Are all the non greedy (or lazy) version of the *,+ and ? quantifiers
  
   re.search('<.*>', '%<foo> <bar> <baz>%') yields <foo> <bar> <baz> as match (first '<' and last '>')

   re.search('<.*?>', '%<foo> <bar> <baz>%') yields <foo> (non greedy uses the fist '>' it finds) 

 ## Grouping

  (<regex>) Defines a sub expression or Grouping

  re.search('(bar)+','foo bar baz')  yields 'bar'
  re.search('(bar)+','foo barbarbar baz')  yields 'barbarbar'
  res.search('bar+', 'foo barbar baz) yields None here barr would be a match since + works on the 'r'

  You can nest group as in '(ba[rz]){2,4}(qux)?' or '(foo(bar)?)+(\d\d\d)?'

  ### Capturing groups

   .groups() re call will return a tuble of all groups matched. for example...

    m = re.search('(\w+),(\w+),(\w+)', 'foo,quux,baz') yiels ...
     <_sre.SRE_Match object; span=(0, 12), match='foo:quux:baz'>
    m.groups() -> ('foo', 'quux', 'baz)

     and you can access foo, quux and baz from tupble returned by groups.
     notice that commas in string are not in tupble as they were not in group.

  ### Backreferences

   \<n> Matches the contents of previously captured group

   value of n can be between 1 and 99

   regex = r'(\w+),\1'
   re.search(regex, 'foo,foo') -> matches becuase you match foo and then foo again after literal commad
   re.search(regex, 'qux,qux') -> matches similar to above
   re.search(reges, 'foo,qux') -> does not match.

  ### Named Capture groups

   (?P<name><regex>) Creates a named capture group

   
   >>> m = re.search(r'(\w+),(\w+),(\w+)', 'foo,quux,baz')
   >>> m.groups()
   ('foo', 'quux', 'baz')
   >>> m.group()
   'foo,quux,baz'
   >>> m.group(1,2,3)
   ('foo', 'quux', 'baz')
   >>> m = re.search('(?P<w1>\w+),(?P<w2>\w+),(?P<w3>\w+)', 'foo,quux,baz')
   >>> m.groups()
   ('foo', 'quux', 'baz')
   >>> m.group('w1')
   'foo'
   >>> m.group('w2')
   'quux'
   >>> m.group('w1','w2','w3')
   ('foo', 'quux', 'baz')

   #### Using named group in regex

    >>> m = re.search(r'(?P<word>\w+),(?P=word)', 'foo,foo')
    >>> m
    <_sre.SRE_Match object; span=(0, 7), match='foo,foo'>
    >>> m.group('word')
    'foo'

    Note: the angle brackets are required when you are naming a group, but not when you are referncing it.

  ### Non captuirng group

   ?:<regex> Creates a non capturing group  A non capturing group mtches the secified <regex> but does not include it in match for later retrieval.

   >>> m = re.search('(\w+),(?:\w+),(\w+)', 'foo,quux,baz')
   >>> m.groups()
   ('foo', 'baz')

   >>> m.group(1)
   'foo'
   >>> m.group(2)
   'baz'

  ### Conditional Matches

   (?(<n>)<yes-regex>|<no-regex>)
   (?(<name>)<yes-regex>|<no-regex>) Specified a conditional match

   A conditional match matches agains one of two specified regexes depending on whether the given group exists.

   >>> regex = r'^(###)?foo(?(1)bar|baz)'
   >>> re.search(regex,'###foobar')
   <re.Match object; span=(0, 9), match='###foobar'>
   >>> print(re.search(regex,'###foobaz'))
   None
   >>> print(re.search(regex, 'foobar'))
   None
   >>> re.search(regex, 'foobaz')
   <_sre.SRE_Match object; span=(0, 6), match='foobaz'>

  ### Lookahead nad Lookbehind assertions

   (?=<lookahead_regex)
   (?!<lookahead_regex) creates a negative lookahead assertion (what follows must not match)
   (?<=<lookbehind_regex)

   What is special about lookahead and lookbehind is that the portion of the sting that matches is not consumed and it isn't part of the returned match object.

    >>> re.search('foo(?=[a-z])', 'foobar')
    <_sre.SRE_Match object; span=(0, 3), match='foo'>
    
    >>> re.search('foo(?=[a-z])', 'foobar')
    <_sre.SRE_Match object; span=(0, 3), match='foo'>
    >>> print(re.search('foo(?![a-z])', 'foobar'))
    None

    >>> print(re.search('foo(?=[a-z])', 'foo123'))
    None
    >>> re.search('foo(?![a-z])', 'foo123')
    <_sre.SRE_Match object; span=(0, 3), match='foo'>

    >>> re.search('(?<=foo)bar','foobar')
    <re.Match object; span=(3, 6), match='bar'>

  ### comments

   (?#...)  Specified a comments

   >>> re.search('bar(?#This is a comment) *baz', 'foo bar baz qux')
   <_sre.SRE_Match object; span=(4, 11), match='bar baz'>




   