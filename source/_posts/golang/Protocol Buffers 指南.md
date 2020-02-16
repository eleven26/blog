---
title: Protocol Buffers(proto3) 指南
date: 2020-02-16 19:25:30
tags: [ProtoBuf]
---

这个指南描述了如何使用 protocol buffer 语言来组织你的 protocol buffer 数据，包括 .proto 文件语法和如何从你的 .proto 文件生成数据访问类。


## 定义一个消息类型

首先我们来看一个简单的例子。假设你要定义一个请求消息格式，每个搜索请求有一个查询字符串，你想要查询特定页数的数据，并且指定每一页的大小。这里是定义消息类型的 .proto 文件。

```
syntax = "proto3";

message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}
```

* 第一行表示我们使用的是 proto3 的语法；如果不指定版本，protobuf 编译器会当作 proto2 处理。这行必须处于文件开头。

* `SearchRequest` 消息类型定义了三个字段，每个字段有一个名字和类型。


## 指定字段类型

在上面的例子中，所有字段都是标量类型：两个整数（page_number 和 result_per_page） 和一个字符串(query)。然而，你也可以组合不同类型的字段到一个字段中，作为一个新的消息类型。


## 分配字段编号

正如你所见，消息中每个字段都有一个唯一的数字。这些字段数字被用来作为二进制消息中的唯一标识，同时一旦消息类型被使用就不应该再修改这个唯一标识。请注意，范围为 1 到 15 的字段号需要一个字节来编码，
包括字段号和字段的类型。16 到 2047 之间的字段编号占用两个字节。因此，应该为经常出现的消息元素保留数字 1 到 15。请记住为将来可能添加的频繁出现的元素流出一些空间。


## 指定字段规则

消息字段可以是以下格式之一：

* 单数：格式正确的消息可以包含零个或一个此字段（但不能超过一个）。这是 proto3 语法的默认字段规则。

* repeated：在格式正确的消息中，此字段可以重复任意次（包括零次）。重复值的顺序将保留。

在 proto3 中，repeated 标量数字类型的字段 packed 默认情况下使用编码。


## 添加更多消息类型

可以在一个 .proto 文件中定义多种消息类型。如果要定义多个相关消息，这很有用，例如，如果要定义你的 SearchResponse 消息类型相对应的回复消息格式，可以将其添加到相同的消息中 .proto:

```
message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}

message SearchResponse {}
```


## 添加注释

要将注释添加到 .proto 文件中，请使用 C/C++ 样式 // 和 /* */ 语法。

```
/* SearchRequest represents a search query, with pagination options to
 indicate which results to include in the response. */
 
message SearchRequest {
    string query = 1;
    int32 page_number = 2; // Which page number do we want ?
    int32 result_per_page = 3; // Number of results to return per page.
}
```


## 保留字段

如果你完全删除一个字段或将其注释掉来更新消息类型，则将来的用户在自己对该类型进行更新时可以重用该字段号。如果他们以后加载相同版本的旧版本，可能会导致严重的问题 .proto，
包括数据损坏，隐私错误等。确保不会发生这种情况的一种方法是，将已删除字段的字段编号（和/或名称，也可能导致 JSON 序列化的问题）指定为 reserved。如果将来有任何用户尝试
使用这些字段标识符，则 proto buf 编译器会报错。

```
message Foo {
    rserved 2, 15, 9 to 11;
    reserved "foo", "bar";
}
```

请注意，你不能在同 reversed 一条语句中混用字段名称和字段编号。


## 你产生了什么 .proto ?

编译器会以你选择的语言生成代码，你将需要使用文件中描述的消息类型，包括获取和设置字段值，将消息序列化为输出流，并从输入流中解析消息。

* 对于 C++，编译器从每个 .proto 生成一个 .h 和 .cc 文件，并为文件中描述的每种消息类型提供一个类。

* 对于 Java，编译器将生成一个 .java 文件，其中包含每种消息类型的类以及 Builder 用于创建消息实例的特殊类。

* Python 稍有不同 - Python 编译器会生成一个模块，其中包含每种消息类型的静态描述符，.proto 然后将该模块与元类一起使用，以在运行时创建必要的 Python 数据访问类。

* 对于 Go，编译器 .pb.go 将为文件中的每种消息类型生成一个具有相应类型的文件。

* 对于 Ruby，编译器将 .rb 使用包含你的消息类型的 Ruby 模块生成文件。



## 默认值

解析消息时，如果编码的消息不包含特定的单数元素，则已解析对象中的相应字段将设置为该字段的默认值。这些默认值是特定于类型的。

* 对于字符串，默认值为空字符串

* 对于字节，默认值为空字节

* 对于布尔值，默认值为 false

* 对于数字类型，默认值为 0

* 对于枚举，默认值为第一个定义的枚举值，必须为 0

* 对于消息字段，未设置该字段。它的确切值取决于语言。

重复字段的默认值为空。

请注意，对于标量消息字段，一旦解析了一条消息，就无法告诉该字段是显式设置为默认值（例如，是否将 boolean 设置为 false）还是根本没有设置：你应该牢记这一点定义消息类型时。


## 枚举

定义消息类型时，你可能希望其字段之一仅具有一个预定义的值列表之一。例如，假设你想为每个 SearchRequest 添加一个 corpus 字段，这个字段可以是  UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS。你可以简单地添加一个枚举类型到 SearchRequest 中。

在如下例子中，我们添加了一个名为 Corpus 的枚举类型，包含了所有可能的值。

```
message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
    enum Corpus {
        UNIVERSAL = 0;
        WEB = 1;
        IMAGES = 2;
        LOCAL = 3;
        NEWS = 4;
        PRODUCTS = 5;
        VIDEO = 6;
    }
    Corpus corpus = 4;
}
```

正如你所看到的，枚举类型字段 Corpus 的第一个常量是 0，每个枚举类型必须包含一个映射为零的常量作为其第一个元素。这是因为：

* 必须有一个零值，以便我们可以使用 0 作为数字默认值

* 零值必须是第一个元素，以便与 proto2 语义兼容，其中第一个枚举值始终是默认值。

你可以通过将相同的值分配给不同枚举常量来定义别名。为此，你需要将 allow_alias 选项设置为 true，否则协议别名会在找到别名时生成一条错误消息。

```
enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
}
enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    // RUNNING = 1; // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

枚举常量必须在 32 位整数范围内。


## 保留值

如果通过完全删除枚举条目或将其注释掉来更新枚举类型，则将来的用户在自己对类型进行更新时可以重用数值。如果他们以后加载相同版本的旧版本，可能会导致严重的问题，
包括数据损坏，隐私错误等。确保不会发生这种情况的一种方法是，将删除的条目的数字值指定为 reversed。如果将来有任何用户尝试使用这些标识符，
则 proto buf 编译器会报错，你可以使用 max 关键字指定保留的数值范围达到最大可能值。

```
enum Foo {
    reserved 2, 15, 9 to 11, 40 to max;
    reserved "FOO", "BAR";
}
```

请注意，你不能在同 reversed 一条语句中混合使用字段名和数字值。


## 使用其他消息类型

你可以使用其他消息类型作为字段类型。例如，假设你想包括 Result 每个消息的 SearchResponse 消息 - 要做到这一点，你可以定义一个 Result 在同一个消息类型 .proto，然后指定类型的字段为 SearchResponse。

```
message SearchResponse {
    repeated Result results = 1;
}

message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
}
```


### 导入定义

在上面的示例中，Result 消息类型与以下文件定义在同一文件中 SearchResponse - 如果要用作字段的消息类型已在另一个 .proto 文件中定义，该怎么办？

你可以 .proto 通过导入其他文件来使用它们的定义。要导入另外一个 .proto 的定义，请在文件顶部添加一个 import 语句；

```
import "myproject/other_protos.proro"
```

默认情况下，你只能使用直接导入 .proto 文件中的定义。但是，有时你可能需要将 .proto 文件移到新位置。 

```
// new.proto
// All definitions are moved here
```

```
// old.proto
// This is the proto that all clients are importing
import public "new.proto";
import "other.proto"
```

```
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

协议编译器使用 -I/--proto_path 标志在协议编译器命令行上指定的一组目录中搜索导入的文件。


## 嵌套类型

你可以在其他消息类型内定义和使用消息类型，如以下示例所示 - 在此处，Result 消息是在消息内定义的 SearchResponse：

```
message SearcResponse {
    message Result {
        string url = 1;
        string title = 2;
        repeated string snippets = 3;
    }
    repeated Result results = 1;
}
```

如果要在其父消息类型之外重用此消息类型，则将其称为：Parent.Type
```
message SomeOtherMessag {
    SearchResponse.Result result = 1;
}
```

你可以根据深度嵌套消息：
```
message Outer {
    message MiddleAA {
        message Inner {
            int64 ival = 1;
            bool booly = 2;
        }
    }
    message MiddleBB {
        message Inner {
            int32 ival = 1;
            bool booly = 2;
        }
    }
}
```


## 更新消息类型

如果现有的消息类型不再满足你的所有需求，但是你仍然希望使用以旧格式创建的代码，请不要担心！在不破坏任何现有代码的情况下更新消息类型非常简单。只要记住以下规则：

* 不要更改任何现有字段的字段编号。

* 如果添加新字段，则仍可以使用新生成的代码来解析使用 "旧" 消息格式通过代码序列化的任何消息。你应该记住这些元素的默认值，以便新代码可以与旧代码生成的消息正确交互。
同样，由新代码创建的消息可以由旧代码解析：旧的二进制文件在解析时只会忽略新字段。

* 只要在更新的消息类型中不再使用字段号，就可以删除字段。你可能想要重命名该字段，或者添加前缀 "OBSOLETE_"，或将字段编号保留，以便将来的用户不会意外地重用该编号。

* int32，uint32，int64，uint64 和 bool 都是兼容的。


## 未知字段

未知字段是格式正确的 proto buf 序列化数据，表示解析器无法识别的字段。例如，当旧二进制文件使用新字段解析新二进制文件发送的数据时，这些新字段将成为旧二进制文件中的未知字段。

最初，proto3 消息在解析过程中总是丢弃未知字段，但是在版本 3.5 中，我们重新引入了保留未知字段以匹配 proto2 行为的功能。

