+++
title = "Optimizing Functions with Python Caching Decorators"
date = "2011-06-11"
tags = ["c", "cache", "decorators", "dynamic programming", "python"]
+++

On these last months I've been solving some problems (such as some
[HMMs](http://en.wikipedia.org/wiki/Hidden_Markov_model) algorithms)
which the best solutions involves some kind of [dynamic
programming](http://en.wikipedia.org/wiki/Dynamic_programming). Some of
them are quite simple to implement, but their recursive formulation are
far more intuitive. The problem is that even in functional languages,
the recursive functions aren't well handled unless you some mechanism
like [tail call](http://en.wikipedia.org/wiki/Tail_call), which aren't
intuitive as we would like to. The simplest example that comes in my
mind is the fibonacci function which is usually defined as:

` fib(0) = 1 fib(1) = 1 fib(n) = fib(n-1) + fib(n-2)`

As we know, almost all the languages compilers and interpreters use the
[call stack](http://en.wikipedia.org/wiki/Call_stack) to call the
recursives cases on functions being executed. We can analyze the
following C fibonacci version:\

```c
int fib(n) {
	 if (n == 0 || n == 1)
        return 1;
    else
        return fib(n-1) + fib(n-2);
}
```

It is really simple to understand when contrasted with the definition.
But, if we make a trace of the the program (even with a small input
value) we'll have something like the following evaluation order:

```c
fib(6) = fib(5) + fib(4)
fib(5) = fib(4) + fib(3)
fib(4) = fib(3) + fib(2)
fib(3) = fib(2) + fib(1)
fib(2) = fib(1) + fib(0)
fib(3) = fib(2) + fib(1)
fib(2) = fib(1) + fib(0)
fib(4) = fib(3) + fib(2)
fib(3) = fib(2) + fib(1)
fib(2) = fib(1) + fib(0)
fib(3) = fib(2) + fib(1)
fib(2) = fib(1) + fib(0)
```

![Call stack for fib(6)

As we can see, there is a repetition of the calculation of fib 4 to 1,
many times and is something we can avoid. In fact, the complexity of
this solution has a exponencial computational complexity because for
each n from input we branch it in 2 until it reachs 0 or 1 approximately
n times, leading us into a O($2^n$) complexity. A simple way to avoid it,
is converting into a interactive form:

```c
int fib(int n) {
    int current = 1;
    int previous = 1;
    int i;
    for (i = 1; i < n; i++) {
        int temp = current; // XXX: nasty
        current += previous;
        previous = temp;
    }
    return current;
}
```

The same result is achieved by using
[tail call](https://secure.wikimedia.org/wikipedia/en/wiki/Tail_call)
for functional languages.

As you can see, it obfuscates the intuitive definition given in as the
recursive formulation. But we still have a problem whenever we calculate
fib(n), we have to recalculate it's previous results even if they was
previously calculated. If this function is used many times in our
program it will take a lot of processing re-computing many of the
values. We can avoid this by using the dynamic programming, which keeps
the record of previously calculated results. The drawback of this
technique is the memory usage, which for large entries can become a
bottleneck. However, processing usually is a more rare computer
resource. A C implementation (not the most elegant) for it is:

```c
// XXX: kids, don't do this at home
int fib_results[10000];
int last_fib;
int fib(int n) {
    if (n <= last_fib)
        return fib_results[n];
   int current = fib_results[last_fib-1];
   int previous = fib_results[last_fib-2];
   for (; last_fib < n; last_fib++) {
       int temp = current;
       current += previous;
       fib_results[last_fib] = current;
       previous = temp;
   }
   return current;
}

int main()
{
    fib_results[0] = 1;
    fib_results[1] = 1;
    last_fib = 1;

    // ... other stuff ...

    return 0;
}
```


As we can see, dynamic programming isn't too hard to implement. On the
other hand, reading a code it's a though task to do unless you are
already familiar with the algorithm.

If we extract what is dynamic programming fundamental concept, which is
"store pre-computed results", we find a regularity in every recursive
function which we can be transformed into a dynamic programming one. One
of the reasons I love python because it's easy to use meta-programming
concepts, and that's what I will use to transform recursive functions
into it's dynamic form in a ridiculous easy way using function
decorators.

Function decorators (or annotations in Java) are a form of
meta-programming for functions. It extends functions with some
functionalities, such as debugging, tracing, adding meta-data to the
function, synchronization or
[memoization](https://secure.wikimedia.org/wikipedia/en/wiki/Memoization)
(not memorization) of values, which is a great way of optimizing
recursive functions by caching their results (if you have enough memory
available). One possible implementation of memoitized decorator in
python is the following:

```python
def cached(function):
    cache = {}
    def wrapper(*args):
        if args in cache:
            return cache[args]
        else:
            result = function(*args)
            cache[args] = result
            return result 
    return wrapper
```

Note that I'm not using kwargs because they're not hashable, such as the
tuple args, and will add a few complexity in the example. See also that
we a have a function that returns another function, which uses a given
one to calculate results and store them in a cache. To cache our fib
function we may use the following code:

```python
@cached
def fib(n):
     if n == 0 or n == 1:
         return 1
     else:
         return fib(n-1) + fib(n-2)

# or in a not so clear version:
def normal_fib(n):
    if n == 0 or n == 1:
        return 1
    else:
        return fib(n-1) + fib(n-2)

fib = cached(normal_fib)
```

This technique is really useful to improve your code performance in a
really easy. On the other hand, it isn't the best solution for almost
all the cases. Many times code a dynamic programming method (if
performance is crucial) will be necessary. Is also important to notice
that I didn't used any cache memory management policy, which is
important to economize memory. Most appropriate cache data structures
(such as numpy arrays for integer arguments) also are welcome. The
python 3.2 version added the lru_cache decorator into the functools
module to make this cache mechanism. If you are already using this
version, it's smarter to use it instead of implementing your one. Here
is how it must be used:

```python
# Python > 3.2 import functools
@functools.lru_cached(max_size=500) # uses a fixed size cache to avoid memory usage explosion
def fib(n)
     ...
```

This technique is very useful not only for economize the CPU resources
but also network (such as caching SQL query results), other IO
operations (such as disk reading) and even user interaction input in a
straightforward way.
