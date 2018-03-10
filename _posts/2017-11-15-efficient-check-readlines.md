---
title: "Efficient way of reading file content - for large files"
tags: [python]
---

There are multiple ways for dealing with files in Python. I'm going to explain why one
solution is more efficient than the other.

You want to check if a group of patterns is contained in a line of a file.
For the example, let's use something simple. The /etc/hosts file on a linux
always contains a loopback interface line such as: `127.0.0.1	localhost`

Let's use some code to check if our file contains the patterns `['127.0.0.1', 'localhost']`:

```python
patterns = ['127.0.0.1', 'localhost']
with open('content.txt', 'r') as f:
	cont = f.readlines()
	for line in cont:
		found = False
		for pat in patterns:
			if pat in line:
				found = True
			else:
				found = False
	if found:
		print 'found'
	else:
		print 'no match.'
```
		
This code is long, and performs a simple task. This code is intended for readability;
The big drawback here besides it's length, is the second line - `f.readlines()` actually
loads the whole file content into memory. This isn't an issue with small files, but when
suddenly your code reads 500MB input file.. this can take loooong time (or even throw an Exception
if you don't have enough free memory).


# We just want to check if the patterns appear on the file..
### Do we really need to load the whole content to memory?

Python is very powerful language. And you gonna see that now.
Instead of loading the file onto memory, Python provides a way to *iterate over
it on a line-by-line basis*. That means, on every iteration *there's only one line loaded
onto memory*. This still produce the same results, and is more efficient with large files.

```python
with open('content.txt', 'r') as f:
	for line in f:
		...
```

Just remove the `readlines()` method and use straight the `for` loop over the *file object*.
This code is more efficient. It now iterates on the file, one line at a time. But wait, we can improve it even more.

# The use of any

What is the purpose of the `if .. else` clause code block?
It checks for *containment*: whether the pattern appears in one of the lines.

>**any** *Return True if any element of the iterable is true.*

We can rephrase our problem to:
Does *any* of the lines in our file, contains *all* the patterns we look for in it?

```python
with open('content.txt', 'r') as f:
	return any(line for line in f if all(pat in line for pat in patterns)
```	

That's it! Let's break this down..

- any: iterates through the lines of the file (loads one at a time) and checks:
- if all: returns True only if all checks are True. We iterate over the patterns, and 
return the result of a containment check: `pat in line`
If all patterns exists, it will return True. Otherwise, it returns False. (exactly what we need)
- The code now returns the value of `any(..)`, which stops on the first match.
If we have found something that is True in any, the iterations stops, because we already know 
that one of the items is True, and that's what we check in this code.
