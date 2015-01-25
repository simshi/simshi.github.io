--------
title: Do I need to include .h file in .c with same name?
tags: compile, c/c++
--------

[1]: /posts/2013-02-26-compiler-warnings-are-your-friends.html

This is also a blog of the [Compiler Warnings are Your Friends][1] series.

## The Problem

`client.c` is using a function `int foo(int a)`, which is implemented in `module.c`, and declared in `module.h`. It's obvious that `client.c` must include `module.h`, otherwise the compiler would complain about undeclared `foo()`, but what is not so obvious to many engineers: shall I include module.h in module.c?

![](/images/module_question.gif)

"It's not needed if module.c doesn't use structures/types defined in module.h.", someone would say.

## Case Study

We don't like gamble, let's try the above example, say we have 3 files, `module.c` doesn't include `module.h` here:

``` c
$ cat -n module.c
 1  //#include "module.h"
 2
 3  int foo(int a, int b)
 4  {
 5      return a/b;
 6  }

$ cat -n module.h
 1  #ifndef _MODULE_H_
 2  #define _MODULE_H_
 3
 4  int foo(int a);
 5
 6  #endif

$ cat -n client.c
 1  #include <stdio.h>
 2
 3  #include "module.h"
 4
 5  int main(void)
 6  {
 7      int i=3;
 8      printf("foo(%d): %d\n", i, foo(i));
 9
10      return 0;
11  }
```

Now let's build them and execute the output:

``` bash
$ gcc -g -Wall -Wextra -c -o client.o client.c
$ gcc -g -Wall -Wextra -c -o module.o module.c
$ gcc -o output client.o module.o
$ ./output
$ foo(3): 0
```

Do you see something very bad are happening? The definition and declaration of foo are different but GCC build them without any complain! i.e. This code has just fooled the compiler and the linker, but the worst part happens at runtime, while it didn't crash to warn us, instead, it just sneaked into the stack, grabbed a random value as its second argument, `int b`, then left us with a surprising output! (_Well, it might crash on some environment, if foo got a zero as `int b`, the behavior is undefined_).

Now, let's include `module.h` in `module.c`, try what we'd find this time:

``` c
$ cat -n module.c
 1  #include "module.h"
 2
 3  int foo(int a, int b)
 4  {
 5      return a/b;
 6  }
```

``` bash
$ cc  -g -Wall -Wextra  -c -o module.o module.c
module.c:4: error: conflicting types for ‘foo’
module.h:4: error: previous declaration of ‘foo’ was here
```

This one line including created an explicit relationship between `module.c` and `module.h`, so GCC figured out that mistake and failed the compiling, that's what we'd eagerly expected, rather than a bug found by feature test, system test, or even by our beloved customers, right?

## Answer

So, now, your questions should have melted away, with the below figure shows what you shall do, and you've already understood the reason behind it:

![](/images/module.gif)

BTW, grabing some declarations by something like `extern int foo(int a);` should be considerated harmful too, it just like a dialect of this kind of mistakes we're trying to avoid, because the extern stuff might be out of date, it would also breaks the relationship among functions' declarations and definitions.

## Solution

But that only leads us to another trouble: how can I find all of these potential bugs, i.e **'definitions without declarations'** in my code? It's quite a lot of tedious hours to find them, right?

Praying for a salvation? Here it is:

> Compiler warnings are your friends.

In this case, GCC has an option `-Wmissing-prototypes` (or `-Wmissing-declarations`), which can find exactly such kind of carelessness, let's comment the include again and compile `module.c` with this option:

``` c
$ cat -n module.c
 1  //#include "module.h"
 2
 3  int foo(int a, int b)
 4  {
 5      return a/b;
 6  }
```

``` bash
$ gcc -c -o module.o -Wall -Wextra -Wmissing-declarations module.c
module.c:4: warning: no previous declaration for ‘foo’
```

See? GCC did all the dirty works for you!

## Reference:

* [Compiler Warnings are Your Friends][1]

