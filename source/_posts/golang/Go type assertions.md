---
title: Go type assertions
date: 2019-12-27 19:46:30
tags: [Go]
---

格式：`x.(T)`


含义：断言 `x` 不是 `nil` 并且存储的是 `T` 类型的值


用途：

1. 检查 `x` 是否为 `nil`

2. 检查 `x` 能否转换为类型 `T`

3. 转换 `x` 为类型 `T`


返回值：

`t := x.(T)`，返回一个类型为 `T` 的值，如果 `x` 为 `nil`，产生 panic

`t, ok := x.(T)` ，如果 `x` 为 `nil` 或者不是 `T` 类型，`ok` 的值为 `false`，否则 `ok` 的值为 `true` 并且 `t` 是一个类型为 `T` 的值。


使用方式：

```
v, ok = x.(T)
v, ok := x.(T)
var v, ok = x.(T)
var v, ok T1 = x.(T)
```


### 实际用途

1. 判断 `interface{}` 的类型

2. 转换 `interface{}` 为具体的类型

```
package main

import "fmt"

type Student struct {
	Name string
}

func test(i interface{}) interface{} {
	return i
}

func main() {
	var i interface{}
	student := Student{Name: "golang"}
	i = test(student)
	//fmt.Println(i.Name)  // i.Name undefined (type interface {} is interface with no methods)
	fmt.Println(i.(Student).Name) // 我们知道具体类型

	// 如果不知道具体类型，可以按照下面的方式判断
    // ok 为 true 说明 i 是 Student 类型，否则不是 Student 类型
	if j, ok := i.(Student); ok {
		fmt.Println(j.Name)
	}

    // 下面的 ok 为 false
    var ii interface{}
    ii = test(ii)
    if v, ok := ii.(Student); ok {
        fmt.Println(v.Name)
    } else {
        fmt.Println("ii is not Student")
    }
}
```

输出

```
golang
golang
ii is not Student
```

