---
title: Ruby 中 __set 和 __get 实现
date: 2019-12-23 18:57:00
tags: [Ruby]
---

PHP 中有 __set、__get、__call 等方法来实现一些动态的特性，比如动态的对象属性等。

本文讲述 Ruby 中对应这几个方法的实现。

* `__call` 对应 Ruby 中 `method_missing(name, *arguments)`

* `__set` 对应 Ruby 中 `method_missing(name=)`

* `__get` 对应 Ruby 中 `method_missing(name)`


> 注意：`method_missing` 里面的 name 是一个符号，不是字符串。


来验证一下：

先定义一个类，里面只有一个 `method_missing` 方法

```
class Person
  def method_missing(name, *arguments)
    puts "name: #{name}"
    print 'arguments:'
    p arguments

    if name == :test        # 这里需要注意的是，name 是一个符号
      puts 'this is a test'
    end
  end
end
```

1. 访问对象属性

```
p = Person.new
p.age
```

输出:

```
name: age
arguments:[]
```

我们发现上面的 `p.age` 实际上是执行了 `method_missing('age')`


2. 设置对象属性

```
p = Person.new
p.age = 23
```

输出

```
name: age=
arguments:[23]
```

我们发现 `p.age = 23` 实际上等同于 `method_missing('age', 23)`


3. 调用一个不存在的方法

```
p = Person.new
puts p.test
```

输出

```
name: test
arguments:[]
this is a test
```


