---
title: Python 一些有意思的语法
description: "由于 Python 这门语言的动态性（运行时 type checking 等），使得它拥有可以通过 eval 等方法动态生成命令等特性，我们可以充分发挥想象力挖掘出一些很有意思的场景"
time:                   2023-01-10T15:44:28+08:00
heroImage: ""
tags: []
---

https://www.youtube.com/watch?v=t863QfAOmlY&t=67s

1. 同时赋值
```py
a = a[0] = [1]
print(a)
```

2. e 的scope
```py
def foo():
    global e
    try:
        function_that_may_error()
    except Exception as e:
        # sth going on
        pass
```

3. 变量绑定
```py
match (1, 2, 3):
    case (x, y, z) if z % 2 == 0:
    print(f"{x=} {y=} {z=}")
case _:
    print(f"{x+y+z=}")
```
