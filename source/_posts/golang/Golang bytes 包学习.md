---
title: Golang bytes 包学习
date: 2020-03-29 13:01:30
tags: [Go]
---

1. [常量](#常量)
2. [变量](#变量)
3. [函数](#函数)
    * [func Compare(a, b []byte) int](#Compare)
    * [func Contains(b, subslice []byte) bool](#Contains)
    * [func ContainsAny(b []byte, chars string) bool](#ContainsAny)
    * [func ContainsRune(b []byte, r rune) bool](#ContainsRune)
    * [func Count(s, sep []byte) int](#Count)
    * [func Equal(a, b []byte) bool](#Equal)
    * [func EqualFold(s, t []byte) bool](#EqualFold)
    * [func Fields(s []byte) [][]byte](#Fields)
    * [func FieldsFunc(s []byte, f func(rune) bool) [][]byte](#FieldsFunc)
    * [func HasPrefix(s, prefix []byte) bool](#HasPrefix)
    * [func HasSuffix(s, suffix []byte) bool](#HasSuffix)
    * [func Index(s, sep []byte) int](#Index)
    * [func IndexAny(s []byte, chars string) int](#IndexAny)
    * [func IndexByte(b []byte, c byte) int](#IndexByte)
    * [func IndexFunc(s []byte, f func(r rune) bool) int](#IndexFunc)
    * [func IndexRune(s []byte, r rune) int](#IndexRune)
    * [func Join(s [][]byte, sep []byte) []byte](#Join)
    * [func LastIndex(s, sep []byte) int](#LastIndex)
    * [func LastIndexAny(s []byte, chars string) int](#LastIndexAny)
    * [func LastIndexByte(s []byte, c byte) int](#LastIndexByte)
    * [func LastIndexFunc(s []byte, f func(r rune) bool) int](#LastIndexFunc)
    * [func Map(mapping func(r rune) rune, s []byte) []byte](#Map)
    * [func Repeat(b []byte, count int) []byte](#Repeat)
    * [func Replace(s, old, new []byte, n int) []byte](#Replace)
    * [func ReplaceAll(s, old, new []byte) []byte](#ReplaceAll)
    * [func Runes(s []byte) []rune](#Runes)
    * [func Split(s, sep []byte) [][]byte](#Split)
    * [func SplitAfter(s, sep []byte) [][]byte](#SplitAfter)
    * [func SplitAfterN(s, sep []byte, n int) [][]byte](#SplitAfterN)
    * [func SplitN(s, sep []byte, n int) [][]byte](#SplitN)
    * [func Title(s []byte) []byte](#Title)
    * [func ToLower(s []byte) []byte](#ToLower)
    * [func ToLowerSpecial(c unicode.SpecialCase, s []byte) []byte](#ToLowerSpecial)
    * [func ToTitle(s []byte) []byte](#ToTitle)
    * [func ToTitleSpecial(c unicode.SpecialCase, s []byte) []byte](#ToTitleSpecial)
    * [func ToUpper(s []byte) []byte](#ToUpper)
    * [func ToUpperSpecial(c unicode.SpecialCase, s []byte) []byte](#ToUpperSpecial)
    * [func ToValidUTF8(s, replacement []byte) []byte](#ToValidUTF8)
    * [func Trim(s []byte, cutset string) []byte](#Trim)
    * [func TrimFunc(s []byte, f func(r rune) bool) []byte](#TrimFunc)
    * [func TrimLeft(s []byte, cutset string) []byte](#TrimLeft)
    * [func TrimLeftFunc(s []byte, f func(r rune) bool) []byte](#TrimLeftFunc)
    * [func TrimPrefix(s, prefix []byte) []byte](#TrimPrefix)
    * [func TrimRight(s []byte, cutset string) []byte](#TrimRight)
    * [func TrimRightFunc(s []byte, f func(r rune) bool) []byte](#TrimRightFunc)
    * [func TrimSpace(s []byte) []byte](#TrimSpace)
    * [func TrimSuffix(s, suffix []byte) []byte](#TrimSuffix)
4. [Buffer](#Buffer)
    * [func NewBuffer(buf []byte) *Buffer](#NewBuffer)
    * [func NewBufferString(s string) *Buffer](#NewBufferString)
5. [Buffer 方法](#Buffer 方法)
    * [func (b *Buffer) Bytes() []byte](#Bytes)
    * [func (b *Buffer) Cap() int](#Cap)
    * [func (b *Buffer) Grow(n int)](#Grow)
    * [func (b *Buffer) Len() int](#Len)
    * [func (b *Buffer) Next(n int) []byte](#Next)
    * [func (b *Buffer) Read(p []byte) (n int, err error)](#Read)
    * [func (b *Buffer) ReadByte() (byte, error)](#ReadByte)
    * [func (b *Buffer) ReadBytes(delim byte) (line []byte, err error)](#ReadBytes)
    * [func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error)](#ReadFrom)
    * [func (b *Buffer) ReadRune() (r rune, size int, err error)](#ReadRune)
    * [func (b *Buffer) ReadString(delim byte) (line string, err error)](#ReadString)
    * [func (b *Buffer) Reset()](#Reset)
    * [func (b *Buffer) String() string](#String)
    * [func (b *Buffer) Truncate(n int)](#Truncate)
    * [func (b *Buffer) UnreadByte() error](#UnreadByte)
    * [func (b *Buffer) UnreadRune() error](#UnreadRune)
    * [func (b *Buffer) Write(p []byte) (n int, err error)](#Write)
    * [func (b *Buffer) WriteByte(c byte) error](#WriteByte)
    * [func (b *Buffer) WriteRune(r rune) (n int, err error)](#WriteRune)
    * [func (b *Buffer) WriteString(s string) (n int, err error)](#WriteString)
    * [func (b *Buffer) WriteTo(w io.Writer) (n int64, err error)](#WriteTo)
6. [Reader](#Reader)
    * [func NewReader(b []byte) *Reader](#NewReader)
7. [Reader 方法](#Reader 方法)
    * [func (r *Reader) Len() int](#Len)
    * [func (r *Reader) Read(b []byte) (n int, err error)](#Read)
    * [func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)](#ReadAt)
    * [func (r *Reader) ReadByte() (byte, error)](#ReadByte)
    * [func (r *Reader) ReadRune() (ch rune, size int, err error)](#ReadRune)
    * [func (r *Reader) Reset(b []byte)](#Reset)
    * [func (r *Reader) Seek(offset int64, whence int) (int64, error)](#Seek)
    * [func (r *Reader) Size() int64](#Size)
    * [func (r *Reader) UnreadByte() error](#UnreadByte)
    * [func (r *Reader) UnreadRune() error](#UnreadRune)
    * [func (r *Reader) WriteTo(w io.Writer) (n int64, err error)](#WriteTo)


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
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleCompare() {
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
    
    	// Output:
    	// a = 'a', b = 'b', a < b
    	// a = 'a', b = 'a', a <= b
    	// a = 'b', b = 'a', a > b
    	// a = 'b', b = 'b', a >= b
    	// a = 'b', b = 'b', a == b
    	// a = 'b', b = 'c', a != b
    }
    
    func ExampleSearch() {
    	var needle []byte
    	var haystack [][]byte // 假设是有序的
    
    	// 模拟测试数据
    	needle = []byte("ccb")
    	haystack = [][]byte{
    		[]byte("cca"),
    		[]byte("ccb"),
    		[]byte("ccc"),
    	}
    
        // haystack 需要是有序的
    	i := sort.Search(len(haystack), func(i int) bool {
    		// Return haystack[i] >= needle.
    		return bytes.Compare(haystack[i], needle) >= 0
    	})
    	if i < len(haystack) && bytes.Equal(haystack[i], needle) {
    		fmt.Println("Found it!")
    	}
    
    	// Output:
    	// Found it!
    }
    ```
</details>

### Contains

```
func Contains(b, subslice []byte) bool
```

检查 `b` 里面是否包含了 `subslice` 字节序列。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleContains() {
    	fmt.Println(bytes.Contains([]byte("seafood"), []byte("foo")))
    	fmt.Println(bytes.Contains([]byte("seafood"), []byte("bar")))
    	fmt.Println(bytes.Contains([]byte("seafood"), []byte("")))
    	fmt.Println(bytes.Contains([]byte(""), []byte("")))
    
    	// Output:
    	// true
    	// false
    	// true
    	// true
    }
    ```
</details>


### ContainsAny

```
func ContainsAny(b []byte, chars string) bool
```

`ContainsAny` 检查字符串 `chars` utf-8编码的字符切片里面是否有任何字节在 b 里面。

> chars 是空字符串的时候都返回 false

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleContainsAny() {
    	fmt.Println(bytes.ContainsAny([]byte("I like seafood."), "fAo!"))
    	fmt.Println(bytes.ContainsAny([]byte("I like seafood."), "去是伟大的."))
    	fmt.Println(bytes.ContainsAny([]byte("I like seafood."), ""))
    	fmt.Println(bytes.ContainsAny([]byte(""), ""))
    
    	// Output:
    	// true
    	// true
    	// false
    	// false
    }
    ```
</details>


### ContainsRune

```
func ContainsRune(b []byte, r rune) bool
```

> rune 可以简单看作是字符类型（可以是多字节字符）。

`ContainsRune` 检查 `r` 是否出现在 utf-8 编码的字节序列中。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleContainsRune() {
    	fmt.Println(bytes.ContainsRune([]byte("I like seafood."), 'f'))
    	fmt.Println(bytes.ContainsRune([]byte("I like seafood."), 'ö'))
    	fmt.Println(bytes.ContainsRune([]byte("去是伟大的!"), '大'))
    	fmt.Println(bytes.ContainsRune([]byte("去是伟大的!"), '!'))
    	fmt.Println(bytes.ContainsRune([]byte(""), '@'))
    
    	// Output:
    	// true
    	// false
    	// true
    	// true
    	// false
    }
    ```
</details>


### Count

```
func Count(s, sep []byte) int
```

`Count` 用来计算 `sep` 出现在 `s` 中的次数，其中 `s` 是 utf-8 编码的字节序列。

> 如果 sep 为空，则返回 s 的长度 + 1

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleCount() {
    	fmt.Println(bytes.Count([]byte("cheese"), []byte("e")))
    	// before & after each rune
    	fmt.Println(bytes.Count([]byte("five"), []byte("")))
    
    	// Output:
    	// 3
    	// 5
    }
    ```
</details>


### Equal

```
func Equal(a, b []byte) bool
```

`Equal` 判断字节切片 `a` 和 `b` 的是否相等。如果传 `nil`，则视作空切片。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleEqual() {
    	fmt.Println(bytes.Equal([]byte("Go"), []byte("Go")))
    	fmt.Println(bytes.Equal([]byte("Go"), []byte("C++")))
    
    	// Output:
    	// true
    	// false
    }
    ```
</details>


### EqualFold

```
func EqualFold(s, t []byte) bool
```

`EqualFold` 比较 UTF-8 编码是否相等，不区分大小写。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleEqualFold() {
    	fmt.Println(bytes.EqualFold([]byte("Go"), []byte("go")))
    
    	// Output:
    	// true
    }
    ```
</details>


### Fields

```
func Fields(s []byte) [][]byte
```

`Fields` 将参数当作一个 UTF-8 编码的字节切片，根据 `unicode.IsSpace` 定义的空白字符来拆分该字节切片，但是拆分的结果不会包含空白字符。如果只包含空白字符，则返回一个空的二维切片。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleFields() {
    	fmt.Printf("Fields are: %q\n", bytes.Fields([]byte("  foo bar baz  ")))
    
    	// Output:
    	// Fields are: ["foo" "bar" "baz"]
    }
    ```
</details>


### FieldsFunc

```
func FieldsFunc(s []byte, f func(rune) bool) [][]byte
```

和 `Fields` 类似，只不过这里接收一个回调让用户可以返回一个布尔值，系统根据返回值决定是否使用这个字符来分割 `Field`。如果所有的字符都满足 `f(r)` 或者字节切片 `s` 的长度为 0，`FieldsFunc` 返回一个空的切片。`FieldsFunc` 不保证其调用 `f(c)` 的顺序。如果 `f` 对于给定的 `c` 没有返回一致的结果，则 `FieldsFunc` 可能会崩溃。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleFieldsFunc() {
    	f := func(c rune) bool {
    		return !unicode.IsLetter(c) && !unicode.IsNumber(c)
    	}
    	fmt.Printf("Fields are: %q\n", bytes.FieldsFunc([]byte("  foo1;bar2,baz3..."), f))
    
    	// Output:
    	// Fields are: ["foo1" "bar2" "baz3"]
    }
    ```
</details>


### HasPrefix

```
func HasPrefix(s, prefix []byte) bool
```

`HasPrefix` 检查字节切片是否以 `prefix` 切片开头。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleHasPrefix() {
    	fmt.Println(bytes.HasPrefix([]byte("Gopher"), []byte("Go")))
    	fmt.Println(bytes.HasPrefix([]byte("Gopher"), []byte("C")))
    	fmt.Println(bytes.HasPrefix([]byte("Gopher"), []byte("")))
    
    	// Output:
    	// true
    	// false
    	// true
    }
    ```
</details>


### HasSuffix

```
func HasSuffix(s, suffix []byte) bool
```

`HasPrefix` 检查字节切片 `s` 是否以字节切片 `prefix` 结尾。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleHasSuffix() {
    	fmt.Println(bytes.HasSuffix([]byte("Amigo"), []byte("go")))
    	fmt.Println(bytes.HasSuffix([]byte("Amigo"), []byte("O")))
    	fmt.Println(bytes.HasSuffix([]byte("Amigo"), []byte("Ami")))
    	fmt.Println(bytes.HasSuffix([]byte("Amigo"), []byte("")))
    
    	// Output:
    	// true
    	// false
    	// false
    	// true
    }
    ```
</details>


### Index

```
func Index(s, sep []byte) int
```

`Index` 返回 `sep` 在 `s` 中的下标，如果 `sep` 不在 `s` 中则返回 -1。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleIndex() {
    	fmt.Println(bytes.Index([]byte("chicken"), []byte("ken")))
    	fmt.Println(bytes.Index([]byte("chicken"), []byte("dmr")))
    
    	// Output:
    	// 4
    	// -1
    }
    ```
</details>


### IndexAny

```
func IndexAny(s []byte, chars string) int
```

检查 `chars` 任意字符是否在 `s` 中，如果在，则返回对应的下标，不存在或者 `chars` 为空则返回 -1。该函数将 `s` 视作 UTF-8 编码的字节切片。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleIndexAny() {
    	fmt.Println(bytes.IndexAny([]byte("chicken"), "aeiouy"))
    	fmt.Println(bytes.IndexAny([]byte("crwth"), "aeiouy"))
    
    	// Output:
    	// 2
    	// -1
    }
    ```
</details>


### IndexByte

```
func IndexByte(b []byte, c byte) int
```

`IndexByte` 返回字节 `c` 在字节切片 `b` 中首次出现的下标，如果 `c` 不在 `b` 中返回 -1。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleIndexByte() {
    	fmt.Println(bytes.IndexByte([]byte("chicken"), byte('k')))
    	fmt.Println(bytes.IndexByte([]byte("chicken"), byte('g')))
    
    	// Output:
    	// 4
    	// -1
    }
    ```
</details>


### IndexFunc

```
func IndexFunc(s []byte, f func(r rune) bool) int
```

和 `Index` 类似，不过接收一个自定义的回调函数来判断字符是否是满足条件的字符，是则返回 true，这个字符的索引就是 `IndexFunc` 的返回值。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleIndexFunc() {
    	f := func(c rune) bool {
    		return unicode.Is(unicode.Han, c)
    	}
    	fmt.Println(bytes.IndexFunc([]byte("Hello, 世界"), f))
    	fmt.Println(bytes.IndexFunc([]byte("Hello, world"), f))
    
    	// Output:
    	// 7
    	// -1
    }
    ```
</details>


### IndexRune

```
func IndexRune(s []byte, r rune) int
```

`IndexRune` 将 `s` 当作 UTF-8 字节切片，返回 `r` 在 `s` 中第一次出现的索引，没有在 `s` 中则返回 -1。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleIndexRune() {
    	fmt.Println(bytes.IndexRune([]byte("chicken"), 'k'))
    	fmt.Println(bytes.IndexRune([]byte("chicken"), 'd'))
    
    	// Output:
    	// 4
    	// -1
    }
    ```
</details>


### Join

```
func Join(s [][]byte, sep []byte) []byte
```

`Join` 使用 `sep` 作为分隔符来连接多个 `[]byte`，返回连接后的结果。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleJoin() {
    	s := [][]byte{
    		[]byte("foo"),
    		[]byte("bar"),
    		[]byte("baz"),
    	}
    	fmt.Printf("%s", bytes.Join(s, []byte(", ")))
    
    	// Output:
    	// foo, bar, baz
    }
    ```
</details>


### LastIndex

```
func LastIndex(s, sep []byte) int
```

`LastIndex` 返回 `sep` 在 `s` 中最后出现的索引，没有出现返回 -1。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleLastIndex() {
    	fmt.Println(bytes.Index([]byte("go gopher"), []byte("go")))
    	fmt.Println(bytes.LastIndex([]byte("go gopher"), []byte("go")))
    	fmt.Println(bytes.LastIndex([]byte("go gopher"), []byte("rodent")))
    
    	// Output:
    	// 0
    	// 3
    	// -1
    }
    ```
</details>


### LastIndexAny

```
func LastIndexAny(s []byte, chars string) int
```

和 `IndexAny` 类似，但是这个是返回最终出现的位置。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleLastIndexAny() {
    	fmt.Println(bytes.LastIndexAny([]byte("go gopher"), "MüQp"))
    	fmt.Println(bytes.LastIndexAny([]byte("go 地鼠"), "地大"))
    	fmt.Println(bytes.LastIndexAny([]byte("go gopher"), "z,!."))
    
    	// Output:
    	// 5
    	// 3
    	// -1
    }
    ```
</details>


### LastIndexByte

```
func LastIndexByte(s []byte, c byte) int
```

`LastIndexByte` 返回 `c` 在 `s` 中最后出现的下标。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleLastIndexByte() {
    	fmt.Println(bytes.LastIndexByte([]byte("go gopher"), byte('g')))
    	fmt.Println(bytes.LastIndexByte([]byte("go gopher"), byte('r')))
    	fmt.Println(bytes.LastIndexByte([]byte("go gopher"), byte('z')))
    
    	// Output:
    	// 3
    	// 8
    	// -1
    }
    ```
</details>


### LastIndexFunc

```
func LastIndexFunc(s []byte, f func(r rune) bool) int
```

`LastIndexFunc` 和 `IndexFunc` 的效果类似，只不过这个是返回最后出现的下标。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleLastIndexFunc() {
    	fmt.Println(bytes.LastIndexFunc([]byte("go gopher!"), unicode.IsLetter))
    	fmt.Println(bytes.LastIndexFunc([]byte("go gopher!"), unicode.IsPunct))
    	fmt.Println(bytes.LastIndexFunc([]byte("go gopher!"), unicode.IsNumber))
    
    	// Output:
    	// 8
    	// 9
    	// -1
    }
    ```
</details>


### Map

```
func Map(mapping func(r rune) rune, s []byte) []byte
```

对 `s` 中的每一个字符 `c` 调用 `mapping(c)`，返回新字符组成的字节切片，如果返回 -1，则返回的结果不包含这个字符。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleMap() {
    	rot13 := func(r rune) rune {
    		switch {
    		case r >= 'A' && r <= 'Z':
    			return 'A' + (r - 'A' + 13) % 26
    		case r >= 'a' && r <= 'z':
    			return 'a' + (r - 'a' + 13) % 26
    		}
    		return r
    	}
    
    	fmt.Printf("%s", bytes.Map(rot13, []byte("'Twas brillig and the slithy gopher...")))
    
    	// Output:
    	// 'Gjnf oevyyvt naq gur fyvgul tbcure...
    }
    ```
</details>


### Repeat

```
func Repeat(b []byte, count int) []byte
```

`Repeat` 将一个字节切片 `b` 重复 `count` 次，返回最后的结果。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleRepeat() {
    	fmt.Printf("ba%s", bytes.Repeat([]byte("na"), 2))
    
    	// Output:
    	// banana
    }
    ```
</details>


### Replace

```
func Replace(s, old, new []byte, n int) []byte
```

将 `s` 字节切片中的 `old` 切片替换为 `new` 切片，最多替换 `n` 次。如果 `n` 小于 0，则不限制替换次数。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleReplace() {
    	fmt.Printf("%s\n", bytes.Replace([]byte("oink oink oink"), []byte("k"), []byte("ky"), 2))
    	fmt.Printf("%s\n", bytes.Replace([]byte("oink oink oink"), []byte("oink"), []byte("moo"), -1))
    
    	// Output:
    	// oinky oinky oink
    	// moo moo moo
    }
    ```
</details>


### ReplaceAll

```
func ReplaceAll(s, old, new []byte) []byte
```

和 `Replace` 类似，不过这个是替换所有的 `old` 为 `new`。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleReplaceAll() {
    	fmt.Printf("%s\n", bytes.ReplaceAll([]byte("oink oink oink"), []byte("oink"), []byte("moo")))
    
    	// Output:
    	// moo moo moo
    }
    ```
</details>


### Runes

```
func Runes(s []byte) []rune
```

`Runes` 将字节切片转换为 `[]byte` 类型，`s` 被视作是 UTF-8 编码的字节切片。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleRunes() {
    	rs := bytes.Runes([]byte("go gopher"))
    	for _, r := range rs {
    		fmt.Printf("%#U\n", r)
    	}
    
    	// Output:
    	// U+0067 'g'
    	// U+006F 'o'
    	// U+0020 ' '
    	// U+0067 'g'
    	// U+006F 'o'
    	// U+0070 'p'
    	// U+0068 'h'
    	// U+0065 'e'
    	// U+0072 'r'
    }
    ```
</details>


### Split

```
func Split(s, sep []byte) [][]byte
```

和 `Join` 操作相反，根据 `sep` 来拆分 `s`。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleSplit() {
    	fmt.Printf("%q\n", bytes.Split([]byte("a,b,c"), []byte(",")))
    	fmt.Printf("%q\n", bytes.Split([]byte("a man a plan a cancel panama"), []byte("a ")))
    	fmt.Printf("%q\n", bytes.Split([]byte(" xyz "), []byte("")))
    	fmt.Printf("%q\n", bytes.Split([]byte(""), []byte("Bernardo O'Higgins")))
    
    	// Output:
    	// ["a" "b" "c"]
    	// ["" "man " "plan " "cancel panama"]
    	// [" " "x" "y" "z" " "]
    	// [""]
    }
    ```
</details>


### SplitAfter

```
func SplitAfter(s, sep []byte) [][]byte
```

和 `Split` 类似，只不过最终结果的每一个字节切片里面包含了分隔符 `sep`。包含在前一个字节切片中。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleSplitAfter() {
    	fmt.Printf("%q\n", bytes.SplitAfter([]byte("a,b,c"), []byte(",")))
    
    	// Output:
    	// ["a," "b," "c"]
    }
    ```
</details>


### SplitAfterN

```
func SplitAfterN(s, sep []byte, n int) [][]byte
```

和 `SplitAfter` 类似，但是最终返回的 `[][]byte` 的长度，最多为 `n`，

* n > 0: 最多返回 `n` 个子切片，最后一个是剩余的切片

* n == 0: 返回 nil（0 个子切片）

* n < 0: 返回所有子切片

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleSplitAfterN() {
    	fmt.Printf("%q\n", bytes.SplitAfterN([]byte("a,b,c"), []byte(","), 2))
    
    	// Output:
    	// ["a," "b,c"]
    }
    ```
</details>


### SplitN

```
func SplitN(s, sep []byte, n int) [][]byte
```

和 `SplitAfterN` 类似，但是这个函数返回的子切片不包含分割符号（除了最后一个有可能包含）。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleSplitN() {
    	fmt.Printf("%q\n", bytes.SplitN([]byte("a,b,c"), []byte(","), 2))
    	z := bytes.SplitN([]byte("a,b,c"), []byte(","), 0)
    	fmt.Printf("%q (nil = %v)\n", z, z == nil)
    
    	// Output:
    	// ["a" "b,c"]
    	//[] (nil = true)
    }
    ```
</details>


### Title

```
func Title(s []byte) []byte
```

将字节切片转换为标题格式（每个单词改成大写）。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleTitle() {
    	fmt.Printf("%s", bytes.Title([]byte("her royal highness")))
    
    	// Output:
    	// Her Royal Highness
    }
    ```
</details>


### ToLower

```
func ToLower(s []byte) []byte
```

返回一个新的字节切片，所有的 Unicode 字符被转换为小写格式。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleToLower() {
    	fmt.Printf("%s", bytes.ToLower([]byte("Gopher")))
    
    	// Output:
    	// gopher
    }
    ```
</details>


### ToLowerSpecial

```
func ToLowerSpecial(c unicode.SpecialCase, s []byte) []byte
```

第一个参数是指定不同语言的大小写的，有可能某些语言的大小写和英语不太一样，其他和 `ToLower` 类似。


<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleToLowerSpecial() {
    	str := []byte("AHOJ VÝVOJÁRİ GOLANG")
    	totitle := bytes.ToLowerSpecial(unicode.AzeriCase, str)
    	fmt.Println("Original : " + string(str))
    	fmt.Println("ToLower : " + string(totitle))
    
    	// Output:
    	// Original : AHOJ VÝVOJÁRİ GOLANG
    	// ToLower : ahoj vývojári golang
    }
    ```
</details>


### ToTitle

```
func ToTitle(s []byte) []byte
```

`ToTitle` 将 `s` 视为 UTF-8 编码的字节，并返回一个副本，其中所有 Unicode 字母都映射到其标题大小写。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleToTitle() {
    	fmt.Printf("%s\n", bytes.ToTitle([]byte("loud noises")))
    	fmt.Printf("%s\n", bytes.ToTitle([]byte("хлеб")))
    
    	// Output:
    	// LOUD NOISES
    	// ХЛЕБ
    }
    ```
</details>


### ToTitleSpecial

```
func ToTitleSpecial(c unicode.SpecialCase, s []byte) []byte
```

`ToTitleSpecial` 将 `s` 视为 UTF-8 编码的字节，并返回一个副本，其中所有 Unicode 字母均映射到其标题大小写，并优先使用特殊的大小写规则。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleToTitleSpecial() {
    	str := []byte("ahoj vývojári golang")
    	totitle := bytes.ToTitleSpecial(unicode.AzeriCase, str)
    	fmt.Println("Original : " + string(str))
    	fmt.Println("ToTitle : " + string(totitle))
    
    	// Output:
    	// Original : ahoj vývojári golang
    	// ToTitle : AHOJ VÝVOJÁRİ GOLANG
    }
    ```
</details>


### ToUpper

```
func ToUpper(s []byte) []byte
```

`ToUpper` 返回字节片 `s` 的副本，其中所有 Unicode 字母都映射到其大写字母。

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleToUpper() {
    	fmt.Printf("%s", bytes.ToUpper([]byte("Gopher")))
    
    	// Output:
    	// GOPHER
    }
    ```
</details>


### ToUpperSpecial

```
func ToUpperSpecial(c unicode.SpecialCase, s []byte) []byte
```

`ToUpperSpecial` 将 `s` 视为 UTF-8 编码的字节，并返回一个副本，其中所有 Unicode 字母均映射为它们的大写字母，优先考虑特殊的大小写规则。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleToUpperSpecial() {
    	str := []byte("ahoj vývojári golang")
    	totitle := bytes.ToUpperSpecial(unicode.AzeriCase, str)
    	fmt.Println("Origin : " + string(str))
    	fmt.Println("ToUpper : " + string(totitle))
    
    	// Output:
    	// Origin : ahoj vývojári golang
    	// ToUpper : AHOJ VÝVOJÁRİ GOLANG
    }
    ```
</details>


### ToValidUTF8

```
func ToValidUTF8(s, replacement []byte) []byte
```

`ToValidUTF8` 将 `s` 视为 UTF-8 编码的字节，并返回一个副本，将其中无效的 UTF-8 替换为 `replacement`，`replacement` 可以为空。


### Trim

```
func Trim(s []byte, cutset string) []byte
```

`Trim` 去除 `s` 前后 `cutset` 字符串里的子字符，返回最后的结果。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleTrim() {
    	fmt.Printf("[%q]", bytes.Trim([]byte(" !!! Achtung! Achtung! !!!"), "! "))
    
    	// Output:
    	// ["Achtung! Achtung"]
    }
    ```
</details>

### TrimFunc

```
func TrimFunc(s []byte, f func(r rune) bool) []byte
```

和 `Trim` 类似，只不过可以通过回调来筛选。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleTrimFunc() {
    	fmt.Println(string(bytes.TrimFunc([]byte("go-gopher!"), unicode.IsLetter)))
    	fmt.Println(string(bytes.TrimFunc([]byte("\"go-gopher!\""), unicode.IsLetter)))
    	fmt.Println(string(bytes.TrimFunc([]byte("go-gopher!"), unicode.IsPunct)))
    	fmt.Println(string(bytes.TrimFunc([]byte("1234go-gopher!567"), unicode.IsNumber)))
    
    	// Output:
    	// -gopher!
    	// "go-gopher!"
    	// go-gopher
    	// go-gopher!
    }
    ```
</details>


### TrimLeft

```
func TrimLeft(s []byte, cutset string) []byte
```

和 `Trim` 类似，但是只移除 `s` 左边的字符。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleTrimLeft() {
    	fmt.Print(string(bytes.TrimLeft([]byte("453gopher8257"), "0123456789")))
    
    	// Output:
    	// gopher8257
    }
    ```
</details>


### TrimLeftFunc

```
func TrimLeftFunc(s []byte, f func(r rune) bool) []byte
```

和 `TrimLeft` 类似，但是可以通过回调筛选需要移除的字符。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleTrimLeftFunc() {
    	fmt.Println(string(bytes.TrimLeftFunc([]byte("go-gopher"), unicode.IsLetter)))
    	fmt.Println(string(bytes.TrimLeftFunc([]byte(".go-gopher!"), unicode.IsPunct)))
    	fmt.Println(string(bytes.TrimLeftFunc([]byte("1234go-gopher!567"), unicode.IsNumber)))
    
    	// Output:
    	// -gopher
    	// go-gopher!
    	// go-gopher!567
    }
    ```
</details>


### TrimPrefix

```
func TrimPrefix(s, prefix []byte) []byte
```

和 `Trim` 类似，但是是移除 `prefix` 指定的前缀。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleTrimPrefix() {
    	var b = []byte("Goodbye,, world!")
    	b = bytes.TrimPrefix(b, []byte("Goodbye,"))
    	b = bytes.TrimPrefix(b, []byte("See ya,"))
    	fmt.Printf("Hello%s", b)
    
    	// Output:
    	// Hello, world!
    }
    ```
</details>


### TrimRight

```
func TrimRight(s []byte, cutset string) []byte
```

和 `TrimLeft` 类似，只不过是移除 `s` 末尾的字符。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleTrimRight() {
    	fmt.Print(string(bytes.TrimRight([]byte("453gopher8257"), "0123456789")))
    
    	// Output:
    	// 453gopher
    }
    ```
</details>


### TrimRightFunc

```
func TrimRightFunc(s []byte, f func(r rune) bool) []byte
```

和 `TrimRight` 类似，只不过可以接收一个回调来筛选需要移除的字符。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"unicode"
    )
    
    func ExampleTrimRightFunc() {
    	fmt.Println(string(bytes.TrimRightFunc([]byte("go-gopher"), unicode.IsLetter)))
    	fmt.Println(string(bytes.TrimRightFunc([]byte("go-gopher!"), unicode.IsPunct)))
    	fmt.Println(string(bytes.TrimRightFunc([]byte("1234go-gopher!567"), unicode.IsNumber)))
    
    	// Output:
    	// go-
    	// go-gopher
    	// 1234go-gopher!
    }
    ```
</details>


### TrimSpace

```
func TrimSpace(s []byte) []byte
```

移除字节切片 `s` 前后的空白字符。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleTrimSpace() {
    	fmt.Printf("%s", bytes.TrimSpace([]byte(" \t\n a lone gopher \n\t\r\n")))
    
    	// Output:
    	// a lone gopher
    }
    ```
</details>


### TrimSuffix

```
func TrimSuffix(s, suffix []byte) []byte
```

移除字节切片 `s` 的后缀。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"os"
    )
    
    func ExampleTrimSuffix() {
    	var b = []byte("Hello, goodbye, etc!")
    	b = bytes.TrimSuffix(b, []byte("goodbye, etc!"))
    	b = bytes.TrimSuffix(b, []byte("gopher"))
    	b = append(b, bytes.TrimSuffix([]byte("world!"), []byte("x!"))...)
    	os.Stdout.Write(b)
    
    	// Output:
    	// Hello, world!
    }
    ```
</details>



## Buffer

一个 `Buffer` 是一个大小不固定的字节缓冲区。`Buffer` 的零值是一个空的缓冲区。

<details>
    <summary>点此查看实例</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    	"os"
    )
    
    func ExampleBuffer() {
    	var b bytes.Buffer // 不需要初始化
    	b.Write([]byte("Hello "))
    	fmt.Fprintf(&b, "world!")
    	b.WriteTo(os.Stdout)
    
    	// Output:
    	// Hello world!
    }
    ```
</details>

<details>
    <summary>点此查看实例(Reader)</summary>

    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"encoding/base64"
    	"io"
    	"os"
    )
    
    func ExampleBuffer1() {
    	// A Buffer can turn a string or a []byte into an io.Reader.
    	buf := bytes.NewBufferString("R29waGVycyBydWxlIQ==")
    	dec := base64.NewDecoder(base64.StdEncoding, buf)
    	io.Copy(os.Stdout, dec)
    
    	// Output:
    	// Gophers rule!
    }
    ```
</details>


### NewBuffer

```
func NewBuffer(buf []byte) *Buffer
```

`NewBuffer` 使用 `buf` 作为其初始内容创建并初始化一个新的 `Buffer`。 新的 `Buffer` 拥有 `buf` 的所有权，并且在此调用之后，调用方不应使用buf。 `NewBuffer` 旨在准备一个 `Buffer` 以读取现有数据。 它也可以用来设置用于写入的内部缓冲区的初始大小。 为此，`buf` 应该具有所需的容量，但长度为零。

在大多数情况下，`new(Buffer)`（或仅声明一个 `Buffer` 变量）足以初始化 `Buffer`。


### NewBufferString

```
func NewBufferString(s string) *Buffer
```

`NewBufferString` 使用字符串 `s` 作为其初始内容创建并初始化一个新的 `Buffer`。目的是准备一个缓冲区以读取现有的字符串。

在大多数情况下，`new(Buffer)` (或仅声明一个 `Buffer` 变量) 足以初始化 `Buffer`。


## Buffer 方法

### Bytes

```
func (b *Buffer) Bytes() []byte
```

字节返回长度为 `b.Len()` 的切片，其中包含缓冲区的未读部分。 该片仅在下一次修改缓冲区之前有效（即，仅在下一次调用诸如 `Read`，`Write`，`Reset` 或 `Truncate` 之类的方法之前）才有效。 切片至少在下一次缓冲区修改之前就将缓冲区内容作为别名，因此对切片的立即更改将影响将来读取的结果。


### Cap

```
func (b *Buffer) Cap() int
```

`Cap` 返回缓冲区底层字节切片的容量，即为缓冲区数据分配的总空间。


### Grow

```
func (b *Buffer) Grow(n int)
```

如有必要，可以增加缓冲区的容量，以保证另外 n 个字节的空间。在 `Grow(n)` 之后，至少可以将 n 个字节写入缓冲区，而无需进行其他分配。如果 n 为负数，`Grow` 会 panic。如果缓冲区无法增长，会引起一个 `ErrTooLarge` 的 panic。


### Len

```
func (b *Buffer) Len() int
```

`Len` 返回缓冲区未读部分的字节数；`b.Len() == len(b.Bytes())`


<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleBufferLen() {
    	var b bytes.Buffer
    	b.Grow(64)
    	b.Write([]byte("abcde"))
    	fmt.Printf("%d", b.Len())
    
    	// Output:
    	// 5
    }
    ```
</details>


### Next

```
func (b *Buffer) Next(n int) []byte
```

`Next` 返回一个包含缓冲区中接下来 `n` 个字节的切片，使缓冲区前进，就好像该字节已由 `Read` 返回。如果缓冲区中的字节数少于 `n` 个，则 `Next` 返回整个缓冲区。切片仅在下一次调用 `read` 或 `write` 方法之前才有效。


### Read

```
func (b *Buffer) Read(p []byte) (n int, err error)
```

`Read` 从缓冲区中读取下一个 `len(p)` 字节，或者直到缓冲区耗尽为止。返回值 `n` 是读取的字节数。如果缓冲区没有数据要返回，则 `err` 为 `io.EOF`（除非 `len(p)` 为 0）；否则为零。


### ReadByte

```
func (b *Buffer) ReadByte() (byte, error)
```

`ReadByte` 读取并从缓冲区返回下一个字节。如果没有可用的字节，则返回错误 `io.EOF`。


### ReadBytes

```
func (b *Buffer) Readbytes(delim byte) (line []byte, err error)
```

`ReadBytes` 读取直到输入中第一次出现 `delim` 为止，并返回一个包含数据的切片，该数据直到并包括 `delim`。如果 `ReadBytes` 在找到定界符之前遇到错误，它将返回错误之前读取的数据和错误本身（通常为 `io.EOF`）。并且仅当返回的数据未以 `delim` 结尾时，`ReadBytes` 返回 `err != nil`。


### ReadFrom

```
func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error)
```

`ReadFrom` 从 `r` 读取数据，直到 `EOF` 并将其附加到缓冲区，然后根据需要增大缓冲区。返回值 `n` 是读取的字节数。读取期间遇到的除 `io.EOF` 以外的任何错误也将返回。如果缓冲区太大，`ReadFrom` 会引发 `ErrTooLarge` panic。


### ReadRune

```
func (b *Buffer) ReadRune() (r rune, size int, err error)
```

`ReadRune` 从缓冲区读取并返回下一个 UTF-8 编码的 Unicode 字符。如果没有可用的字节，则返回的错误是 `io.EOF`。如果字节是错误的 UTF-8 编码，则它消耗一个字节并返回 U+FFFD，1。


### ReadString

```
func (b *Buffer) ReadString(delim type) (line string, err error)
```

`ReadString` 读取直到输入中第一次出现 `delim` 为止，并返回一个字符串，其中包含直到定界符（包括定界符）的数据。如果 `ReadString` 在找到定界符之前遇到错误，它将返回错误之前读取的数据和错误本身（通常为 `io.EOF`）。当且仅当返回的数据未以 `delim` 结尾时，`ReadString` 才返回 `err != nil`。


### Reset

```
func (b *Buffer) Reset()
```

重置会将缓冲区重置为空，但会保留基础存储空间以供将来的写操作使用。重置与 `Truncate(0)` 相同。


### String

```
func (b *Buffer) String()
```

`String` 以字符串形式返回缓冲区未读部分的内容。如果 `Buffer` 是 `nil` 指针，则返回 "<nil>"。

要更有效地构建字符串，请常见 `strings.Builder` 类型。


### Truncate

```
func (b *Buffer) Truncate(n int)
```

`Truncate` 会丢弃缓冲区前 `n` 个未读取字节以外的所有字节，但会继续使用相同的已分配存储。如果 `n` 为负数或大于缓冲区的长度，则会发生 panic.


### UnreadByte

```
func (b *Buffer) UnreadByte() error
```

`UnreadByte` 将最近一次成功读取操作的最后一个字节重新设置为未读取。如果自从上次成功读取后有写缓冲区操作，或者上次读取有错误，或者上次读取到 0 字节，`UnreadByte` 返回一个错误。


### UnreadRune

```
func (b *Buffer) UnreadRune() error
```

和 `UnreadByte` 类似，只不过操作的是 `rune`


### Write

```
func (b *Buffer) Write(p []byte) (n int, err error)
```

写操作会将 `p` 的内容附加到缓冲区，并根据需要扩展缓冲区。返回值 `n` 是 `p` 的长度；错误始终为零。如果缓冲区太大，则 `Write` 会引发 `ErrTooLarge` panic。


### WriteByte

```
func (b *Buffer) WriteByte(c byte) error
```

写入一个字节到 Buffer。

`WriteByte` 将字节 `c` 附加到缓冲区，根据需要增大缓冲区。返回的错误始终为 `nil`，但包含该错误以匹配 `bufio.Writer` 的 `WriteByte`。如果缓冲区太大，`WriteByte` 会引发 `ErrTooLarge` panic。


### WriteRune

```
func (b *Buffer) WriteRune(r rune) (n int, err error)
```

和 `ReadRune` 类似，不过 `WriteRune` 是写入一个 `rune` 类型的数据。


### WriteString

```
func (b *Buffer) WriteString(s string) (n int, err error)
```

写入一个字符串到 Buffer。 

`WriteString` 将 `s` 的内容附加到缓冲区，根据需要增大缓冲区。返回值 `n` 是 `s` 的长度；错误始终为 `nil`。如果缓冲区太大，引发 `ErrTooLarge` panic。


### WriteTo

```
func (b *Buffer) WriteTo(w io.Writer) (n int64, err error)
```

`WriteTo` 将数据写入 `w`，直到缓冲区耗尽或发生错误。返回值 `n` 是写入的字节数。它始终适合 `int`，但与 `io.WriterTo` 接口匹配为 `int64`。写入期间遇到的任何错误也将返回。



## Reader

`Reader` 通过读取字节切片来实现 `io.Reader`, `io.ReaderAt`, `io.WriterTo`, `io.Seeker`, `io.ByteScanner` 和 `io.RuneScanner` 接口。与 `Buffer` 不同，`Reader` 是只读的，并支持查找。`Reader` 的零值的操作类似空切片的 `Reader`。


### NewReader

```
func NewReader(b []byte) *Reader
```

`NewReader` 从 `b` 返回一个新的 `Reader`，这个 `Reader` 将会从 `b` 读取数据。


### Len

```
func (r *Reader) Len() int
```

`Len` 返回切片中未读部分的长度

<details>
    <summary>点此查看实例</summary>
    
    ```
    package golang_bytes_package_example
    
    import (
    	"bytes"
    	"fmt"
    )
    
    func ExampleReaderLen() {
    	fmt.Println(bytes.NewReader([]byte("Hi!")).Len())
    	fmt.Println(bytes.NewReader([]byte("こんにちは!")).Len())
    
    	// Output:
    	// 3
    	// 16
    }
    ```
</details>


### Read

```
func (r *Reader) Read(b []byte) (n int, err error)
```

`Read` 实现了 `io.Reader` 接口。


### ReadAt

```
func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)
```

`ReadAt` 实现了 `io.ReaderAt` 接口。


### ReadByte

```
func (r *Reader) ReadByte() (byte, error)
```

`ReadByte` 实现了 `io.ByteReader` 接口


### ReadRune

```
func (r *Reader) ReadRune() (ch rune, size int, err error)
```

`ReadRune` 实现了 `io.RuneReader` 接口


### Reset

```
func (r *Reader) Reset(b []byte)
```

`Reset` 将 `Reader` 的内容重置为 `b` 字节切片。


### Seek

```
func (r *Reader) Seek(offset int64, whence int) (int64, error)
```

`Seek` 实现了 `io.Seeker` 接口


### Size

```
func (r *Reader) Size() int64
```

`Size` 返回了 `Reader` 底层切片的原始长度。`Size` 是可通过 `ReadAt` 读取的字节数。返回的值始终相同，并且不受任何其他方法调用的影响。


### UnreadByte

```
func (r *Reader) UnreadByte() error
```

`UnreadByte` 在实现 `io.ByteScanner` 接口时对 `ReadByte` 进行了补充。


### UnreadRune

```
func (r *Reader) UnreadRune() error
```

`UnreadRune` 在实现 `io.RuneScanner` 接口方面对 `ReadRune` 进行了补充。


### WriteTo

```
func (r *Reader) WriteTo(w io.Writer) (n int64, err error)
```

`WriteTo` 实现了 `io.WriterTo` 接口。
