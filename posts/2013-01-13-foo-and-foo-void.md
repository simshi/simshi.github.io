----
title: What's the difference between `foo(void)` and `foo(void)`?
tags: c/c++, compile
----

![](http://imgs.xkcd.com/comics/debugger.png)

## Problem
Let's begin with a little bit of C code:
``` c
//void.h
int foo();

//void.c
int foo(int x)
{
    return x+1;
}

//main.c
#include <stdio.h>
#include "void.h"

int main()
{
    printf("foo:%d\n", foo());
    return 0;
}
```

Use below command to compile and link them together:

``` bash
gcc *.c -o void -Wall -Wextra
```

Which result do you expect?

1. gcc warns about unmatched prototypes of 'foo';
2. compilation is OK, but run the './void' results into a 'segmentation fault';
3. compilation is OK, but run the './void' prints a random value.

> .
> .
> .
> .
> .
> .
> .
> .

The result is the 3rd one in my environment, gcc gives no warnings on this piece of code, the result is random, but it works without any segmentation fault.

## Analysis

The mumbo-jumbo behind it is how gcc inteprete the decleraration in `void.h`, as ANSI C mentioned:

> The empty list in a function declarator that is not part of a function definition specifies that no information about the number or types of the parameters is supplied.

As it implies, the declaration `int foo();` in `foo.h` **doesn't mean `foo` has no parameter, it means not specified**. Hence, gcc accepts it because it doesn't conflict with `int foo(int x)` (though the result is unexpected).

The story can't endup here, next question should be how to express a funtion doesn't have any parameter?

Let's refer to ANSI C paper again:

> The special case of void as the only item in the list specifies that the function has no parameters.

No surperise what the above code actually saying is `int foo(void);`.

## Solution

But you can imagine if this scenario happened it might cost an engineer lots of time to debug. How to avoid such boring things? The GCC compile option `-Wstrict-prototypes` is a simple and elegent method you're looking for:

``` bash
gcc *.c -o void -Wall -Wextra -Wstrict-prototypes
```

now gcc generates warnings like:

> void.h:1: warning: function declaration isn't a prototype

It helps you to find all those potential issues.

## Note

* C++ works on strict prototypes by default, so `int foo();` has the same meaning with `int foo(void);` in C++;
* Some compilers don't generate any warning in my test, please find documents here if you want to know more.
* Check out compiling tactics at [Compiler Warnings are Your Friends](/posts/2013-02-26-compiler-warnings-are-your-friends.html).

_Many thanks to [xkcd.com](http://xkcd.com) for the comic at begin, along with many other instresting comics, and its kind [license](https://xkcd.com/license.html)._

