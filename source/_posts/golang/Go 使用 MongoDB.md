---
title: Go 使用 MongoDB
date: 2020-03-08 13:11:30
tags: [Go, MongoDB]
---

* [MongoDB 安装(Docker)](#MongoDB-安装-Docker)
* [安装 MongoDB Go 驱动](#安装-MongoDB-Go-驱动)
* [使用 Go Driver 连接到 MongoDB](#使用-Go-Driver-连接到-MongoDB)
* [在 Go 里面使用 BSON 对象](#在-Go-里面使用-BSON-对象)
* [CRUD 操作](#CRUD-操作)
    * [插入文档](#插入文档)
    * [更新文档](#更新文档)
    * [查询文档](#查询文档)
    * [删除文档](#删除文档)
* [下一步](#下一步)

## MongoDB 安装(Docker)

先装个 mongo，为了省事就用 docker 了。

docker 的 daemon.json 加一个国内的源地址：

```
"registry-mirrors": [
    "http://hub-mirror.c.163.com"
]
```

然后拉取 mongo 镜像:

```
docker pull mongodb
```

启动 mongo:

```
docker run -p 27017:27017 mongo
```


## 安装 MongoDB Go 驱动

```
go get go.mongodb.org/mongo-driver
```


## 基础代码

创建 main.go 文件，并且导入 `bson`，`mongo` 和 `mongo/options` 包。

```
package main

import(
    "context"
    "fmt"
    "log"

    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

// 在后面的代码中将会使用这个 Trainer 结构体
type Trainer struct {
    Name string
    Age  int
    City string
}

func main() {
    // 
}
```


## 使用 Go Driver 连接到 MongoDB

一旦 MongoDB 的 Go 驱动被导入之后，我们就可以使用 `mongo.Connect` 函数来连接到 MongoDB。你必须传递一个 `context` 和 `options.ClientOptions` 对象给 `mongo.Connect` 函数。[options.ClientOptions](https://godoc.org/go.mongodb.org/mongo-driver/mongo/options) 被用来设置连接配置。

```
// 设置连接参数
clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")

// 连接到 MongoDB
client, err := mongo.Connect(context.TODO(), clientOptions)

if err != nil {
    log.Fatal(err)
}

// 检查连接
err = client.Ping(context.TODO(), nil)

if err != nil {
    log.Fatal(err)
}

fmt.Println("Connected to MongoDB!")
```

一旦你连接上了 MongoDB，你现在可以获取 test 数据库的 trainers 集合。

```
collection := client.Dataabse("test").Collection("trainers")
```

下面的代码将会使用 `collection` 来查询 `trainers` 集合。

如果你不在需要查询 MongoDB，就可以使用 `client.Disconnect()` 来关闭 MongoDB 连接。

```
err = client.Disconnect(context.TODO())

if err != nil {
    log.Fatal(err)
}
fmt.Println("Connection to MongoDB closed.")
```


## 在 Go 里面使用 BSON 对象

在发送查询请求到 MongoDB 之前，了解如何使用 BSON 对象是非常有必要的。在 MongoDB 中存储的 JSON 文档叫 BSON，是以二进制形式存储的。不像其他数据库以字符串形式存储 JSOn 数据，BSON 还包含了字段类型，这使得应用程序更容易可靠地处理、排序和比较。Go 的 MongoDB 驱动有两个表示 BSON 数据的类型：D 类型和 Raw 类型。

D 类型用于使用 Go 中的原始类型来构建 BSON 对象。这对于传递给 MongoDB 的命令特别有用。D 类型包括四种类型：

* D：一个 BSON 文档。

* M：无序的 map。

* A：一个 BSON 数组。

* E：D 里面的单个元素。

这里是一个使用 D 类型筛选文档的例子，这里会筛选出名字为 Alice 或者 Bob 的数据：

```
bson.D({
    "name",
    bson.D{
        {"$in": bson.A{"Alice", "Bob"}}
    }
})
```


## CRUD 操作

一旦你连接到了数据库，我们就可以开始添加或者操作数据库中的数据了。`Collection` 类型有一些方法允许你发送查询到数据库中。

### 插入文档

首先，创建一些新的 `Trainer` 结构体来插入到数据库中：

```
ash := Trainer{"Ash", 10, "Pallet Town"}
misty := Trainer{"Misty", 10, "Cerulean City"}
brock := Trainer{"Brock", 15, "Pewter" City}
```

可以使用 `collection.InsertOne()` 方法来插入单个文档：

```
insertResult, err := collection.InsertOne(context.TODO(), ash)
if err != nil {
    log.Fatal(err)
}

fmt.Println("Inserted a single document: ", insertResult.InsertedID)
```

如果我们想一次性插入多个文档，可以传递一个切片给 `collection.InsertMany` 方法：

```
trainers := []interface{}{misty, brock}

insertManyResult, err := collection.InsertMany(context.TODO(), trainers)
if err != nil {
    log.Fatal(err)
}

fmt.Println("Inserted multiple documents: ", insertManyResult.InsertedIDs)
```

### 更新文档

`collection.UpdateOne()` 方法允许你更新单个文档。它需要一个 `bson.D` 类型的参数来筛选数据库中特定的文档，然后更新它。

```
// 筛选 name 字段的值为 Ash 的记录
filter := bson.D{{"name", "Ash"}}

// 更新 age 字段，值加 1
update := bson.D{
    {"$inc", bson.D{
        {"age", 1}
    }}
}
```

这几行代码会将数据库中名字为 Ash 的文档的 age 字段的值加 1。

```
updateResult, err := collection.UpdateOne(context.TODO(), filter, update)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Matched %v documents and updated %v documents.\n", updateResult.MatchedCount, updateResult.ModifiedCount)
```


### 查询文档

如果想要查询一个文档，同样需要提供一个 filter 文档来筛选，同时需要一个接收结果的指针。为了查询单个文档，可以使用 `collection.FindOne()` 方法。这个方法会返回一条匹配上的记录并解析到我们指针指向的地方。你可以使用和上面相同的 filter 来查询 name 为 Ash 的记录。

```
// 创建一个变量用来接收解析后的结果
var result Trainer

err = collection.FindOne(context.TODO(), filter).Decode(&result)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Found a single document: %+v\n", result)
```

如果想要查询多个文档，可以使用 `collection.Find()` 方法。这个方法返回一个 `Cursor`（游标）。一个 `Cursor` （游标）可以让我们一次获取到其中一条记录。一旦一个游标遍历完毕，你应该关闭这个游标。

下面的例子中，我们在 Find 的时候同时指定了额外的一些选项，这里我们设置最多获取的记录数为 2。

```
// 这个选项会被传递给 Find 方法
findOptions := options.Find()
findOptions.SetLimit(2)

// 这是用来保存查询结果的数组
var results []*Trainer

// 传递 bson.D{{}} 作为 filter 来匹配 collection 中的所有文档
cur, err := collection.Find(context.TODO(), bson.D{{}}, findOptions)
if err != nil {
    log.Fatal(err)
}

// 查询多个文档的时候返回一个 cursor
// 迭代这个 cursor 允许我们一次解码一个文档
for cur.Next(context.TODO()) {

    // 创建一个用来接收单个文档的变量
    var elem Trainer
    err := cur.Decode(&elem)
    if err != nil {
        log.Fatal(err)
    }

    results = append(results, &elem)
}

if err := cur.Err(); err != nil {
    log.Fatal(err)
}

// 一旦结束了获取数据的操作，我们需要关闭 cursor
cur.Close(context.TODO())

fmt.Println("Found multiple documents (array of pointers): %+v\n", results)
```

### 删除文档

最后，你可以使用 `collection.DeleteOne()` 或者 `collection.DeleteMany()` 来删除文档。这里传递了 ```bson.D{{}}``` 作为 filter 参数，将会匹配集合中的所有文档。你也可以使用 `collection.Drop` 来删除整个集合。

```
deleteResult, err := collection.DeleteMany(context.TODO(), bson.D{{}})
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult.DeletedCount)
```


## 下一步

MongoDB Go 驱动的完整文档可以在 [pkg.go.dev](https://pkg.go.dev/go.mongodb.org/mongo-driver/mongo?tab=doc) 中查看。

