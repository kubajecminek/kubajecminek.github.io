---
title: "Quine Exercise"
date: 2024-09-25T21:50:26+02:00
Tags:
  - Programming
---
After reading this fantastic
[essay](https://www.cs.cmu.edu/~rdriley/487/papers/Thompson_1984_ReflectionsonTrustingTrust.pdf)
by Ken Thompson, I’ve decided to take on the Quine challenge. According to
Wikipedia:

> A quine is a computer program that takes no input and produces a copy of its
> own source code as its only output.

It might feel easy at first, but once you try it, you’ll quickly encounter the
infinite recursion problem. The following is definitely not the shortest (and
most likely not the original) solution, but I had a great time writing it.

```python
a = [97, 32, 61, 32, 63, 10, 10, 102, 111, 114, 32, 120, 32, 105, 110, 32, 97, 58, 10, 32, 32, 32, 32, 112, 114, 105, 110, 116, 40, 34, 37, 114, 34, 32, 37, 32, 97, 44, 32, 101, 110, 100, 61, 34, 34, 41, 32, 105, 102, 32, 120, 32, 61, 61, 32, 54, 51, 32, 101, 108, 115, 101, 32, 112, 114, 105, 110, 116, 40, 99, 104, 114, 40, 120, 41, 44, 32, 101, 110, 100, 61, 34, 34, 41, 10]

for x in a:
    print("%r" % a, end="") if x == 63 else print(chr(x), end="")
```

We can check that it works with somethig like this:

```shell
python quine.py | git diff --no-index -- quine.py -
```

I encourage you to write your own. It is very rewarding. You might even feel
so hyped that you’ll want to write a blog post about it.