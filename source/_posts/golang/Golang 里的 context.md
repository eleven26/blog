---
title: Golang 里的 context
date: 2020-03-27 16:10:30
tags: [Go]
---

## context 的作用

go 的编程中，常常会在一个 goroutine 中启动多个 goroutine，然后有可能在这些 goroutine 中又启动多个 goroutine。

![goroutine-0](/images/go/goroutine-0.png)

如上图，在 main 函数中，启动了一个 goroutine A 和 goroutine B，然后 goroutine A 中又启动了 goroutine A1 和 goroutine A2，goroutine B 中也是。

有时候，我们可能想要取消当前的处理，这个时候自然而然的也要取消子协程的执行进程。这个时候就需要一种机制来做这件事。context 就是设计来做这件事的。

比如在 web 应用中，当子协程的某些处理时间过长的时候，我们可能想要终止下游的处理，防止协程长期占用资源。保证其他客户端的请求正常处理。


## context 的不同类型

### context.Background

往往用做父 context，比如在 main 中定义的 context，然后在 main 中将这个 context 传递给子协程。

### context.TODO

不需要使用什么类型的 context 的时候，使用这个 context.TODO

### context.WithTimeout


### context.WithDeadline


### context.WithCancel


