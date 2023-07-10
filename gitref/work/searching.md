---
layout: page
title: "Searching In Your Work"
permalink: /gitref/work_locally/searching
---

Git ships with a built in grep that allows you to search though any [committed tree]({% link gitref/definitions.md %}#committed-tree), or the index for a string of regular expressions.  The benefit of git grep over regular grep is that its faster and you can search through any [tree]({% link gitref/definitions.md %}#tree) in Git not just the working directory.

By default git grep will look through files in your local directory.  

**You can use the -n or --line-number option to print out the line numbers where Git has found matches.**

`git grep -n my_magic_function` or `git grep --line-number my_magic_function`

There are many options to git grep.  See `man git grep` for what's available.

**You can use -c or --count to get the file names and found instance counts.**

`git grep -c TODO` or `git grep --count TODO`

**Look for lines defining either of two options**

The example below will look for a line that has #define and either MAX_PATH or PATH_MAX.

`git grep -e '#define' --and \( -e MAX_PATH -e PATH_MAX \)`

**See the function name of the git grep match**

You can have git show you the preceding line of function matches with the -p or --show-function argument.  In other words Git Grep will look for function definition where your search matches and prints that so you know which function its in.   In example below I am searching for 'return' as its a good example of seeing the match and the function name.

`git grep -p return *.c` or `git grep --show-function return *.c`

You can search for complex combinations of strings with the --and flag, which ensures that multiple matches must occur in the same line of text. 

**Exclude a directory from your search**

The example below looks for solution, excluding files in Documentation.

`git grep solution -- :^Documentation`

**search for lines with values in a file that has all the listed values**

The example below looks for a line that has NODE or Unexpected in files that have lines that match both.
`git grep --all-match -e NODE -e Unexpected`

**search older version of code using a tag**

The example below will search for lines that define a constant whose name contains either of the substrings LINK or BUF_MAX specifically in an older version of the codebase by specifying the tag v1.8.0

`git grep --break --heading -n -e '#define' --and \( -e LINK -e BUF_MAX \) v1.8.0`

* --break argument will print empty lines between matches from different files

* --heading argument will show the file name above the matches

* -n or --line-number will add line numbers to output

* -e argument specified that the next parameter in command is the pattern.