---
title: Go MongoDB Driver 实例
date: 2020-03-08 20:11:30
tags: [Go, MongoDB]
---

所有的 API 可以在 [Go MongoDB Driver](https://pkg.go.dev/go.mongodb.org/mongo-driver/mongo?tab=doc) 找到。

在下面的例子中，都使用下面的这个 `db()` 函数来获取 `mongo.Database` 的实例。

```
import (
	"context"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"log"
)

func db() *mongo.Database {
    clientOptions := options.Client().ApplyURL("mongodb://localhost:27017")
    client, err := mongo.Connect(context.TODO(), clientOptions)
    if err != nil {
        log.Fatal(err)
    }

    err = client.Ping(context.TODO(), nil)
    if err != nil {
        log.Fatal(err)
    }
    return client.Database("test")
}
```

## 实例一 插入单条记录

```
import (
	"context"
	"fmt"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"log"
)

func main() {
	db := db()
	coll := db.Collection("inventory_test")
	coll.Drop(context.TODO())

	// E represents a BSON element for a D. It is usually used inside a D.
	// type E struct {
	//	 Key   string
	//	 Value interface{}
	// }
	result, err := coll.InsertOne(
		context.TODO(),
		// bson.D 是一个 []bson.E 类型
		bson.D{
			{"item", "canvas"},  // 这是一个 bson.E 类型元素
			{"qty", 100},
			{"tags", bson.A{"cotton"}},
			{"size", bson.D{
				{"h", 28},
				{"w", 35.5},
				{"uom", "cm"},
			}},
		},
	)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Inserted ID is %+v\n", result.InsertedID)
	
	// 我们会在 MongoDB 中看到如下内容：
	// {
	//     "_id": ObjectId("5e64e556478fb77672d8b5b5"),
	//     "item": "canvas",
	//     "qty": NumberInt("100"),
	//     "tags": [
	//         "cotton"
	//     ],
	//     "size": {
	//         "h": NumberInt("28"),
	//         "w": 35.5,
	//         "uom": "cm"
	//     }
	// }
}
```

## 实例二 查询单条记录

```
import (
	"context"
	"fmt"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"log"
)

type Size struct {
	H   int
	W   float64
	Uom string
}

type InventoryTest struct {
	ID   string `bson:"_id"`
	Item string
	Qty  int
	Tags []string
	Size Size
}

func main() {
	db := db()
	coll := db.Collection("inventory_test")

	cursor, err := coll.Find(
		context.TODO(),
		bson.D{
			{"item", "canvas"},
		},
	)
	if err != nil {
		panic(err)
	}

	var inventoryTest InventoryTest
    // 获取游标的下一条记录
	cursor.Next(context.TODO())
	err = cursor.Decode(&inventoryTest)
	if err != nil {
		panic(err)
	}
	fmt.Printf("inventoryTest is %+v\n", inventoryTest)

    // inventoryTest is {ID:5e64ea4919abb0178c2787f8 Item:canvas Qty:100 Tags:[cotton] Size:{H:28 W:35.5 Uom:cm}}
}
```


## 实例三 插入多条记录

```
import (
	"context"
	"fmt"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"log"
)

func main() {
	db := db()
	coll := db.Collection("inventory_test")

	result, err := coll.InsertMany(
		context.TODO(),
		[]interface{}{
			bson.D{
				{"item", "journal"},
				{"qty", int32(25)},
				{"tags", bson.A{"blank", "red"}},
				{"size", bson.D{
					{"h", 14},
					{"w", 21},
					{"uom", "cm"},
				}},
			},
			bson.D{
				{"item", "mat"},
				{"qty", int32(25)},
				{"tags", bson.A{"gray"}},
				{"size", bson.D{
					{"h", 27.9},
					{"w", 35.5},
					{"uom", "cm"},
				}},
			},
			bson.D{
				{"item", "mousepad"},
				{"qty", 25},
				{"tags", bson.A{"gel", "blue"}},
				{"size", bson.D{
					{"h", 19},
					{"w", 22.85},
					{"uom", "cm"},
				}},
			},
		},
	)
	if err != nil {
		panic(err)
	}
	fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)

	// InsertedIDs: [ObjectID("5e64ee5bacce2998dc112460") ObjectID("5e64ee5bacce2998dc112461") ObjectID("5e64ee5bacce2998dc112462")]
}
```


## 实例四 查询顶层字段

```
package main

import (
	"context"
	"fmt"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"log"
)

func db() *mongo.Database {
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}
	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}
	return client.Database("test")
}

type Size struct {
	H   int
	W   float64
	Uom string
}

type InventoryTest struct {
	ID   string `bson:"_id"`
	Item string
	Qty  int
	Tags []string
	Size Size
}

func main() {
	db := db()
	coll := db.Collection("inventory_query_top")

	err := coll.Drop(context.TODO())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 6
		docs := []interface{}{
			bson.D{
				{"item", "journal"},
				{"qty", 25},
				{"size", bson.D{
					{"h", 14},
					{"w", 21},
					{"uom", "cm"},
				}},
				{"status", "A"},
			},
			bson.D{
				{"item", "notebook"},
				{"qty", 50},
				{"size", bson.D{
					{"h", 8.5},
					{"w", 11},
					{"uom", "in"},
				}},
				{"status", "A"},
			},
			bson.D{
				{"item", "paper"},
				{"qty", 100},
				{"size", bson.D{
					{"h", 8.5},
					{"w", 11},
					{"uom", "in"},
				}},
				{"status", "D"},
			},
			bson.D{
				{"item", "planner"},
				{"qty", 75},
				{"size", bson.D{
					{"h", 22.85},
					{"w", 30},
					{"uom", "cm"},
				}},
				{"status", "D"},
			},
			bson.D{
				{"item", "postcard"},
				{"qty", 45},
				{"size", bson.D{
					{"h", 10},
					{"w", 15.25},
					{"uom", "cm"},
				}},
				{"status", "A"},
			},
		}

		result, err := coll.InsertMany(context.TODO(), docs)
		if err != nil {
			panic(err)
		}
		fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)
	}

	{
		// Start Example 7
		cursor, _ := coll.Find(
			context.TODO(),
			bson.D{},
		)
		length := 0
		for cursor.Next(context.TODO()) {
			length++
		}
		// 5
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 8
		cursor, _ := coll.Find(
			context.TODO(),
			bson.D{
				{"status", "D"},
			},
		)
		length := 0
		for cursor.Next(context.TODO()) {
			length++
		}
		// 2
		fmt.Printf("length: %+v\n", length)
	}

	{
		// Start Example 9
		cursor, _ := coll.Find(
			context.TODO(),
			bson.D{
				{"status", bson.D{{"$in", bson.A{"A", "D"}}}},
			},
		)
		length := 0
		for cursor.Next(context.TODO()) {
			length++
		}
		// 5
		fmt.Printf("length: %+v\n", length)
	}

	{
		// Start Example 10
		cursor, _ := coll.Find(
			context.TODO(),
			bson.D{
				{"status", "A"},
				{"qty", bson.D{{"$lt", 30}}},
			},
		)
		length := 0
		for cursor.Next(context.TODO()) {
			length++
		}
		// 1
		fmt.Printf("length: %+v\n", length)
	}

	{
		// Start Example 11
		cursor, _ := coll.Find(
			context.TODO(),
			bson.D{
				{"$or", bson.A{
					bson.D{{"status", "A"}},
					bson.D{{"qty", bson.D{{"$lt", 30}}}},
				}},
			},
		)
		length := 0
		for cursor.Next(context.TODO()) {
			length++
		}
		// 3
		fmt.Printf("length: %+v\n", length)
	}

	{
		// Start Example 12
		cursor, _ := coll.Find(
			context.TODO(),
			bson.D{
				{"status", "A"},
				{"$or", bson.A{
					bson.D{{"qty", bson.D{{"$lt", 30}}}},
					bson.D{{"item", primitive.Regex{Pattern: "^p", Options: ""}}},
				}},
			},
		)
		length := 0
		for cursor.Next(context.TODO()) {
			length++
		}
		// 2
		fmt.Printf("length: %+v\n", length)
	}
}
```