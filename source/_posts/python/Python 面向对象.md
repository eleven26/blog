---
title: Python 面向对象
date: 2019-12-03 14:12:00
tags: [Python]
---

面向对象各概念：

* 类：用来描述具有相同的属性和方法的对象的集合。它定义了该集合中每个对象所共有的属性和方法。对象是类的实例。
* 方法：类中定义的函数。
* 类常量：类常量在整个实例化的对象中是公用的。类常量定义在类中且在函数体之外。类常量通常不作为实例变量使用。
* 数据成员：类常量或者实例变量用于处理类及其实例对象的相关的数据。
* 方法重写：如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
* 局部变量：在类的声明中，属性是用变量来表示的，这种变量就称为实例变量，实例变量就是一个用 self 修饰的变量。
* 继承：即一个派生类（derived class）继承基类（base class）的字段和方法。继承也允许把一个派生类的对象作为一个基类对象对待。
    例如，有这样一个设计：一个 Dog 类型的对象派生自 Animal 类，这里模拟 "是一个(is-a)" 关系（例图，Dog 是一个 Animal）
* 实例化：创建一个类的实例，类的具体对象。
* 对象：通过类定义的数据结构实例。对象包括两个数据成员（类变量和实例变量）和方法。

> 类的继承机制允许多个基类，派生类可以覆盖基类中的任何方法，方法中可以调用基类中的同名方法。
> 对象可以包含任意数量和类型的数据。


### 类定义

```
class ClassName:
    <statement-1>
    .
    .
    .
    <statement-N>
```

类实例化后，可以使用其属性，实际上，创建一个类之后，可以通过类名访问其属性。


### 类对象

类对象支持两种操作：属性引用和实例化。（这里的类对象是指类本身，而不是类实例化的对象）

属性引用使用和 Python 中所有的属性引用一样的标准语法：`obj.name`

类对象创建后，类命名空间中所有的命名都是有效属性名。所以如果类定义是这样的：

```
#!/usr/bin/python3

class MyClass:
    """一个简单的类实例"""
    i = 12345
    def f(self):
        return 'hello world'

# 实例化类
x = MyClass()

# 访问类的属性和方法
print("MyClass 类的属性 i 为：", x.i)
print("MyClass 类的方法 f 输出为：", x.f())
```

以上创建了一个新的类实例并将该对象赋给局部变量 x，x 为空的对象。

执行以上程序输出结果为：

```
MyClass 类的属性 i 为： 12345
MyClass 类的方法 f 输出为： hello world
```

类有一个 `__init__()` 方法，类的实例化操作会自动调用 `__init__()` 方法。如下实例化类 MyClass，对应的 `__init__()` 方法就会被调用：

```
x = MyClass()
```

当然，`__init__()` 方法可以有参数，参数通过 `__init__()` 传递到类的实例化操作上。例如：

```
#!/usr/bin/python3

class Complex:
    def __init__(self, realpart, imagpart):
        self.r = realpart
        self.i = imagpart

x = Complex(3.0, -4.5)
print(x.r, x.i)  # 输出结果：3.0 -4.5
```

#### self 代表类的实例，而非类

类的方法与普通的函数只有一个特别的区别 -- 它们必须有一个额外的第一个参数名称，按照惯例它的名称是 `self`

```
class Test:
    def prt(self):
        print(self)
        pring(self.__class__)

t = Test()
t.prt()
```

以上实例执行结果为：

```
<__main__.Test object at 0x1053bbb00>
<class '__main__.Test'>
```

从执行结果可以很明显的看出，`self` 代表的是类的实例，代表当前对象的地址，而 `self.class` 则指向类。
self 不是 python 关键字，我们把他换成其他变量名也可以：

```
class Test:
    def prt(runoob):
        print(runoob)
        print(runoob.__class__)

t = Test()
t.prt()
```

以上实例执行结果为：

```
<__main__.Test object at 0x100a32588>
<class '__main__.Test'>
```


### 类的方法

在类的内部，使用 def 关键字来定义一个方法，与一般函数定义不同，类方法必须包含参数 self，且为第一个参数，self 代表的是类的实例。

```
#!/usr/bin/python

# 类定义
class People:
    # 定义类基本属性
    name = ''
    age = 0
    # 定义私有属性，私有属性在外部无法直接进行访问
    __weight = 0
    # 定义构造方法
    def __init__(self, n, a, w):
        self.name = n
        self.age = a
        self.__weight = w
    def speak(self):
        print("{} 说: 我 {} 岁。".format(self.name, self.age))

# 实例化类
p = People('runoob', 10, 30)
p.speak()
```

执行以上程序输出结果为：

```
runoob 说: 我 10 岁。
```


### 继承

```
class DerivedClassName(BaseClassName):
    pass
```

需要注意圆括号中基类的顺序，若是基类中有相同的方法名，而在子类使用时未指定，python 从左至右搜索，即方法在子类中未找到时，从左到右查找基类中是否包含方法。

BaseClassName(基类名) 必须与派生类定义在同一个作用域内。除了类，还可以用表达式，基类定义在另一个模块中时这一点非常有用：

```
class DerivedClassName(modname.BaseClassName):
```

```
#!/usr/bin/python3

# 类定义
class People:
    # 定义基本属性
    name = ''
    age = 0
    # 定义私有属性，私有属性在类外部无法直接进行访问
    __weight = 0
    # 定义构造方法
    def __init__(self, n, a, w):
        self.name = n
        self.age = a
        self.__weight = w
    def speak(self):
        print("{} 说：我 {} 岁。".format(self.name, self.age))

# 单继承示例
class Student(People):
    grade = ''
    def __init__(self, n, a, w, g):
        # 调用父类的构造函数
        People.__init__(self, n, a, w)
        self.grade = g
    # 覆写父类的方法
    def speak(self):
        print("{} 说：我 {} 岁了，我在读 {} 年级。".format(self.name, self.age, self.grade))

s = Student('ken', 10, 60, 3)
s.speak()
```

执行以上程序输出结果为：

```
ken 说：我 10 岁了，我在读 3 年级。
```


### 多继承

Python 同样支持多继承。多继承的类定义形式如下例：

```
class DrivedClassName(Base1, Base2, Base3):
    pass
```

需要注意圆括号中父类的顺序，若是父类中有相同的方法名，而在子类使用时未指定，python 从左至右搜索，即方法在子类中未找到时，从左到右查找父类中是否包含方法。

```
#!/usr/bin/python3

# 类定义
class people:
    # 定义基本属性
    name = ''
    age = 0
    # 定义私有属性，私有属性在类外部无法直接进行访问
    __weight = 0
    # 定义构造方法
    def __init__(self, n, a, w):
        self.name = n
        self.age = a
        self.__weight = w
    def speak(self):
        print("{} 说：我 {} 岁。".format(self.name, self.age))

# 单继承示例
class student(people):
    grade = ''
    def __init__(self, n, a, w, g):
        # 调用父类的构造函数
        people.__init__(self, n, a, w)
        self.grade = g
    # 覆写父类的方法
    def speak(self):
        print("{} 说: 我 %d 岁了，我在读 {} 年级。".format(self.name, self.age, self.grade))

# 另一个类，多重继承之前的准备
class speaker():
    topic = ''
    name = ''
    def __init__(self, n, t)
        self.name = n
        self.topic = t
    def speak(self):
        print("我叫 {}，我是一个演说家，我演讲的主题是 {}".format(self.name, self.topic))

# 多重继承
class sample(speaker, student):
    a = ''
    def __init__(self, n, a, w, g, t):
        student.__init__(self, n, a, w, g)
        speaker.__init__(self, n, t)

test = sample('Tim', 25, 80, 4, "Python")
test.speak() # 方法同名，默认调用的是在括号中排前的父类的方法
```

执行以上程序输出结果为：

```
我叫 Tim，我是一个演说家，我演讲的主题是 Python
```


### 方法重写

如果你的父类方法的功能不能满足你的需求，你可以在子类重写你父类的方法。如：

```
class Parent:
    def my_method(self):
        print('调用父类方法')

class Child(Parent):
    def my_method(self):
        print('调用子类方法')

c = Child() # 子类实例
c.my_method() # 子类调用重写方法
super(Child, c).my_method() # 用子类对象调用父类已被覆盖的方法
```

super 函数是用于调用父类的一个方法。

执行以上程序输出结果为：

```
调用子类方法
调用父类方法
```


### 类属性与方法

#### 类的私有属性

__private_attrs: 两个下划线开头，声明该属性为私有，不能在类的外部被使用或者直接访问。在类内部的方法中使用时 self.__private_attrs。

#### 类的方法

在类的内部，使用 def 关键字来定义一个方法，与一般函数定义不同，类方法必须包含参数 self，且为第一个参数，self 代表的是类的实例。
self 的名字并不是规定死的，也可以使用 this，但是最好还是按照约定使用 self。

#### 类的私有方法：

__private_method: 两个下划线开头，声明该方法为私有方法，只能在类的内部调用，不能在类的外部调用。self.__private_method。

#### 实例

```
class JustCounter:
    __secret_count = 0 # 私有变量
    public_count = 0 # 公开变量
    
    def count(self):
        self.__secret_count += 1
        self.public_count += 1
        print(self.__secret_count)

counter = JustCounter()
counter.count()
counter.count()
print(counter.public_count)
print(counter.__secret_count) # 报错
```

以上实例输出结果为：

```
1
Traceback (most recent call last):
2
  File "/Users/ruby/Code/DevAdmin/python-deployer/tests/oop/__init__.py", line 17, in <module>
    print(counter.__secret_count)  # 报错
AttributeError: 'JustCounter' object has no attribute '__secret_count'
2
```


类的私有方法实例如下：

```
class Site:
    def __init__(self, name, url):
        self.name = name  # 公开变量
        self.__url = url  # 私有变量

    def who(self):
        print('name:', self.name)
        print('url:', self.__url)

    def __foo(self):  # 私有方法
        print('这是私有方法')

    def foo(self):  # 公开方法
        print('这是公共方法')
        self.__foo()

x = Site('百度', 'https://www.baidu.com')
x.who()  # 正常输出
x.foo()  # 正常输出
x.__foo() # 报错
```

以上实例执行结果：

```
Traceback (most recent call last):
  File "/Users/ruby/Code/DevAdmin/python-deployer/tests/oop/__init__.py", line 22, in <module>
    x.__foo() # 报错
AttributeError: 'Site' object has no attribute '__foo'
name: 百度
url: https://www.baidu.com
这是公共方法
这是私有方法
```

### 类的专有方法

* __init__: 构造函数，在生成对象时调用

* __del__: 析构函数，释放对象时使用

* __repr__: 打印，转换

* __setitem__: 按照索引赋值

* __getitem__: 按照索引获取值

* __len__: 获得长度

* __cmp__: 比较运算

* __call__: 函数调用

* __add__: 加运算

* __sub__: 减运算

* __mul__: 乘运算

* __truediv__: 除运算

* __mod__: 求余运算

* __pow__: 乘方


### 运算符重载

Python 同样支持运算符重载

```
class Vector:
    def __init__(self, a, b):
        self.a = a
        self.b = b

    def __str__(self):
        return 'Vector ({}, {})'.format(self.a, self.b)

    def __add__(self, other):
        return Vector(self.a + other.a, self.b + other.b)

v1 = Vector(2, 10)
v2 = Vector(5, -2)
print(v1 + v2)
```

以上代码执行结果如下：

```
Vector (7, 8)
```
