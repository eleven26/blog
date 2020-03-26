---
title: MongoDB $elemMatch 操作符.md
date: 2020-03-10 07:05:00
tags: [MongoDB]
---

`$elemMatch` 是 MongoDB 中用来查询内嵌文档的操作符。

## 创建一个简单文档

```
db.test.insert({
    "id": 1,
    "members": [
        {
            "name": "Jack",
            "age": 27,
            "gender": "M"
        },
        {
            "name": "Tony",
            "age": 23,
            "gender": "F"
        },
        {
            "name": "Lucy",
            "age": 21,
            "gender": "M"
        }
    ]
})
```


## 使用多种方式尝试查询

a. 使用 `db.test.find({"members": {"name": "Jack"}})` 进行查询:

```
db.test.find({"members": {"name": "Jack"}})
```

查询的结果是空集。（因为 members 没有 name 字段）

b. 只有完全匹配一个的时候才能获取到结果，因此：

```
db.test.find({"members": {"name": "Jack", "age": 27, "gender": "M"}})
```

可以得到结果

c. 如果把键值进行颠倒，也得不到结果:

```
db.test.find({"members": {"name": "Jack", "gender": "M", "age": 27}})
```

得到的结果是空集。

d. 我们这样查询:

```
db.test.find({"members.name": "Jack"})
```

是可以查询出结果的。

e. 如果需要两个属性

```
db.test.find({"members.name": "Jack", "members.age": 27})
```

也可以查询结果。

f. 我们再进行破坏性测试

```
db.test.find({"members.name": "Jack", "members.age": 23})
```

也可以查询出结果。

不过我们应该注意到：Jack 是数组中第一个元素的键值，而 23 是数组中第二个元素的键值，这样也可以查询出结果。


## 使用 $elemMatch 操作符查询

对于我们的一些应用来说，以上结果显然不是我们想要的结果。所以我们应该使用 $elemMatch 操作符:

a. $elemMatch + 同一个元素中的键值组合

```
db.test.find({"members": {"$elemMatch": {"name": "Jack", "age": 27}}})
```

可以查询出结果。

b. $elemMatch + 不同元素的键值组合

```
db.test.find({"members": {"$elemMatch": {"name": "Jack", "age": 23}}})
```

查询不出结果。

因此，a 展示的嵌套查询正是我们想要的查询方式。
