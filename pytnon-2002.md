[目录](README.md)   [下一章](python-1011.md)    [上一章](python-2001.md)
# 深入解释yield和Generators（生成器）
> 生成器和yield关键字可能是Python里面最强大的最难理解的概念之一（或许没有之一）， 但是并不妨碍yield成为Python里面最强大的关键字，对于初学者来讲确实非常难于理解，来看一篇关于yield的国外大牛写的文章，让你快速理解yield。 文章有点长，请耐心读完， 过程中有些例子， 循序渐进，让你不觉得枯燥。
# 生成器
> 生成器是通过一个或多个yield表达式构成的函数，每一个生成器都是一个迭代器（但是迭代器不一定是生成器）。如果一个函数包含yield关键字，这个函数就会变为一个生成器。生成器并不会一次返回所有结果，而是每次遇到yield关键字后返回相应结果，并保留函数当前的运行状态，等待下一次的调用。由于生成器也是一个迭代器，那么它就应该支持next方法来获取下一个值。（也可以使用.__next__()属性, 在python2 中是.next()）
# 协程与子例程
> 我们调用一个普通的Python函数时，一般是从函数的第一行代码开始执行，结束于return语句、异常或者函数结束（可以看作隐式的返回None）。一旦函数将控制权交还给调用者，就意味着全部结束。函数中做的所有工作以及保存在局部变量中的数据都将丢失。再次调用这个函数时，一切都将从头创建。对于在计算机编程中所讨论的函数，这是很标准的流程。这样的函数只能返回一个值，不过，有时可以创建能产生一个序列的函数还是有帮助的。要做到这一点，这种函数需要能够“保存自己的工作”。我说过，能够“产生一个序列”是因为我们的函数并没有像通常意义那样返回。return隐含的意思是函数正将执行代码的控制权返回给函数被调用的地方。而"yield"的隐含意思是控制权的转移是临时和自愿的，我们的函数将来还会收回控制权。

> 在Python中，拥有这种能力的“函数”被称为生成器，它非常的有用。生成器（以及yield语句）最初的引入是为了让程序员可以更简单的编写用来产生值的序列的代码。 以前，要实现类似随机数生成器的东西，需要实现一个类或者一个模块，在生成数据的同时保持对每次调用之间状态的跟踪。引入生成器之后，这变得非常简单。为了更好的理解生成器所解决的问题，让我们来看一个例子。在了解这个例子的过程中，请始终记住我们需要解决的问题：生成值的序列。注意：在Python之外，最简单的生成器应该是被称为协程（coroutines）的东西。在本文中，我将使用这个术语。请记住，在Python的概念中，这里提到的协程就是生成器。Python正式的术语是生成器；协程只是便于讨论，在语言层面并没有正式定义。

# 例子：有趣的素数
假设你的老板让你写一个函数，输入参数是一个int的list，返回一个可以迭代的包含素数1 的结果。记住，迭代器（Iterable） 只是对象每次返回特定成员的一种能力。
你肯定认为"这很简单"，然后很快写出下面的代码：
```python

def get_primes(input_list):
    result_list = list()
    for element in input_list:
        if is_prime(element):
            result_list.append()
    return result_list
# 或者更好一些的...
def get_primes(input_list):
    return (element for element in input_list if is_prime(element))
# 下面是 is_prime 的一种实现...
def is_prime(number):
    if number > 1:
        if number == 2:
            return True
        if number % 2 == 0:
            return False
        for current in range(3, int(math.sqrt(number) + 1), 2):
            if number % current == 0: 
                return False
        return True
    return False
```
上面 is_prime 的实现完全满足了需求，所以我们告诉老板已经搞定了。她反馈说我们的函数工作正常，正是她想要的。
# 处理无限序列
噢，真是如此吗？过了几天，老板过来告诉我们她遇到了一些小问题：她打算把我们的get_primes函数用于一个很大的包含数字的list。实际上，这个list非常大，仅仅是创建这个list就会用完系统的所有内存。为此，她希望能够在调用get_primes函数时带上一个start参数，返回所有大于这个参数的素数（也许她要解决 Project Euler problem 10）。
我们来看看这个新需求，很明显只是简单的修改get_primes是不可能的。 自然，我们不可能返回包含从start到无穷的所有的素数的列表 (虽然有很多有用的应用程序可以用来操作无限序列)。看上去用普通函数处理这个问题的可能性比较渺茫。
在我们放弃之前，让我们确定一下最核心的障碍，是什么阻止我们编写满足老板新需求的函数。通过思考，我们得到这样的结论：函数只有一次返回结果的机会，因而必须一次返回所有的结果。得出这样的结论似乎毫无意义；“函数不就是这样工作的么”，通常我们都这么认为的。可是，不学不成，不问不知，“如果它们并非如此呢？”
想象一下，如果get_primes可以只是简单返回下一个值，而不是一次返回全部的值，我们能做什么？我们就不再需要创建列表。没有列表，就没有内存的问题。由于老板告诉我们的是，她只需要遍历结果，她不会知道我们实现上的区别

不幸的是，这样做看上去似乎不太可能。即使是我们有神奇的函数，可以让我们从n遍历到无限大，我们也会在返回第一个值之后卡住：
```python

def get_primes(start):
    for element in magical_infinite_range(start):
        if is_prime(element):
            return element
```
假设这样去调用get_primes：
```python

def solve_number_10():
    # She *is* working on Project Euler #10, I knew it!
    total = 2
    for next_prime in get_primes(3):
        if next_prime < 2000000:
            total += next_prime
        else:
            print(total)
            return
```
显然，在get_primes中，一上来就会碰到输入等于3的，并且在函数的第4行返回。与直接返回不同，我们需要的是在退出时可以为下一次请求准备一个值。
不过函数做不到这一点。当函数返回时，意味着全部完成。我们保证函数可以再次被调用，但是我们没法保证说，“呃，这次从上次退出时的第4行开始执行，而不是常规的从第一行开始”。函数只有一个单一的入口：函数的第1行代码
# 走进生成器
这类问题极其常见以至于Python专门加入了一个结构来解决它：生成器。一个生成器会“生成”值。创建一个生成器几乎和生成器函数的原理一样简单。一个生成器函数的定义很像一个普通的函数，除了当它要生成一个值的时候，使用yield关键字而不是return。如果一个def的主体包含yield，这个函数会自动变成一个生成器（即使它包含一个return）。除了以上内容，创建一个生成器没有什么多余步骤了。
生成器函数返回生成器的迭代器。这可能是你最后一次见到“生成器的迭代器”这个术语了， 因为它们通常就被称作“生成器”。要注意的是生成器就是一类特殊的迭代器。作为一个迭代器，生成器必须要定义一些方法(method)，其中一个就是__next__()【注意： 在python2中是： next() 方法】。如同迭代器一样，我们可以使用next()函数来获取下一个值。
为了从生成器获取下一个值，我们使用next()函数，就像对付迭代器一样。
(next()会操心如何调用生成器的__next__()方法)。既然生成器是一个迭代器，它可以被用在for循环中。
每当生成器被调用的时候，它会返回一个值给调用者。在生成器内部使用yield来完成这个动作(例如yield 7)。为了记住yield到底干了什么，最简单的方法是把它当作专门给生成器函数用的特殊的return(加上点小魔法)。**
yield就是专门给生成器用的return(加上点小魔法)。
下面是一个简单的生成器函数：
```python
>>> def simple_generator_function():
>>>    yield 1
>>>    yield 2
>>>    yield 3
```
这里有两个简单的方法来使用它：
```python
>>> for value in simple_generator_function():
>>>     print(value)
1
2
3
>>> our_generator = simple_generator_function()
>>> next(our_generator)
1
>>> next(our_generator)
2
>>> next(our_generator)
3
```
# 魔法?
那么神奇的部分在哪里?我很高兴你问了这个问题!当一个生成器函数调用yield，生成器函数的“状态”会被冻结，所有的变量的值会被保留下来，下一行要执行的代码的位置也会被记录，直到再次调用next()。一旦next()再次被调用，生成器函数会从它上次离开的地方开始。如果永远不调用next()，yield保存的状态就被无视了。
我们来重写get_primes()函数，这次我们把它写作一个生成器。注意我们不再需要magical_infinite_range函数了。使用一个简单的while循环，我们创造了自己的无穷串列。
```python
def get_primes(number):
    while True:
        if is_prime(number):
            yield number
        number += 1
```
如果生成器函数调用了return，或者执行到函数的末尾，会出现一个StopIteration异常。 这会通知next()的调用者这个生成器没有下一个值了(这就是普通迭代器的行为)。这也是这个while循环在我们的get_primes()函数出现的原因。如果没有这个while，当我们第二次调用next()的时候，生成器函数会执行到函数末尾，触发StopIteration异常。一旦生成器的值用完了，再调用next()就会出现错误，所以你只能将每个生成器的使用一次。下面的代码是错误的：
```python
>>> our_generator = simple_generator_function()
>>> for value in our_generator:
>>>     print(value)
>>> # 我们的生成器没有下一个值了...
>>> print(next(our_generator))
Traceback (most recent call last):
  File "<ipython-input-13-7e48a609051a>", line 1, in <module>
    next(our_generator)
StopIteration
>>> # 然而，我们总可以再创建一个生成器
>>> # 只需再次调用生成器函数即可
>>> new_generator = simple_generator_function()
>>> print(next(new_generator)) # 工作正常
1
```
因此，这个while循环是用来确保生成器函数永远也不会执行到函数末尾的。只要调用next()这个生成器就会生成一个值。这是一个处理无穷序列的常见方法（这类生成器也是很常见的）。
## 执行流程
让我们回到调用get_primes的地方：solve_number_10。
```python
def solve_number_10():
    # She *is* working on Project Euler #10, I knew it!
    total = 2
    for next_prime in get_primes(3):
        if next_prime < 2000000:
            total += next_prime
        else:
            print(total)
            return
```
我们来看一下solve_number_10的for循环中对get_primes的调用，观察一下前几个元素是如何创建的有助于我们的理解。当for循环从get_primes请求第一个值时，我们进入get_primes，这时与进入普通函数没有区别。
进入第三行的while循环
停在if条件判断（3是素数）
通过yield将3和执行控制权返回给solve_number_10
接下来，回到insolve_number_10：
for循环得到返回值3
for循环将其赋给next_prime
total加上next_prime
for循环从get_primes请求下一个值
这次，进入get_primes时并没有从开头执行，我们从第5行继续执行，也就是上次离开的地方。
```python

def get_primes(number):
    while True:
        if is_prime(number):
            yield number
        number += 1 # <<<<<<<<<<
```
最关键的是，number还保持我们上次调用yield时的值（例如3）。记住，yield会将值传给next()的调用方，同时还会保存生成器函数的“状态”。接下来，number加到4，回到while循环的开始处，然后继续增加直到得到下一个素数（5）。我们再一次把number的值通过yield返回给solve_number_10的for循环。这个周期会一直执行，直到for循环结束（得到的素数大于2,000,000）。

# 总结
关键点：
generator是用来产生一系列值的
yield则像是generator函数的返回结果
yield唯一所做的另一件事就是保存一个generator函数的状态
generator就是一个特殊类型的迭代器（iterator）
和迭代器相似，我们可以通过使用next()来从generator中获取下一个值
通过隐式地调用next()来忽略一些值