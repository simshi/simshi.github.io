----
title: Format Types Mismatch in printf/fprintf
tags: compile, c/c++
----

This is one blog of the [Compiler Warnings are Your Friends](/posts/2013-02-26-compiler-warnings-are-your-friends.html) series.

## Demonstrate Code

One of my colleagues met a crash issue of his code, so he consulted me, below code are simplified for the key point only.

```c
#include <stdio.h>

int main() {
    unsigned long long num = 4294967299ull; /*(0x100000003)*/
    unsigned int a = 5;
    const char* pa = "pa";

    printf("size:%d,llu:%llu\n", sizeof(num), num);
    printf("a:%d,num:%lu,pa:%p\n", a, num, pa);
    printf("a:%d,num:%lu,pa:%s\n", a, num, pa);

    return 0;
}
```

## The Compiler Warning

Let's compile the above code:

``` bash
$ gcc -o llu llu.c
llu.c:9:5: warning: format ‘%lu’ expects argument of type ‘long unsigned int’,
but argument 3 has type ‘long long unsigned int’ [-Wformat]
```

It means we provided a `long long unsigned int` argument, but use "%lu" to format it in printf. Nevertheless, the build is successful. (_In fact, there are more than one mistake within above code, and you'll get more warnings if compiled it with a modern compiler, e.g. use `%d` to print the result of sizeof, which is typed as `size_t`. But, let's focuse on the `llu` issue first_);

### Result of Running it

On a x86 (little-endian) PC, I got (_Probably you'll get different result on a 64bit machine_):

``` bash
$ ./llu
size:8,llu:4294967299
a:5,num:3,pa:0x1
/bin/bash: line 1:  3440
Segmentation fault      (core dumped) ./llu
```

As you can see, the value of num can be printed correctly by "%llu" in the first printf, but in second printf its value changed to 3 instead of 4294967299, and the followed `pa` got a value `0x1` (obviously it's a wrong pointer value). Even worse, the third printf caused a 'Segmentation fault'.

## Analysis

In the second `printf`, we use "%lu" in the format string, so `printf` grabbed 4 low bytes (`0x00000003`) from `num`, i.e. it was truncated, hence we got "num:3". Then "%p" grabbed the left 4 high bytes of num (`0x00000001`), printed it as a pointer value, so we got "pa:0x1".

In the third `printf`, the story is similar, but `printf` tried to dereference the invalid pointer address (i.e. "0x1") when it met "%s", so the program crashed.

The stack frame of printf:

+----------------------------------+
| 4 bytes "pa", a pointer          |
+----------------------------------+
| 8 bytes "num" 0x0000000100000003 |
+----------------------------------+
| 4 bytes "a" 0x00000005           |
+----------------------------------+

Interpreted incorrectly as:

+-------+--------+
| pa    |        |
+-------+--------+
| "%p"  | pa:0x1 |
+-------+--------+
| "%lu" | num:3  |
+-------+--------+
| "%d"  | a:5    |
+-------+--------+

## Solution and Suggestion

Firstly, as I mentioned in **Compiler Warnings are Your Friends**, you should fix all the compiler warnings, and turn on "-Werror" finally. Then this kind of bug would never leak to other people.

And the solution is very simple, just use correct length modifiers in format instrcutions: hh, h, l, ll, etc.

If you're working on platform which has `<inttypes.h>`, it's better to make it cross-platform by using macros it provides:

```c
#include <inttypes.h>

int64_t it;
uint64_t ut;

printf("%" PRId64 "n", it);
printf("%" PRIu64 "n", ut);
```

Please find more details about `inttypes.h` at [stackoverflow](http://stackoverflow.com/questions/6299323/good-introduction-to-inttypes-h).

## Reference
* [Compiler Warnings are Your Friends](/posts/2013-02-26-compiler-warnings-are-your-friends.html)

