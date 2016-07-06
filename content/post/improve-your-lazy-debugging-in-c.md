+++
title = "Improve your lazy debugging in C++"
date = "2013-12-27"
tags = ["__LINE__", "__PRETTY_FUNCTION__", "c", "c++", "cout", "debug", "gcc", "macro", "preprocessor", "printf"]
+++
Are you too lazy to learn your debugger? When debugging, I usually want
to know if some code section is reached.

I can do this:

```c
void foobar() {
    printf("foobar()\n");
    // ... do foobar
}
```

But I should use [`cout` instead `printf` in C++](http://stackoverflow.com/questions/2872543/printf-vs-cout-in-c).
[`printf` is faster](http://programming-designs.com/2009/02/c-speed-test-part-2-printf-vs-cout/),
but in this case it should be irrelevant. So I have:

```c++
void foobar() {
    std::cout << "foobar()" << std::endl;
    // ... do foobar
}
```

I could also use [GCC's function name macro `__PRETTY_FUNCTION__`](http://gcc.gnu.org/onlinedocs/gcc/Function-Names.html)
and [`__LINE__` macro](http://gcc.gnu.org/onlinedocs/cpp/Standard-Predefined-Macros.html)
to identify the reached code section easier.

```c++
void foobar() {
    std::cout <<  __PRETTY_FUNCTION__ << " at line " <<  __LINE__ << std::endl;
    // ... do foobar
}
```

Add that on a macro to reuse it:

```c++
#if DEBUG
#include 
#define REACH std::cout <<  __PRETTY_FUNCTION__ << " at line " <<  __LINE__ << std::endl;
#endif

void foobar() {
    REACH
    // ... do foobar
}
```

And happy debugging!

Not.
