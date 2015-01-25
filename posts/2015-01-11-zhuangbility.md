----
title: The Many Adventures of Zhuangbility
tags: zhuangbility, mumble
----

# Origin

I should have started this blog long long ago, but failed. Last week, Eric asked Zhang Yu and me to actually do something, to record some thoughts and funny things, so now, here it is.

# Hakyll

In fact, I have tried Hakyll once two years ago, but gave up quickly, due to my laziness. :p Nevertheless, Haskel is good for zhuangbility, so I'm back!

# Self-Cultivation of me, a coder

Yes, it's a blog of a coder, is this a good title?

# Test CSS:

```python
from functools import reduce

_Meta = ((1000, 'M'),
         (900, 'CM'), (500, 'D'), (400, 'CD'), (100, 'C'),
         (90, 'XC'), (50,'L'), (40, 'XL'), (10, 'X'),
         (9, 'IX'), (5, 'V'), (4, 'IV'), (1, 'I'))

def romanize(number):
    def next(result, remain, radix, symbol):
        while remain >= radix:
            result, remain = (result + symbol, remain - radix)
        return result, remain

    r, _ = reduce(lambda r, m:next(*(r+m)), _Meta, ('', number))

    return r
```

# References

- [Show content of posts in index.html by teaser](http://reichertbrothers.com/blog/posts/2014-04-08-hakyll-teasers.html)
- [Enable tags](http://javran.github.io/posts/2014-03-01-add-tags-to-your-hakyll-blog.html)

