---
title: Python 装饰器入门
date: 2019-12-04 10:41:00
tags: [Python]
---

## 前言

首先来看看我们正常调用一个函数的过程：

```
def add(a, b):
    print('{} + {} = {}'.format(a, b, a + b))

add(3, 5)
```

执行以上代码输出:

```
3 + 5 = 8
```

后来，我们的 add 方法做的东西越来越多，然后我们想在 add 方法执行之前打印执行开始时间，执行之后打印结束时间，并打印总耗时时间。

这个需求很简单，在原来的基础上 add 方法调用前后加上几个 print 语句就行了:


```
import time

def add(a, b):
    time.sleep(1)
    print('{} + {} = {}'.format(a, b, a + b))

start_time = time.time()
print('add 方法开始执行...')

add(3, 5)

end_time = time.time()
print('add 方法执行完毕. 总耗时: {}'.format(end_time - start_time))
```

正如我们期待的那样，输出了我们想要的结果：

```
add 方法开始执行...
3 + 5 = 8
add 方法执行完毕. 总耗时: 1.004680871963501
```

后来，我们又发现，又几个地方都需要统计耗时，然后我们想到了，干脆把 print 的调用放到 add 里面:


```
import time

def add(a, b):
    start_time = time.time()
    print('add 方法开始执行...')

    time.sleep(1)
    print('{} + {} = {}'.format(a, b, a + b))

    end_time = time.time()
    print('add 方法执行完毕. 总耗时: {}'.format(end_time - start_time))

add(3, 5)
```

再一次，代码如我们期待的那样实现了我们的想法:

```
add 方法开始执行...
3 + 5 = 8
add 方法执行完毕. 总耗时: 1.002723217010498
```

过了一段时间之后，我们发现，另一个 sub 方法我们也想统计，这时候，我们会想，把之前写的那几行 print 代码复制粘贴过去就行了，
但是我们转念一想，要是以后有其他方法也要统计，那岂不是太蠢了。

不用怕，Python 为这种想法提供了支持，Python 利用装饰器可以给我们的函数 "装饰" 一番。

装饰器是声明？其实就是一个返回函数的函数，把我们的函数作为参数传给装饰器，*然后实际调用的时候其实调用的是装饰器函数*。

我们知道，函数名可以作为参数传递，然后利用函数名加括号的方式调用。

在装饰器里面，我们可以做一些其他处理，然后调用被装饰的函数，当然不调用也是可以的，但是没什么意义。


## 装饰器语法糖

@ 符号就是装饰器的语法糖。

它放在一个函数开始定义的地方，它就像一顶帽子一样戴在这个函数的头上。和这个函数绑定在一起。

我们在调用这个函数的时候，第一件事并不是执行这个函数，而是将这个函数作为参数传入它头顶上这顶帽子，这顶帽子称为装饰函数或装饰器。


装饰器的使用方法很固定：

* 先定义一个装饰函数（帽子）（也可以用类、偏函数实现）

* 再定义你的业务函数、或者类（人）

* 最后把这顶帽子戴在这个人的头上


## 一个最简单的装饰器

```
import time


def logger(func):
    def wrapper(*args, **kwargs):
        print('执行 {}'.format(func.__name__))
        func(*args, **kwargs)

    return wrapper


@logger
def add(a, b):
    print('{} + {} = {}'.format(a, b, a + b))


add(1, 2)
```

执行以上代码的输出如下：

```
执行 add
1 + 2 = 3
```

我们可以把装饰器起作用的过程看作（伪代码）:

```
wrapper = logger(add)  # 拿到装饰后的函数
wrapper(*args, **kwargs)  # 实际调用的函数，这个函数里面会调用被装饰的函数，同时我们也在里面做了其他操作（print）
```


## 入门用法


### 日志打印器

功能：

* 在函数执行之前，先打印一行日志，告知开始执行

* 在函数执行完，告知执行完毕

```
def logger(func):
    def wrapper(*args, **kwargs):
        print('开始执行函数 {}'.format(func.__name__))
        
        # 真正执行的是这行
        func(*args, **kwargs)

        print('函数 {} 执行完毕！'.format(func.__name__))
    return wrapper


@logger
def add(x, y):
    print('{} + {} = {}'.format(x, y, x+ y))


add(1, 2)
```


输出：

```
开始执行函数 add
1 + 2 = 3
函数 add 执行完毕！
```

### 时间计算器

功能：计算一个函数的执行时长

```
import time

def timer(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        
        func(*args, **kwargs)
        
        t2 = time.time()
        cost_time = t2 - t1
        print('花费时间: {}秒'.format(cost_time))

    return wrapper


@timer
def want_sleep(sleep_time):
    time.sleep(sleep_time)


want_sleep(1)
```

输出：

```
花费时间: 1.0049278736114502秒
```


## 带参数的函数装饰器

上面的装饰器是不能接收参数的，只适用于一些简单的场景。不传参的装饰器，只能对被装饰的函数执行固定逻辑。

装饰器本身是一个函数，既然作为一个函数都不能携带函数，那这个函数的功能就很受限，只能执行固定的逻辑。
这无疑是不合理的。而如果我们要用到两个内容大体一致，只是某些地方不同的逻辑。
不传参的话，我们就要写两个装饰器。

那么如何实现装饰器传参呢？装饰器函数体需要两层嵌套：

```
def say_hello(country):
    def wrapper(func):
        def deco(*args, **kwargs):
            if country == 'china':
                print('你好！')
            elif country == 'america':
                print('hello.')
            else:
                return
            
            func(*args, **kwargs)
        return deco
    return wrapper


@say_hello('china')
def chinese():
    print('我来自中国')


@say_hello('america')
def american():
    print('I am from America')


chinese()
american()
```

输出：

```
你好！
我来自中国
hello.
I am from America
```

我们可以把以上过程看作如下伪代码：

```
decorator = say_hello('china')  # 这一层用以接收装饰器参数
wrapper = decorator(chinese)    # wrapper 是一个闭包，我们在装饰器传的参数已经被这个闭包保留下来了
func = wrapper()                # 调用 wrapper，取得装饰后的函数
func(*args, **kwargs)           # 调用装饰后的函数(deco)
```

简单一点，就是：

```
(say_hello('china')(func))()
```

而不用参数的时候是：

```
say_hello(func)()
```


## 高阶用法: 不带参数的类装饰器

以上都是基于函数实现的装饰器，还有基于类实现的装饰器。

基于类装饰器的实现，必须实现 __call__ 和 __init__ 两个内置函数。

* __init__: 接收被装饰的函数

* __call__: 实现装饰逻辑

```
class logger(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("[INFO]: the function [{func}] is running...".format(func=self.func.__name__))
        return self.func(*args, **kwargs)


@logger
def say(something):
    print("say {}!".format(something))


say("hello")
```

输出：

```
[INFO]: the function say() is running...
say hello!
```

我们可以把上面的过程看作：

```
(new logger(func))(*args, **kwargs)
```


## 高阶用法：带参数的类装饰器

上面不带参数的例子，只能打印 INFO 级别的日志，正常情况下，我们还需要打印 DEBUG WARNING 等级别的日志。

这就需要给类装饰器传入参数，给这个函数指定级别了。

带参数和不带参数的类装饰器有很大的不同。

* __init__: 不再接收被装饰函数，而是接收传入参数

* __call__: 接收被装饰函数，实现装饰逻辑

```
class logger(object):
    def __init__(self, level='INFO'):
        self.level = level

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            print('[{level}]: the function {func} is running...'.format(level=self.level, func=self.func.__name__))
            func(*args, **kwargs)
        return wrapper


@logger(level='WARNING')
def say(something):
    print("say {}!".format(something))


say('hello')
```

输出：

```
[WARNING]: the function say1 is running...
say hello!
```

我们可以把上面的过程看作：

```
(new logger1(level='WARNING'))(func)(*args, **kwargs)
```

