---
title: Go MongoDB Driver 中的 A D M E 类型是什么
date: 2020-03-10 07:33:00
tags: [MongoDB]
---

先看看源码

```
// D is an ordered representation of a BSON document. This type should be used when the order of the elements matters,
// such as MongoDB command documents. If the order of the elements does not matter, an M should be used instead.
//
// Example usage:
//
// 		bson.D{{"foo", "bar"}, {"hello", "world"}, {"pi", 3.14159}}
type D = primitive.D

// E represents a BSON element for a D. It is usually used inside a D.
type E = primitive.E

// M is an unordered representation of a BSON document. This type should be used when the order of the elements does not
// matter. This type is handled as a regular map[string]interface{} when encoding and decoding. Elements will be
// serialized in an undefined, random order. If the order of the elements matters, a D should be used instead.
//
// Example usage:
//
// 		bson.M{"foo": "bar", "hello": "world", "pi": 3.14159}
type M = primitive.M

// An A is an ordered representation of a BSON array.
//
// Example usage:
//
// 		bson.A{"bar", "world", 3.14159, bson.D{{"qux", 12345}}}
type A = primitive.A
```

在 Go MongoDB Driver 中，我们调用 MongoDB Driver 提供的方法的时候，往往需要传入 `bson.D`, `bson.A`, `bson.M`, `bson.A` 这几种类型。

## bson.D

用来表示一个有序的 BSON 文档，当我们需要传递一些有序的参数的时候可以使用，比如 MongoDB 中的聚合操作，聚合操作往往是一组操作的集合，比如先筛选、然后分组、然后求和，这个顺序是不能乱的。


## bson.E

用来表示 bson.D 中的一个属性，类型定义如下:

```
// E represents a BSON element for a D. It is usually used inside a D.
type E struct {
	Key   string
	Value interface{}
}
```

可以类比 json 对象里面的某一个属性以及其值。


### 为什么不是 bson.M 中的属性呢？

首先，bson.M 其实是一个 map，不接受这种类型，bson.D 实际上是 []bson.E 类型，是 bson.E 的切片。

其次，bson.M 里面是顺序无关的，普通的 map 就已经足够了，没必要再定义一个类型。


##  bson.M

用来表示无需的 BSON 文档，就是一个普通的 map 类型。在保存到 MongoDB 中的时候，字段顺序是不确定的，而 bson.D 的顺序是确定的。

顺序可能大部分情况都是不需要的。不过在匹配嵌套的 BSON 数组文档的时候，可能会有问题。但还是有其他的解决办法的。

可参考本站的: MongoDB $elemMatch 操作符


## bson.A

用来表示 BSON 文档中的数组类型，是有序的。



