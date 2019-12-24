---
title: Go 等待协程完成
date: 2019-12-24 16:18:30
tags: [Go]
---

## 使用 sync.WaitGroup

为了等待 goroutine 结束，我们可以使用 [sync.WaitGroup](https://golang.org/pkg/sync/#WaitGroup) 来实现等待

```
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, wg *sync.WaitGroup) {
	fmt.Printf("Worker %d starting\n", id)

	time.Sleep(time.Second)

	fmt.Printf("Worker %d done\n", id)

	wg.Done()  // 协程完成，等待的协程数 - 1，减到 0 的时候就继续执行 wg.Wait() 后面的代码
}

func main()  {
	var wg sync.WaitGroup

	for i := 1; i <= 5; i++ {
		wg.Add(1) // 等待的协程数 + 1
		go worker(i, &wg)
	}

	wg.Wait() // 等待所有协程完成

	fmt.Println("All done!")
}

```


## 使用 channel

```
package main

import (
	"fmt"
	"time"
)

func worker(id int, ch chan int) {
	fmt.Printf("Worker %d starting\n", id)

	time.Sleep(time.Second)

	ch <- id
}

func main() {
	ch := make(chan int)

	for i := 1; i <= 5; i++ {
		go worker(i, ch)
	}

	for i := 1; i <= 5; i++ {
		fmt.Printf("Worker %d done\n", <-ch)
	}

	fmt.Println("All done!")
}

```

chanel 的特性是从 channel 中获取数据的时候会引起阻塞，直到 channel 有数据，所以我们可以利用这个特性在 goroutine 的最后往 channel 里面放东西，
然后主协程里面从 channel 里面获取东西，只需要次数一致就可以了。

> 这种方式也是官方推荐的同步方式，sync 通常用于比较底层的同步
