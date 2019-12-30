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

在本节中，我们将看到如何使用另外一个标准接口 `flag.Value` 来帮助我们定义命令行标志。考虑如下一个程序，它实现了睡眠指定时间的功能。

```
var period = flag.Duration("period", 1 * time.Second, "sleep period")

func main() {
    flag.Parse()
    fmt.Printf("Sleeping for %v...", *period)
    time.Sleep(*period)
    fmt.Println()
}
```

在程序进入睡眠前输出了睡眠时长。fmt 包调用了 `time.Duration` 的 `String` 方法，可以按照一个用户友好的方式来输出，
而不是输出一个以纳秒为单位的数字。

默认的睡眠时间是 1s，但是可以用 `-period` 命令行标志来控制。`flag.Duration` 函数创建了一个 `time.Duration` 类型的标志变量，
并且允许用户用一种友好的方式来指定时长，比如可以用 String 方法对应的记录方法。这种对称的设计提供了一个良好的用户接口。

```
$ ./sleep -period 50ms
```

因为时间长度类的命令行标志广泛应用，所以这个功能内置到了 flag 包。支持自定义类型其实也不难，只须定义一个满足 flag.Value 接口的类型，其定义如下所示：

```
package flag

// Value 接口代表了存储在标志内的值
type Value interface {
    String() string
    Set(string) error
}
```

String 方法用于格式化标志对应的值，可用于输出命令行帮助消息。由于有了该方法，因此每个 flag.Value 其实也是 fmt.Stringer。
Set 方法解析了传入的字符串参数并更新标志值。可以认为 Set 方法是 String 方法的逆操作，两个方法使用同样的记法规格是一个很好的实践。


## 接口值

从概念上来讲，一个接口类型的值（简称接口值）其实有两个部分：一个具体类型和该类型的一个值。二者称为接口的动态类型和动态值。

对于像 Go 这样的静态类型语言，类型仅仅是一个编译时的概念，所以类型不是一个值。在我们的概念模型中，用类型描述符来提供每个类型的具体信息，
比如它的名字和方法。对于一个接口值，类型部分就用对应的类型描述符来表述。


如下四个语句中，变量 w 有三个不同的值（最初和最后是同一个值）：

```
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

接下来我们详细地查看一下在每个语句之后 w 的值和相关的动态行为。第一个语句声明 w：

```
var w io.Writer
```

在 Go 语言中，变量总是初始化为一个特定的值，接口也不例外。接口的零值就是把它的动态类型和值都设置为 nil。

一个接口值是否是 nil 取决于它的动态类型，所以现在这是一个 nil 接口值。可以用 w == nil 或者 w != nil 来检测一个接口值是否是 nil。

调用一个 nil 接口的任何方法都会导致崩溃：

```
w.Write([]byte("hello")) // 崩溃：对空指针取引用值
```

第二个语句把一个 *os.File 类型的值赋给了 w：

```
w = os.Stdout
```

这次赋值把一个具体类型隐式转换为一个接口类型，它与对应的显式转换 `io.Writer(os.Stdout)` 等价。不管这种类型的转换是隐式的还是显式的，
它都可以转换操作数的类型和值。接口值的动态类型会设置为指针类型 *os.File 的类型描述符，它的动态值会设置为 `os.Stdout` 的副本，即一个
指向代表进程的标准输出的 os.File 类型的指针。

![7-1](/images/go/7-1.png)

调用该接口值的 Write 方法，会实际调用 (*os.File).Write 方法，即输出 "hello"

```
w.Write([]byte("hello")) // "hello"
```

一般来讲，在编译时我们无法知道一个接口值的动态类型会是什么，所以通过接口来做调用必然需要使用动态分发。编译器必须生成一段代码来从类型描述符
拿到名为 Writer 的方法地址，再间接调用该方法地址。调用的接收者就是接口值的动态值，即 `os.Stdout`，所以实际效果与直接调用等价：

```
os.Stdout.Writer([]byte("hello")) // "hello"
```

第三个语句把一个 *bytes.Buffer 类型的值赋给了接口值：

```
w = new(bytes.Buffer0
```

动态类型现在是 `*bytes.Buffer`，动态值现在则是一个指向新分配缓冲区的指针：

![7-3](/images/go/7-3.png)

调用 Write 方法的机制也跟第二个语句一致：

```
w.Write([]byte("hello")) // 把 "hello" 写入 bytes.Buffer
```

这次，类型描述符是 *bytes.Buffer，所以调用的是 (*bytes.Buffer).Write 方法，方法的接收者是缓冲区的地址。调用该方法会追加 "hello" 到缓冲区。

最后，第四个语句把 nil 赋给了接口值：

```
w = nil
```

这个语句把动态类型和动态值都设置为 nil，把 w 恢复到了它刚声明时的状态。

一个接口值可以指向多个任意大的动态值。比如，`time.Time` 类型可以表明一个时刻，它是一个包含几个非导出字段的结构。如果从它创建一个接口值：

```
var x interface{} = time.Now()
```

结果是，x 的类型值是一个 `time.Time` 结构的值。

接口值可以用 == 和 != 操作符来做比较。如果两个接口值都是 nil 或者二者的动态类型完全一致且二者动态值相等（使用动态类型的 == 操作符来做比较），
那么两个接口值相等。因为接口值是可以比较的，所以它们可以作为 map 的键，也可以作为 switch 语句的操作数。

需要注意的是，在比较两个接口值的时，如果两个接口值的动态类型一致，但对应的动态值是不可比较的（比如 slice），
那么这个比较会以崩溃的方式失败：

```
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // 宕机：试图比较不可比较类型 []int
```

其它类型要么是可以安全比较的（比如基础类型和指针），要么是完全不可比较的，但当比较接口值或者其中包含接口值的聚合类型时，
我们必须消息崩溃的可能性。当把接口作为 map 的键或者 switch 语句的操作数时，也存在类似的风险。
仅在能确认接口值包含的动态值可以比较时，才比较接口值。

当处理错误或者调试时，能拿到接口值的动态类型是很有帮助的。可以使用 fmt 包的 %T 来实现这个需求。

```
var w io.Writer
fmt.Println("%T\n", w) // "<nil>"

w = os.Stdout
fmt.Println("%T\n", w) // "*os.File"

w = new(bytes.Buffer)
fmt.Println("%T\n", w) // "*bytes.Buffer"
```

在内部实现中，fmt 用反射来拿到接口动态类型的名字。


### 注意：含有空指针的非空接口

空的接口值（其中不包含任何信息）与仅仅动态值为 nil 的接口值是不一样的。

考虑如下程序，当 debug 设置为 true 时，主函数收集函数 f 的输出到一个缓冲区中：

```
const debug = true

func main() {
    var buf *bytes.Buffer
    if debug {
        buf = new(bytes.Buffer) // 启用输出收集
    }
    f(buf) // 注意：微妙的错误
    if debug {
        // 使用 buf
    }
}

// 如果 out 不是 nil，那么会向其写入输出的数据
func f(out io.Writer) {
    // ... 其它代码 ...
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```

当设置 debug 为 false 时，我们会觉得仅仅是不再收集输出，但实际上会导致程序在调用 `out.Write` 时崩溃：

```
if out != nil {
    out.Write([]byte("done!\n")) // 宕机：对空指针取引用值
}
```

当 main 函数调用 f 时，它把一个类型为 *bytes.Buffer 的空指针赋给了 out 参数，所以 out 是一个包含空指针的非空接口，
所以防御性检查 out != nil 仍然是 true。

如前所述，动态分发机制决定了我们肯定会调用 (*bytes.Buffer).Write，只不过这次接收者值为空。
对于某些类型，比如 *os.File，空接收值是合法的，但对于 *bytes.Buffer 则不行。方法尽管被调用了，但在尝试访问缓冲区时崩溃了。

问题在于，尽管一个空的 `*bytes.Buffer` 指针拥有的方法满足了该接口，但它并不满足该接口所需的一些行为。
特别是，这个调用违背了 `(*bytes.Buffer).Write` 的一个隐式的前置条件，即接收者不能为空，所以把空指针赋给这个接口就是一个错误。
解决方案是把 main 函数中的 buf 类型修改为 `io.Writer`，从而避免在最开始就把一个功能不完整的值赋给一个接口。

```
var buf io.Writer
if debug {
    buf = new(bytes.Buffer) // 启用输出收集
}
f(buf) // OK
```


## error 接口

error 只是一个接口类型，包含一个返回错误消息的方法：

```
type error interface {
    Error() string
}
```

构造 error 最简单的方法是调用 `errors.New`，它会返回一个包含特定的错误消息的新 error 实例。完整的 error 包只有如下 4 行代码：

```
package errors

func New(text string) error {
    return &errorString{text}
}

func (e *errorString) Error() string {
    return e.text
}
```

底层的 errorString 类型是一个结构，而没有直接用字符串，主要是为了避免将来无意间的布局变更。满足 error 接口的是 *errorString 指针，
而不是原始的 errorString，主要是为了让每次 New 分配的 error 实例都互不相等。我们不希望出现像 io.EOF 这样重要的错误，与仅仅
包含同样错误消息的一个错误相等。

```
fmt.Println(errors.New("EOF") == errors.New("EOF")) // false
```

直接调用 `errors.New` 比较罕见，因为有一个更易用的封装函数 `fmt.Errorf`，它还额外提供了字符串格式化功能。

```
package fmt

import "errors"

func Errorf(format string, args ...interface{}) error {
    return errors.New(Sprintf(format, args...))
}
```

尽管 `*errorString` 可能是最简单的 error 类型，但这样简单的 error 类型远不止一个。


## 类型断言

类型断言是一个作用在接口值上的操作，写出来类似于 `x.(T)`，其中 x 是一个接口类型的表达式，而 T 是一个类型（称为断言类型）。
类型断言会检查作为操作数的动态类型是否满足指定的断言类型。

这儿有两个可能。首先，如果断言类型 T 是一个具体类型，那么类型断言会检查 x 的动态类型是否就是 T。如果检查成功，类型断言的结果就是 x 的动态值，
类型当然是 T。换句话说，类型断言就是用来从它的操作数中把具体类型的值提取出来的操作。如果检查失败，那么操作崩溃。比如：

```
var w io.Writer
w = os.Stdout
f := w.(*os.File) // 成功：f == os.Stdout
c := w.(*bytes.Buffer) // 崩溃：接口持有的是 *os.File，不是 *bytes.Buffer
```

其次，如果断言类型 T 是一个接口类型，那么类型断言检查 x 的动态类型是否满足 T。如果检查成功，动态值并没有提取出来，结果仍然是一个接口值，
接口值的类型和值部分也没有变更，只是结果的类型为接口类型 T。换句话说，类型断言是一个接口值表达式，从一个接口类型变为拥有另外一套方法的接口类型
（通常方法数量是增多），但保留了接口值中的动态类型和动态值部分。

如下类型断言代码中，w 和 rw 都持有 os.Stdout，于是所有对应的动态类型都是 *os.File，但 w 作为 `io.Writer` 仅暴露了文件的 Write 方法，
而 rw 还暴露了它的 Read 方法。

```
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter) // 成功：*os.File 有 Read 和 Write 方法

w = new(ByteCounter)
rw = w.(io.ReadWriter) // 崩溃：*ByteCounter 没有 Read 方法
```

无论哪种类型作为断言类型，如果操作数是一个空接口值，类型断言都失败。很少需要从一个接口类型向一个要求更宽松的类型做类型断言，
该宽松类型的接口方法比原类型的少，而且是子集。因为除了在操作 nil 之外的情况下，在其它情况下这种操作与赋值一致。

```
w = rw // io.ReadWriter 可以赋给 io.Writer
w = rw.(io.Writer) // 仅当 rw == nil 时失败
```

我们经常无法确定一个接口值的动态类型，这时就需要检测它是否是某一个特定类型。如果类型断言出现在需要两个结果的赋值表达式中，
那么断言不会在失败时崩溃，而是会多返回一个布尔型的返回值来指示断言是否成功。

```
var w io.Writer = os.Stdout
f, ok := w.(*os.File) // 成功: ok，f == os.Stdout
b, ok := w.(*bytes.Buffer) // 失败: !ok, b == nil
```

按照惯例，一般把第二个返回值赋给一个名为 ok 的变量。如果操作失败，ok 为 false，而第一个返回值为断言类型的零值，
在这个例子中就是 *bytes.Buffer 的空指针。

ok 返回值通常马上就用来决定下一步做什么。下面 if 表达式的扩展形式就可以让我们写出相当紧凑的代码:

```
if f, ok := w.(*os.File); ok {
    // ...use w...
}
```


## 使用类型断言来识别错误

考虑一下 os 包中的文件操作返回的错误集合，I/O 会因为很多原因失败，但有三类原因通常必须单独处理：
文件已存储（创建操作），文件没找到（读取操作）以及权限不足。os 包提供了三个帮助函数来对错误进行分类：

```
package os

func IsExist(err error) bool
func IsNotExist(err error) bool
func IsPermission(err error) bool
```

一个幼稚的实现会通过检查错误消息是否包含特定的字符串来做判断：

```
func IsNotExist(err error) bool {
    // 注意：不健壮
    return strings.Contains(err.Error(), "file does not exist")
}
```

但由于处理 I/O 错误的逻辑会随着平台的变化而变化，因此这种方法很不健壮，同样的错误可能会用完全不同的错误消息来报告。
检查错误消息是否包含特定的字符串，这种方法在单元测试中还算够用，但对于生产级的代码则远远不够。

一个更可靠的方法是用专门的类型来代表结构化的错误值。os 包定义了一个 PathError 类型来表示在与一个路径相关的操作上发生错误
（比如 Open 或者 Delete），一个类似的 LinkError 用来表述在与两个文件路径相关的操作上发生错误（比如 Symlink 和 Rename）。
下面是 os.PathError 的定义：

```
package os

// PathError 记录了错误以及错误相关的操作和文件路径
type PathError struct {
    Op string
    Path string
    Err error
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

很多客户端忽略了 PathError，该用一种统一的方法来处理所有的错误，即调用 Error 方法。
PathError 的 Error 方法只是拼接了所有的字段，而 PathError 的结构则保留了错误所有的底层信息。
对于那些需要区分错误的客户端，可以使用类型断言来检查错误的特定类型，这些类型包含的细节远远多于一个简单的字符串。

```
_, err = os.Open("/no/such/file")
fmt.Println(err) // "open /no/such/file: No such file or directory"
fmt.Println("%#v\n", err)
// 输出：
// &os.PathError{Op: "open", Path: "/no/such/file", Err: 0x2}
```

这也是之前三个帮助函数的工作方式。比如，如下所示的 IsNotExist 判断错误是否等于 `syscall.ENOENT`，或者
等于另一个错误 os.ErrNotExist，或者是一个 *PathError，并且底层的错误是上面二者之一。

```
import (
    "errors"
    "syscall"
)

var ErrNotExist = errors.New("file does not exist")
// IsNotExist 返回一个布尔值，该值表明错误是否代表文件或者目录不存在
// report that a file or directory does not exist. It is satisfied by
// ErrNotExist 和其它一些系统调用错误会返回 true
func IsNotExist(err error) bool {
    if pe, ok := err.(*PathError); ok {
        err = pe.Err
    }
    return err == syscall.ENOENT || err == ErrNotExist
}
```

实际使用情况如下：

```
_, err := os.Open("/no/such/file")
fmt.Println(os.IsNotExist(err)) // "true"
```

当然，如果错误消息已被 `fmt.Println` 这类的方法合并到一个大字符串中，那么 PathError
的结构信息就丢失了。错误识别通常必须在失败操作发生时马上处理，而不是等到错误消息返回给调用者之后。
