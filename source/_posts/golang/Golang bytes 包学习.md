---
title: Golang bytes 包学习
date: 2020-03-29 13:01:30
tags: [Go]
---

1. [常量](/常量)
2. [变量](/变量)
3. [函数](/函数)
    * [func Compare(a, b []byte) int](/Compare)
4. Buffer
5. Reader


## 常量

`MinRead` 是传递给 `Buffer.ReadFrom` 方法的最小切片大小。只要 `Buffer` 里面的内容长度没有超出 `MinRead` 个字节，`ReadFrom` 不会增长 `Buffer` 里面的缓冲区大小。

```
const MinRead = 512
```


## 变量

当内存装不下需要放入 buffer 的数据的时候，会抛出一个 panic，内容就是 `ErrTooLarge`

```
var ErrTooLarge = errors.New("bytes.Buffer: too large")
```

## 函数

### Compare

```
func Compare(a, b []byte) int
```

按字典顺序比较两个字节切片。如果 a == b，返回 0，如果 a < b 返回 -1，如果 a > b 返回 1。


<details>
    <summary>实例</summary>
    
    ```
    package main
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func main() {
    	// 通过将结果与零进行比较来解释比较结果
    	var a, b []byte
    
    	a = []byte("a")
    	b = []byte("b")
    	if bytes.Compare(a, b) < 0 {
    		// a 小于 b
    		fmt.Printf("a = '%s', b = '%s', a < b\n", a, b)
    	}
    
    	a = []byte("a")
    	b = []byte("a")
    	if bytes.Compare(a, b) <= 0 {
    		// a 小于或等于 b
    		fmt.Printf("a = '%s', b = '%s', a <= b\n", a, b)
    	}
    
    	a = []byte("b")
    	b = []byte("a")
    	if bytes.Compare(a, b) > 0 {
    		// a 大于 b
    		fmt.Printf("a = '%s', b = '%s', a > b\n", a, b)
    	}
    
    	a = []byte("b")
    	b = []byte("b")
    	if bytes.Compare(a, b) >= 0 {
    		// a 大于或等于 b
    		fmt.Printf("a = '%s', b = '%s', a >= b\n", a, b)
    	}
    
    	// 判断两个字节切片是否相等
    	if bytes.Equal(a, b) {
    		// a 等于 b
    		fmt.Printf("a = '%s', b = '%s', a == b\n", a, b)
    	}
    	b = []byte("c")
    	if !bytes.Equal(a, b) {
    		// a 不等于 b
    		fmt.Printf("a = '%s', b = '%s', a != b\n", a, b)
    	}
    }
    // Output:
    // a = 'a', b = 'b', a < b
    // a = 'a', b = 'a', a <= b
    // a = 'b', b = 'a', a > b
    // a = 'b', b = 'b', a >= b
    // a = 'b', b = 'b', a == b
    // a = 'b', b = 'c', a != b
    ```
</details>
