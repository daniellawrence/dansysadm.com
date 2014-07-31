Title: A simple example using a python cache decorator
Date: 2012-02-01 10:20
Category: blog
Tags: python, cache, decorator
Summary: Using the classic Fibonacci equation we can show the differences and benefits of using a python decorator as a caching engine for functions.

Using the classic Fibonacci equation we can show the differences and benefits of using a python decorator as a caching engine for functions.

Taking the below code to work out the 10th Fibonacci number, however I had added a sleep of one second every-time the Fibonacci function is called.

```python
#!/usr/bin/env python
from time import sleep

def fibonacci(n):
   "Return the nth fibonacci number."
   sleep(1)
   if n in (0, 1):
      return n
   return fibonacci(n-1) + fibonacci(n-2)
print "working out fibonacci(10)..."
print fibonacci(10)
```

This increases the non-sleep execution time from __0m0.019s__ to __2m57.215s__.

This will allow you see the changes in execution time from cached vs non-cached answers. 
As when the answer is cached it will skip over the sleep statement.

In this example the sleep statement is a real world place holder for timely execution time.

Memorize code
-------------

By adding the below function memorize, it will simply create a dictionary of past results based on the arguments used to call the function.

```python
def memorize(function):
  memo = {}
  def wrapper(*args):
    if args in memo:
      return memo[args]
    else:
      rv = function(*args)
      memo[args] = rv
      return rv
  return wrapper
```

Using Memorize 
--------------

All you need to do to cache the answers of the function is to add @memorize decorator above the fibonacci function.

```python
#!/usr/bin/env python
from time import sleep

def memorize(function):
  memo = {}
  def wrapper(*args):
    if args in memo:
      print "returning %s from cache" % memo[args]
      return memo[args]
    else:
      rv = function(*args)
      memo[args] = rv
      return rv
  return wrapper

@memorize
def fibonacci(n):
   "Return the nth fibonacci number."
   sleep(1)
   if n in (0, 1):
      return n
   return fibonacci(n-1) + fibonacci(n-2)

print "working out fibonacci(10)..."
print fibonacci(10)

```

Results
-------

Now when you execute the code to find the 10th Fibonacci number. The function will return the cached results to speed up the answer.

Moving the execution time from 2m57.215s to 0m11.032s.

To the extreme
--------------

In the code to fibonacci(10) another 5 times, then you will really see the power of caching, as without caching this took over 10 minutes to run on my machine.

 
```python
print "working out fibonacci(10)..."
print fibonacci(10)
print "working out fibonacci(10)..."
print fibonacci(10)
print "working out fibonacci(10)..."
print fibonacci(10)
print "working out fibonacci(10)..."
print fibonacci(10)
print "working out fibonacci(10)..."
print fibonacci(10)
print "working out fibonacci(10)..."
print fibonacci(10)
```

However with cache..

The above only slowed down the code by .004 of a second (for a total user time of 0m11.036s).
