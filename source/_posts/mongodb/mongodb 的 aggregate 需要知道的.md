---
title: mongodb 的 aggregate 需要知道的
date: 2020-02-27 19:20:00
tags: [MongoDB]
---

今天在排查 stage 环境的 mongodb log 的时候发现有很多慢查询，本来想尝试去优化一波，但无奈的是实在是看不懂那些 aggregate 语句，目前只好先来补习一下 mongodb 的 aggregate 相关知识。

## mongodb aggregate 做了什么？

**我们在使用 mongodb 的 aggregate 操作的时候会传递一个数组参数，这个数组里面的每一项定义了这一步需要做的操作，然后把这一步的结果传递给数组下一个聚合操作，知道执行完所有操作。**

> MongoDB 的聚合框架以数据处理管道的概念为模型。文档进入多阶段流水线，该流水线将文档转换成汇总结果。


## 例子（来自官网）

假设我们有如下数据：

orders:

```
{
    cust_id: "A123",
    amount: 500,
    status: "A"
},
{
    cust_id: "A123",
    amount: 250,
    status: "A"
},
{
    cust_id: "B212",
    amount: 200,
    status: "A"
},
{
    cust_id: "A123",
    amount: 300,
    status: "D"
},
```

我们对这个 collection 执行如下操作：

```
db.orders.aggregate([
    { $match: { status: "A" } },
    { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

## 这个操作做了什么？

### 首先，根据传递的条件筛选出符合条件的数据

```
 { $match: { status: "A" } },
```

首先，筛选出 `status` 为 `"A"` 的记录。这个过程会得到下面的记录：

```
{
    cust_id: "A123",
    amount: 500,
    status: "A"
},
{
    cust_id: "A123",
    amount: 250,
    status: "A"
},
{
    cust_id: "B212",
    amount: 200,
    status: "A"
}
```


### 然后，根据 cust_id 分组，并且统计 amount 的总数

```
{ $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
```

`$group` 表明我们要对上一步的结果进行分组，`_id` 表明我们需要根据 `"$cust_id""` 来分组，相同 `cust_id` 的数组分为一组，最后，`$sum` 表明我们需要根据后面的表达式来当前分组执行求和操作，求和的字段为 `amount`。

这个过程会得到下面的结果：

```
{
    _id: "A123",
    total: 750
},
{
    _id: "B212",
    total: 200
}
```

