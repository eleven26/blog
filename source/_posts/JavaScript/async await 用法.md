---
title: async await 用法
date: 2019-06-01 18:28:00
tags: [JavaScript]
---

`Async/Await` 是 `Promise` 的语法糖，有了 `async/await` 我们可以以一种更简便的方式来实现 `Promise` 的功能

### Async

`async` 函数， `async` 被放置在一个函数前面，就像下面这样：

```javascript
async function f() {
  return 1
}
```

函数前面的 `async` 一词意味着一个简单的事情：这个函数总是返回一个 `promise`，如果代码中有 `return <非promise>` 语句，
JavaScript 会自动把返回的这个 `value` 值包装成 `promise` 的 `resolved` 值。

例如，上面的代码返回 `resolved` 值为 `1` 的 `promise`：

```javascript
async function f() {
  return 1
}
f().then(alert) // 1
```

我们也可以显式的返回一个 `promise`，这个将会是同样的结果：
```javascript
async function f() {
  return Promise.resolve(1)
}
f().then(alert) // 1
```

所以，`async` 确保了函数返回一个 `promise`，即使其中包含非 `promise`。

还有另外一个关键词 `await`，只能在 `async` 函数里使用。


#### Await

语法如下：

```javascript
// 只能在 async 函数内部使用
let value = await promise
```

关键词 `await` 可以让 JavaScript 进行等待，直到一个 `promise` 执行并返回它的结果，JavaScript 才会继续往下执行。

例子：

```javascript
async function f() {
  let promise = new Promise(resolve => {
      setTimeout(() => resolve(1), 1000)
  })
  let result = await promise // 等待，直到 promise 返回一个 resolve 值
  alert(result) // 1
}
f()
```

#### 注意事项
* 不能在常规函数里使用 `await`
* `await` 不能工作在顶级作用域


#### 总结
* 放在一个函数前面的 `async` 有两个作用
    - 使函数总是返回一个 `promise`
    - 允许在这其中使用 `await`
    
* `promise` 前面的 `await` 关键字能够使 JavaScript 等待，直到 `promise` 处理结束。然后：
    - 如果它是一个错误，异常就产生了，就像在哪个地方调用了 `throw error` 一样
    - 否则，它会返回一个结果，我们可以将它分配给一个值

* `async/await` 一起提供了一个很好的框架来编写易于读写的异步代码，有了 `async/await`，我们很少需要写 `promise.then/catch`，
但是我们仍然不应该忘记它们是基于 `promise` 的。
