[目录](README.md)

# Iterable 与 Iterator
* ## for...in 循环的本质
    - 一个对象 X 是“可迭代的（iterable）”，就是说这个对象支持一个名为         __iter__() 的方法，这个方法返回一种叫“迭代器（iterator）”的对象
    - 一个迭代器（ietrator）必须支持一个名为 __next__() 的方法，这个方法每    次返回 X 里的下一个元素，直到所有元素被遍历正好一次
> 凡是 iterable 的东西，都可以放在 for...in 循环中进行迭代操作。

```python
class squares:
    # 实例变量 self.base 用来记录当前平方数的基数，这个基数应该从 1 开始逐渐递增。
    # 然后定义了三个标准方法：
    #   __init__() 把 self.base 设置为初始值 0；
    #   __iter__() 返回迭代器，这里就是自己，我们直接返回 self；
    #   __next__() 是迭代器主方法，这里我们先把 self.base 加一，然后返回它的平方，由于初始为 0，第一次调用返回的是 1 的平方。
    # 最后定义 next 作为 __next__ 的别名，方便调用（也与 Python 2.x 兼容）。
    def __init__(self):
        self.base = 0
        
    def __iter__(self):
        return self

    def __next__(self):
        self.base += 1
        return self.base * self.base

    next = __next__

# 创建 squares 类的实例对象 iter，然后就可以调用其 next() 方法来迭代
iter = squares()
iter.next()

```

> 凡是看上去很“常规”和“常见”的需求，一般都会有优秀的通用解决方案，我们找出来   用比自己写好，这叫“不重复发明轮子（Don't Reinvent The Wheel）”原理。这些   标准解决方案在 itertools 模块中，用于对无限迭代器进行切片的方法叫做         islice ()。

>  islice() 做的事情相当于从迭代器产生的无限序列中切出一段来，它返回的也是一个迭代器，返回的迭代器输出的就是有头有尾、有限的序列了。

>  islice() 接受三个参数：一个迭代器实例，切片的起始和结束序号（序号从 0 开    始算）；切的时候和 range() 函数的算法一样，包含起始序号那一项，但不包括结    束序号的一项。比如 islice(f, 0, 10) 就是从迭代器 f 生成的序列中切出第 1     到第 10 项，返回这 10 项对应的有限迭代器。

>  其实 islice() 还可以有第四个参数：步长 step，缺省值为 1，就是起始到结束     范围内每一项都保留；如果指定了别的 step 值，那么就是每隔 step-1 项输出     一项。基本逻辑和 range() 函数一样。

>  因为 islice() 返回的还是迭代器，所以可以直接用于 for...in 循环中

```python
from itertools import islice

f = fib()
for n in islice(f, 0, 10):
    print(n)
```