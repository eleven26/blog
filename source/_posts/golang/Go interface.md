---
title: Go interface
date: 2019-12-28 19:52:30
tags: [Go]
---

接口类型是对其它类型行为的概括与抽象。通过使用接口，我们可以写出更加灵活和通用的函数，这些函数不用绑定在一个特定的类型实现上。

很多面向对象的语言都有接口这个概念，Go 语言的接口的独特之处在于它是隐式实现。换句话说，对于一个具体的类型，无须声明它实现了哪些接口，只要
提供接口所需的方法即可。这种设计让你无须改变已有类型的实现，就可以为这些类型创建新的接口，对于那些不能修改包的类型，这一点特别有用。


## 接口即约定

具体类型，如 int、float32 等，指定了它们所含数据的精确布局，还暴露了基于这个精确布局的内部操作。比如对于数值类型有算术操作，对于 slice
类型我们有索引、append、range 操作等。具体类型还会通过其方法来提供额外的能力。总之，如果你知道了一个具体类型的数据，那么你就
精确地知道了它是什么以及它能干什么。

Go 语言中还有另外一种类型成为称为 接口类型。接口是一种抽象类型，它并没有暴露所含数据的布局或者内部结构，当然也没有那些数据的基本操作，
它所提供的仅仅是一些方法而已。如果你拿到一个接口类型的值，你无从知道它是什么，你能知道的仅仅是它能做什么，
或者更精确地讲，仅仅是它提供了哪些方法。



## 接口类型

一个接口类型定义了一套方法，如果一个具体类型要实现该接口，那么必须实现接口类型定义中的所有方法。

`io.Writer` 是一个广泛使用的接口，它负责所有可以写入字节类型的抽象，包括文件、内存缓冲区、网络链接、HTTP 客户端等。
io 包还定义了很多有用的接口。Reader 就抽象了所有可以读取字节的类型，Closer 抽象了所有可以关闭的类型，比如文件或者网络连接。

```
package go

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```

另外，我们还可以发现通过组合已有接口得到新的接口，如：

```
type ReaderWriter interface {
    Reader
    Writer
}

type ReaderWriteCloser interface {
    Reader
    Writer
    Closer
}
```

如上语法称为嵌入式接口，与嵌入式结构类似，让我们可以直接使用一个接口，而不用逐一写出这个接口所包含的方法。

如下所示，尽管不够简洁，但是可以不用嵌入式来声明 `io.ReadWriter`：

```
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}

// 也可以混合使用两种方式：
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Writer
}
```

三种声明的效果都是一样的。方法定义的顺序也是无意义的，真正有意义的只有接口的方法集合。



## 实现接口


如果一个类型实现了一个接口要求的所有方法，那么这个类型实现了这个接口。比如 `*os.File` 类型实现了 `io.Reader`、`io.Writer`、`Closer` 和 `ReaderWriter` 接口。
`*bytes.Buffer` 实现了 `Reader`、`Writer` 和 `ReaderWriter`，但是没有实现 `Closer`，因为它没有 `Close` 方法。
为了简化表述，Go 程序员通常说一个具体类型 "是一个"（is-a）特定的接口类型，这其实代表着该具体类型实现了该接口。
比如，`*bytes.Buffer` 是一个 `io.Writer`；`*os.File` 是一个 `io.ReaderWriter`。

接口的赋值很简单，仅当一个表达式实现了一个接口时，这个表达式才可以赋给该接口。所以：

```
var w io.Writer
w = os.Stdout  // OK: *os.File 有 Write 方法
w = new(bytes.Buffer) // OK: *bytes.Buffer 有 Write 方法
w = time.Second  // 编译错误: time.Duration 缺少 Write 方法

var rwc io.ReadWriteCloser
rwc = os.Stdout // OK: *os.File 有 Read、Write、Close 方法
rwc = new(bytes.Buffer) // 编译错误: *bytes.Buffer 缺少 Close 方法
```

当右侧表达式也是一个接口时，该规则也有效：

```
w = rwc // OK: io.ReadWriteCloser 有 Write 方法
rwc = w // 编译错误: io.Writer 缺少 Close 方法
```

因为 `ReadWriter` 和 `ReadWriterCloser` 接口包含了 `Writer` 的所有方法，所以任何实现了 `ReadWriter` 或 `ReadWriterCloser` 类型
的方法都必然实现了 `Writer`。


在进一步讨论之前，我们先解释一下一个类型有某一个方法的具体含义。对每一个具体类型 T，部分方法的接收者就是 T，而其它方法的接收者则是 *T 指针。
同时我们对类型 T 的变量直接调用 *T 的方法也可以是合法的，只要改变量是可变的，编译器隐式地帮你完成了取地址的操作。但这仅仅是一个语法糖，
类型 T 的方法没有对应的指针 *T 多，所以实现的接口也可能比对应的指针少。

比如，下面的 IntSet 类型的 String 方法，需要一个指针接收者，所以我们无法从一个无地址的 IntSet 值上调用该方法：

```
type IntSet struct {}
func (*IntSet) String() string {}

var _ = IntSet{}.String() // 编译错误: String 方法需要 *IntSet 接收者
```

但可以从一个 IntSet 变量上调用该方法：

```
var s IntSet
var _ = s.String() // OK: s 是一个变量，&s 有 String 方法
```

因为只有 *IntSet 有 String 方法，所以也只有实现了 fmt.Stringer 接口：

```
var _ fmt.Stringer = &s // OK
var _ fmt.Stringer = s // 编译错误: IntSet 缺少 String 方法
```

正如信封封装了信件，接口也封装了所对应的类型和数据，只有通过接口暴露的方法才可以调用，类型的其它方法则无法通过接口来调用:

```
os.Stdout.Write([]byte("hello")) // OK: *os.File 有 Write 方法
os.Stdout.Close() // OK: *os.File 有 Close 方法

var w io.Writer
w = os.Stdout
w.Write([]byte("hello")) // OK: io.Writer 有 Write 方法
w.Close() // 编译错误: io.Writer 缺少 Close 方法
```

一个拥有更多方法的接口，比如 `io.ReadWriter`，与 `io.Reader` 相比，给了我们它所指向数据的更多信息，当然也给实现这个接口提出更高的门槛。
那么对于接口类型 `interface{}`，它完全不包含任何方法，通过这个接口能得到对应具体类型的什么信息呢？

确实什么信息也得不到。看起来这个接口没有任何用途，但实际上称为空接口类型的 `interface{}` 是不可缺少的。正因为空接口类型没有对其实现类型
有任何要求，所以我们可以把任何值赋给空接口类型。

```
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```

当然，即使我们创建了一个指向布尔值、浮点数、字符串或者其它类型的 `interface{}` 接口，也无法直接使用其中的值，毕竟这个接口不包含任何方法。
我们需要一个方法从空接口中还原出实际值，如类型断言。

判断是否实现接口只需要比较具体类型和接口类型的方法，所以没有必要在具体类型的定义中声明这种关系。也就是说，偶尔在注释中标注也不坏，但对于程序来讲，
这种关系声明不是必需的。如下声明在编译器就断言了 `*bytes.Buffer` 类型的一个值必然实现了 `io.Writer`：

```
// *bytes.Buffer 必须实现 io.Writer
var w io.Writer = new(bytes.Buffer)
```

我们甚至不需要创建一个新的变量，因为 `*bytes.Buffer` 的任意值都实现了这个接口，甚至 nil，在我们用 `*bytes.Buffer(nil)` 来强制类型转换后，
也实现了这个接口。当然，既然我们不想引用 w，那么我们可以把它替换为空白标识符。

```
// *bytes.Buffer 必须实现 io.Writer
var _ io.Writer = (*bytes.Buffer)(nil)
```

非空的接口类型（比如 io.Writer）通常由一个指针类型来实现，特别是当接口类型的一个或多个方法暗示会修改接收者的情形（比如 Write 方法）。
一个指向结构的指针才是最常见的方法接收者。

指针类型肯定不是实现接口的唯一类型，即使是那些包含了会改变接收者方法的接口类型，也可以由 Go 的其它引用类型来实现。

一个具体类型可以实现很多不相关的接口。比如一个程序管理或者销售数字文化商品，比如音乐、电影和图书。那么它可能定义了如下具体类型：

```
Album
Book
Movie
Magazine
Podcast
TVEpisode
Track
```

我们可以把感兴趣的每一种抽象都用一种接口类型来表示。一些属性是所有商品都具备的，比如标题、创建日期以及创建者列表（作者或者艺术家）。

```
type Artifact interface {
    Title() String
    Creators() []string
    Created() time.Time
}
```

其它属性则局限于特定类型的商品。比如字数这个属性只与书和杂志相关，而屏幕分辨率则只与电影和电视剧相关。

```
type Text interface {
    Pages() int
    Words() int
    PageSize() int
}

type Audio interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string // 比如 "MP3"、"WAV"
}

type Video interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string // 比如 "MP4"、"WMV"
    Resolution() (x, y int)
}
```

这些接口只是一种把具体类型分组并暴露它们共性的方式，未来我们也可以发现其它的分组方式。比如，如果我们要把 Audio 和 Video 按照同样的方式来处理，
就可以定义一个 Streamer 接口来呈现它们的共性，而不用修改现有的类型定义。

```
type Streamer interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string
}
```

从具体类型出发、提取其共性而得出的每一种分组方式都可以表示为一种接口类型。
与基于类的语言（它们显式地声明了一个类型实现所有的接口）不同的是，在 Go 语言里我们可以在需要时才定义新的抽象和分组，并且不用修改原有类型的定义。
当需要使用另一个作者写的包里的具体类型时，这一点特别有用。当然，还需要这些具体类型在底层是真正有共性的。


## 使用 flag.Value 来解析参数


