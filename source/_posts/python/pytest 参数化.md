---
title: pytest 参数化
date: 2020-03-26 10:32:00
tags: [Python, pytest]
---

0. 参数化是什么?
1. pytest 参数化的几种写法
2. 装饰测试类
3. 装饰测试函数
4. ids


## 参数化是什么？

在单元测试中，我们有时候可能需要使用多组测试数据来测试同一个功能，在 pytest 中可以使用 `@pytest.mark.parametrize` 装饰器来装饰一个类或者方法来传递多组测试数据来测试同一个功能。


## pytest 参数化的几种写法

1. 使用多个形参接收参数化数据

test_1.py

```
import pytest

data = [
    (1, 2, 3),
    (4, 5, 9),
]


def add(a, b):
    return a + b


@pytest.mark.parametrize('a, b, expect', data)
def test_parametrize_1(a, b, expect):
    print("\n参数: a={}, b={}\n".format(a, b))
    assert add(a, b) == expect

```

运行代码:

```
pytest -sv test_1.py
```

输出:

```
tests/aa/test_aa.py::test_parametrize_1[1-2-3] 
参数: a=1, b=2

PASSED
tests/aa/test_aa.py::test_parametrize_1[4-5-9] 
参数: a=4, b=5

PASSED
```

> 在这个例子中，我们定义了一个数组，里面保存了两个元组，同时在测试函数上面使用了 `@pytest.mark.parametrize('a, b, expect', data)` 来装饰。
> 这个情况下，pytest 会循环 data 数据，把里面的每一项的数据拿出来，传递给 `test_parametrize_1` 运行。

实际效果可以看作如下伪代码：

```
for value in data:
    for a, b, expect in value:
        test_parametrize_1(a, b, expect)
```


如果只有一个参数，那就更简单了：

```
import pytest

data = [
    1, 3
]


@pytest.mark.parametrize('a', data)
def test_parametrize_1(a):
    print("\n参数: a={}\n".format(a))
```

输出：

```
tests/aa/test_aa.py::test_parametrize_1[1] 
参数: a=1

PASSED
tests/aa/test_aa.py::test_parametrize_1[3] 
参数: a=3

PASSED
```


2. 使用一个形参接收参数化数据

```
import pytest

data = [
    [1, 2, 3],
    [4, 5, 9],
]


@pytest.mark.parametrize('value', data)
def test_parametrize_1(value):
    print("\n测试数据为\n{}".format(value))
    actual = value[0] + value[1]
    assert actual == value[2]
```

输出：

```
tests/aa/test_aa.py::test_parametrize_1[value0] 
测试数据为
(1, 2, 3)
PASSED
tests/aa/test_aa.py::test_parametrize_1[value1] 
测试数据为
(4, 5, 9)
PASSED
```

这里的 value 参数直接接收了 data 里面的每一项。

和第一种 `@pytest.mark.parametrize('a, b, expect', data)` 的区别是，这里的 value 接收了 `[a, b, expect]` 这个数组作为值。


## 装饰测试类

`@pytest.mark.parametrize` 也可以用于装饰类，这样一来，该类中的测试方法就都有了这些参数，等同于给该类中的每一个测试方法加上 `@pytest.mark.parametrize`。

例子

```
import pytest

data = [
    [1, 2, 3],
    [4, 5, 9],
]


@pytest.mark.parametrize('value', data)
class TestParametrize:
    def test_parametrize_1(self, value):
        print("\ntest_parametrize_1测试数据为\n{}".format(value))

    def test_parametrize_2(self, value):
        print("\ntest_parametrize_2 测试数据为\n{}".format(value))

```


输出

```
tests/aa/test_aa.py::TestParametrize::test_parametrize_1[value0] 
test_parametrize_1 测试数据为
[1, 2, 3]
PASSED
tests/aa/test_aa.py::TestParametrize::test_parametrize_1[value1] 
test_parametrize_1 测试数据为
[4, 5, 9]
PASSED
tests/aa/test_aa.py::TestParametrize::test_parametrize_2[value0] 
test_parametrize_2 测试数据为
[1, 2, 3]
PASSED
tests/aa/test_aa.py::TestParametrize::test_parametrize_2[value1] 
test_parametrize_2 测试数据为
[4, 5, 9]
PASSED
```


## 装饰测试函数

这个在第一小节中已经提及


## ids

我们在上面的测试中发现一些输出内容如下面的格式：

```
tests/aa/test_aa.py::TestParametrize::test_parametrize_1[value0]
```

后面有个方括号来表示当前测试的是第几组参数。这个我们可以自定义格式：


```
import pytest

data = [
    [1, 2, 3],
    [4, 5, 9],
]


# 这个格式会替代输出中方括号的内容
ids = ["a={}, b={}, expect={}".format(a, b, expect) for a, b, expect in data]


@pytest.mark.parametrize('value', data, ids=ids)
class TestParametrize:
    def test_parametrize_1(self, value):
        print("\ntest_parametrize_1 测试数据为\n{}".format(value))

    def test_parametrize_2(self, value):
        print("\ntest_parametrize_2 测试数据为\n{}".format(value))

```

输出

```
tests/aa/test_aa.py::TestParametrize::test_parametrize_1[a=1, b=2, expect=3] 
test_parametrize_1 测试数据为
[1, 2, 3]
PASSED
tests/aa/test_aa.py::TestParametrize::test_parametrize_1[a=4, b=5, expect=9] 
test_parametrize_1 测试数据为
[4, 5, 9]
PASSED
tests/aa/test_aa.py::TestParametrize::test_parametrize_2[a=1, b=2, expect=3] 
test_parametrize_2 测试数据为
[1, 2, 3]
PASSED
tests/aa/test_aa.py::TestParametrize::test_parametrize_2[a=4, b=5, expect=9] 
test_parametrize_2 测试数据为
[4, 5, 9]
PASSED

```

这样一来我们的输出就更加的直观了。

> 输出的时候需要添加 `-sv` 参数，如 `pytest -sv tests/aa/test_aa.py`
