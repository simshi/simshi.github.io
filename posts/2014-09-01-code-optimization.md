---------------------
title: On Code Optimization
tags: compiler, c/c++
---------------------

## Software Exorcism

Recent months, Eric, Harry and I has translated a book, [Software Exorcism](http://www.amazon.com/Software-Exorcism-Handbook-Debugging-Optimizing/dp/1590592344), into [Chinese version](http://book.douban.com/subject/25900304/), and here is the [translators' preface written by Eric](http://zhangliaoyuan.me/blog/2014/02/24/translating-preface-of-software-exorcism/). It's a book on debugging and optimizing legacy code, but in this blog, I'll only talk about a little thought on optimization.

## Branch Elimination

Check the below C code, it has a if-else statement which would created a branching structure in machine code eventually:

``` c
#include <string.h>

typedef unsigned char BOOLEAN;
#define TRUE 1
#define FALSE 0


BOOLEAN editName(char *name)
{
    if (strlen(name)>0) {
        return(TRUE);
    } else {
        return(FALSE);
    }
}
```

Let's compile it by gcc on a x86 platform:

``` asm
editName(char*):
	pushq	%rbp
	movq	%rsp, %rbp
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rax
	movzbl	(%rax), %eax
	testb	%al, %al
	je	.L2
	movl	$1, %eax
	jmp	.L3
.L2:
	movl	$0, %eax
.L3:
	popq	%rbp
	ret
```

As you can see, there is a branch in above asm code, `je .L2`, seems it's a optimization point if we really despaired in performance target. So the book author suggested to optimize it as below:

``` c
BOOLEAN editNameWithoutBranch(char *name)
{
    return(strlen(name));
}
```

Let's check the result on the same platform:

``` asm
editNameWithoutBranch(char*):
	pushq	%rbp
	movq	%rsp, %rbp
	subq	$16, %rsp
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rax
	movq	%rax, %rdi
	call	strlen
	leave
	ret
```

What's changed here? We can see there is no branching instruction in the asm code generated from `editNameWithoutBranch`, but (here comes the truth) there is a function call on `strlen`, which is eliminated in the if-else version. For if-else version, gcc is smart enough to generated some code as `if (name[0] == 0)` (`testb %al %al`), instead of calling the `strlen`.

So, which version would be better in performance? It depends, different platforms (mostly impacted by CPU architecture) give different results, but at least we know, optimizing at such a low lever may not a wise thing to do, **compiler can do smart things for us, and usually it's better than most programmers at most time**.

Now, to get the most out of compiler intelligence, let's try `-O2` for above code, I got:

``` asm
editName(char*):
	cmpb	$0, (%rdi); compare name[0] with '\0'
	setne	%al
	ret

editNameWithoutBranch(char*):
	subq	$8, %rsp
	call	strlen
	addq	$8, %rsp
	ret
```

This time, you can figure out the winner easily, `return name[0] != '\0';` is definitely faster than `return strlen(name);`.

## But Why?

The reason why compiler can optimize away the calling to `strlen` in the `editName` lies in the definition of `TRUE` and `FALSE`, as you can see, they're just constant values, on the contrary, `editNameWithoutBranch` returns a type of `BOOLEAN` which is type-defined as `unsigned char`, which various from 0 to 255, so the compiler can't eliminate the calling of `strlen`.

If we change the return type to real `boolean`, see what would happen:

``` c
bool editNameWithBoolean(char *name)
{
  return strlen(name);
}
```

Compile it with `gcc -std=c99 -O2`:

``` asm
editNameWithBoolean(char*):
	cmpb	$0, (%rdi)
	setne	%al
	ret
```

It's identical with the if-else version in `-O2`, so, that's it!

## Conclusion

Like the book author mentioned in his book, I think finding/using a better algorithm (strategy) is more important things for a programmer to do, because low-level optimization like the above case is a territory of compilers, it's where compilers are shining, usually doing bettering than most programmers. What compiler can't do better is knowing the purpose of your program, so, **as a rational programmer, we'd better to invest our energy on high-level optimization like algorithms or code designs when it comes to optimize code performance**.

## Reference

* Check out compiling tactics at [Compiler Warnings are Your Friends](/posts/2013-02-26-compiler-warnings-are-your-friends.html).

