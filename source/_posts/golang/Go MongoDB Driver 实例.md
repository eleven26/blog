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


## 实例五 查询嵌套的字段

```
func main() {
	db := db()
	coll := db.Collection("inventory_query_embedded")

	err := coll.Drop(context.TODO())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 14
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
			},
		}

		result, _ := coll.InsertMany(context.Background(), docs)
		fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)
	}

	{
		// Start Example 15
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"size", bson.D{
					{"h", 14},
					{"w", 21},
					{"uom", "cm"},
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 16
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"size", bson.D{
					{"w", 21},
					{"h", 14},
					{"uom", "cm"},
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 17
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{{"size.uom", "in"}},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 18
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"size.h", bson.D{
					{"$lt", 15},
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 19
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"size.h", bson.D{
					{"$lt", 15},
				}},
				{"size.uom", "in"},
				{"status", "D"},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}
}
```


## 实例六 查询数组

```
func main() {
	db := db()
	coll := db.Collection("inventory_query_array")

	err := coll.Drop(context.TODO())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 20
		docs := []interface{}{
			bson.D{
				{"item", "journal"},
				{"qty", 25},
				{"tags", bson.A{"blank", "red"}},
				{"dim_cm", bson.A{14, 21}},
			},
			bson.D{
				{"item", "notebook"},
				{"qty", 50},
				{"tags", bson.A{"red", "blank"}},
				{"dim_cm", bson.A{14, 21}},
			},
			bson.D{
				{"item", "paper"},
				{"qty", 100},
				{"tags", bson.A{"red", "blank", "plain"}},
				{"dim_cm", bson.A{14, 21}},
			},
			bson.D{
				{"item", "planner"},
				{"qty", 75},
				{"tags", bson.A{"blank", "red"}},
				{"dim_cm", bson.A{22.85, 30}},
			},
			bson.D{
				{"item", "postcard"},
				{"qty", 45},
				{"tags", bson.A{"blue"}},
				{"dim_cm", bson.A{10, 15.25}},
			},
		}

		result, _ := coll.InsertMany(context.Background(), docs)
		fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)
	}

	{
		// Start Example 21
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{{"tags", bson.A{"red", "blank"}}},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 22
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"tags", bson.D{{"$all", bson.A{"red", "blank"}}}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 23
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"tags", "red"},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 24
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"dim_cm", bson.D{
					{"$gt", 25},
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 25
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"dim_cm", bson.D{
					{"$gt", 15},
					{"$lt", 20},
				}},
			},
		)

		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 26
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"dim_cm", bson.D{
					{"$elemMatch", bson.D{
						{"$gt", 22},
						{"$lt", 30},
					}},
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 27
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"dim_cm.1", bson.D{
					{"$gt", 25},
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 28
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"tags", bson.D{
					{"$size", 3},
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}
}
```


## 实例七 查询嵌套文档

```
func main() {
	db := db()
	coll := db.Collection("inventory_query_embedded")

	err := coll.Drop(context.TODO())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 29
		docs := []interface{}{
			bson.D{
				{"item", "journal"},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "A"},
						{"qty", 5},
					},
					bson.D{
						{"warehourse", "C"},
						{"qty", 15},
					},
				}},
			},
			bson.D{
				{"item", "notebook"},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "C"},
						{"qty", 5},
					},
				}},
			},
			bson.D{
				{"item", "paper"},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "A"},
						{"qty", 60},
					},
					bson.D{
						{"warehouse", "B"},
						{"qty", 15},
					},
				}},
			},
			bson.D{
				{"item", "planner"},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "A"},
						{"qty", 40},
					},
					bson.D{
						{"warehouse", "B"},
						{"qty", 5},
					},
				}},
			},
			bson.D{
				{"item", "postcard"},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "B"},
						{"qty", 15},
					},
					bson.D{
						{"warehouse", "C"},
						{"qty", 35},
					},
				}},
			},
		}

		result, _ := coll.InsertMany(context.Background(), docs)
		fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)
	}

	{
		// Start Example 30
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock", bson.D{
					{"warehouse", "A"},
					{"qty", 5},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length) // 1
	}

	{
		// Start Example 31
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock", bson.D{
					{"qty", 5},
					{"warehouse", "A"},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length) // 0
	}

	{
		// Start Example 32
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock.0.qty", bson.D{
					{"$lte", 20},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length) // 3
	}

	{
		// Start Example 33
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock.qty", bson.D{
					{"$lte", 20},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length) // 5
	}

	{
		// Start Example 34
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock", bson.D{
					{"$elemMatch", bson.D{
						{"qty", 5},
						{"warehouse", "A"},
					}},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length) // 1
	}

	{
		// Start Example 35
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock", bson.D{
					{"$elemMatch", bson.D{
						{"qty", bson.D{
							{"$gt", 10},
							{"$lte", 20},
						}},
					}},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length) // 3
	}

	{
		// Start Example 36
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock.qty", bson.D{
					{"$gt", 10},
					{"$lte", 20},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length) // 4
	}

	{
		// Start Example 37
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"instock.qty", 5},
				{"instock.warehouse", "A"},
			})
		type Stock struct {
			Warehouse string
			Qty       int
		}
		type Data struct {
			Item    string
			Instock []Stock
		}
		length := 0
		for cursor.Next(context.Background()) {
			length++
			var d Data
			cursor.Decode(&d)
			// {Item:journal Instock:[{Warehouse:A Qty:5} {Warehouse: Qty:15}]}
			// {Item:planner Instock:[{Warehouse:A Qty:40} {Warehouse:B Qty:5}]}
			fmt.Printf("%+v\n", d)
		}
		fmt.Printf("length: %v\n", length) // 2
	}
}
```


## 实例八 查询为 null 或缺失的字段

```
func main() {
	db := db()
	coll := db.Collection("inventory_query_null_missing")

	err := coll.Drop(context.Background())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 38
		docs := []interface{}{
			bson.D{
				{"_id", 1},
				{"item", nil},
			},
			bson.D{
				{"_id", 2},
			},
		}
		result, _ := coll.InsertMany(context.Background(), docs)
		fmt.Printf("InsertedIDs: %v\n", result.InsertedIDs)
	}

	{
		// Start Example 39
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"item", nil},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 40
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"item", bson.D{
					{"$type", 10}, // 10 等同于 "null", https://www.runoob.com/mongodb/mongodb-operators-type.html
				}},
			},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}

	{
		// Start Example 41
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"item", bson.D{
					{"$exists", false},
				}},
			})
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %v\n", length)
	}
}
```


## 实例九 Projection


```
func containsKey(doc bson.Raw, key ...string) bool {
	_, err := doc.LookupErr(key...)
	if err != nil {
		return false
	}
	return true
}

func main() {
	db := db()
	coll := db.Collection("inventory_project")

	err := coll.Drop(context.Background())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 42
		docs := []interface{}{
			bson.D{
				{"item", "journal"},
				{"status", "A"},
				{"size", bson.D{
					{"h", 14},
					{"w", 21},
					{"uom", "cm"},
				}},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "A"},
						{"qty", 5},
					},
				}},
			},
			bson.D{
				{"item", "notebook"},
				{"status", "A"},
				{"size", bson.D{
					{"h", 8.5},
					{"w", 11},
					{"uom", "in"},
				}},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "EC"},
						{"qty", 5},
					},
				}},
			},
			bson.D{
				{"item", "paper"},
				{"status", "D"},
				{"size", bson.D{
					{"h", 8.5},
					{"w", 11},
					{"uom", "in"},
				}},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "A"},
						{"qty", 60},
					},
				}},
			},
			bson.D{
				{"item", "postcard"},
				{"status", "A"},
				{"size", bson.D{
					{"h", 10},
					{"w", 15.25},
					{"uom", "cm"},
				}},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "B"},
						{"qty", 15},
					},
					bson.D{
						{"warehouse", "EC"},
						{"qty", 35},
					},
				}},
			},
		}
		result, _ := coll.InsertMany(context.Background(), docs)
		fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)
	}

	{
		// Start Example 43
		cursor, _ := coll.Find(
			context.Background(),
			bson.D{{"status", "A"}},
		)
		length := 0
		for cursor.Next(context.Background()) {
			length++
		}
		fmt.Printf("length: %+v\n", length)
	}

	{
		// Start Example 44
		projection := bson.D{
			{"item", 1},
			{"status", 1},
		}

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
			options.Find().SetProjection(projection),
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contain _id: %v\n", containsKey(doc, "_id"))
			fmt.Printf("contain item: %v\n", containsKey(doc, "item"))
			fmt.Printf("contain status: %v\n", containsKey(doc, "status"))
			fmt.Printf("contain size: %v\n", containsKey(doc, "size"))
			fmt.Printf("contain instock: %v\n", containsKey(doc, "instock"))

			break
		}
	}

	{
		// Start Example 45
		projection := bson.D{
			{"item", 1},
			{"status", 1},
			{"_id", 0},
		}

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
			options.Find().SetProjection(projection),
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contain _id: %v\n", containsKey(doc, "_id"))
			fmt.Printf("contain item: %v\n", containsKey(doc, "item"))
			fmt.Printf("contain status: %v\n", containsKey(doc, "status"))
			fmt.Printf("contain size: %v\n", containsKey(doc, "size"))
			fmt.Printf("contain instock: %v\n", containsKey(doc, "instock"))

			break
		}
	}

	{
		// Start Example 46
		projection := bson.D{
			{"status", 0},
			{"instock", 0},
		}

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
			options.Find().SetProjection(projection),
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contain _id: %v\n", containsKey(doc, "_id"))
			fmt.Printf("contain item: %v\n", containsKey(doc, "item"))
			fmt.Printf("contain status: %v\n", containsKey(doc, "status"))
			fmt.Printf("contain size: %v\n", containsKey(doc, "size"))
			fmt.Printf("contain instock: %v\n", containsKey(doc, "instock"))

			break
		}
	}

	{
		// Start Example 47
		projection := bson.D{
			{"item", 1},
			{"status", 1},
			{"size.uom", 1},
		}

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
			options.Find().SetProjection(projection),
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contain _id: %v\n", containsKey(doc, "_id"))
			fmt.Printf("contain item: %v\n", containsKey(doc, "item"))
			fmt.Printf("contain status: %v\n", containsKey(doc, "status"))
			fmt.Printf("contain size: %v\n", containsKey(doc, "size"))
			fmt.Printf("contain instock: %v\n", containsKey(doc, "instock"))

			fmt.Printf("contain size.uom: %v\n", containsKey(doc, "size", "uom"))
			fmt.Printf("contain size.h: %v\n", containsKey(doc, "size", "h"))
			fmt.Printf("contain size.w: %v\n", containsKey(doc, "size", "w"))

			break
		}
	}

	{
		// Start Example 48
		projection := bson.D{
			{"size.uom", 0},
		}

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
			options.Find().SetProjection(projection),
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contain _id: %v\n", containsKey(doc, "_id"))
			fmt.Printf("contain item: %v\n", containsKey(doc, "item"))
			fmt.Printf("contain status: %v\n", containsKey(doc, "status"))
			fmt.Printf("contain size: %v\n", containsKey(doc, "size"))
			fmt.Printf("contain instock: %v\n", containsKey(doc, "instock"))

			fmt.Printf("contain size.uom: %v\n", containsKey(doc, "size", "uom"))
			fmt.Printf("contain size.h: %v\n", containsKey(doc, "size", "h"))
			fmt.Printf("contain size.w: %v\n", containsKey(doc, "size", "w"))

			break
		}
	}

	{
		// Start Example 49
		projection := bson.D{
			{"item", 1},
			{"status", 1},
			{"instock.qty", 1},
		}

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
			options.Find().SetProjection(projection),
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contain _id: %v\n", containsKey(doc, "_id"))
			fmt.Printf("contain item: %v\n", containsKey(doc, "item"))
			fmt.Printf("contain status: %v\n", containsKey(doc, "status"))
			fmt.Printf("contain size: %v\n", containsKey(doc, "size"))
			fmt.Printf("contain instock: %v\n", containsKey(doc, "instock"))

			instock, _ := doc.LookupErr("instock")
			vals, _ := instock.Array().Values()

			for _, val := range vals {
				subdoc := val.Document()
				elems, _ := subdoc.Elements()
				fmt.Printf("length of elems: %+v\n", len(elems))
			}

			break
		}
	}

	{
		// Start Example 50
		projection := bson.D{
			{"item", 1},
			{"status", 1},
			{"instock", bson.D{
				{"$slice", -1},
			}},
		}

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
			options.Find().SetProjection(projection),
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contain _id: %v\n", containsKey(doc, "_id"))
			fmt.Printf("contain item: %v\n", containsKey(doc, "item"))
			fmt.Printf("contain status: %v\n", containsKey(doc, "status"))
			fmt.Printf("contain size: %v\n", containsKey(doc, "size"))
			fmt.Printf("contain instock: %v\n", containsKey(doc, "instock"))

			break
		}
	}
}
```


## 实例十 更新

```
func main() {
	db := db()
	coll := db.Collection("inventory_update")

	err := coll.Drop(context.Background())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 51
		docs := []interface{}{
			bson.D{
				{"item", "canvas"},
				{"qty", 100},
				{"size", bson.D{
					{"h", 28},
					{"w", 35.5},
					{"uom", "cm"},
				}},
				{"status", "A"},
			},
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
				{"item", "mat"},
				{"qty", 85},
				{"size", bson.D{
					{"h", 27.9},
					{"w", 35.5},
					{"uom", "cm"},
				}},
				{"status", "A"},
			},
			bson.D{
				{"item", "mousepad"},
				{"qty", 25},
				{"size", bson.D{
					{"h", 19},
					{"w", 22.85},
					{"uom", "in"},
				}},
				{"status", "P"},
			},
			bson.D{
				{"item", "notebook"},
				{"qty", 50},
				{"size", bson.D{
					{"h", 8.5},
					{"w", 11},
					{"uom", "in"},
				}},
				{"status", "P"},
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
			bson.D{
				{"item", "sketchbook"},
				{"qty", 80},
				{"size", bson.D{
					{"h", 14},
					{"w", 21},
					{"uom", "cm"},
				}},
				{"status", "A"},
			},
			bson.D{
				{"item", "sketch pad"},
				{"qty", 95},
				{"size", bson.D{
					{"h", 22.85},
					{"w", 30.5},
					{"uom", "cm"},
				}},
				{"status", "A"},
			},
		}

		result, _ := coll.InsertMany(context.Background(), docs)
		fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)
	}

	{
		// Start Example 52
		result, _ := coll.UpdateOne(
			context.Background(),
			bson.D{
				{"item", "paper"},
			},
			bson.D{
				{"$set", bson.D{
					{"size.uom", "cm"},
					{"status", "{"},
				}},
				{"$currentDate", bson.D{
					{"lastModified", true},
				}},
			},
		)
		fmt.Printf("result.MatchedCount: %+v\n", result.MatchedCount)
		fmt.Printf("result.ModifiedCount: %+v\n", result.ModifiedCount)

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"item", "paper"},
			},
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			uom, _ := doc.LookupErr("size", "uom")
			fmt.Printf("uom.StringValue() = %+v\n", uom.StringValue())

			status, _ := doc.LookupErr("status")
			fmt.Printf("status.StringValue() = %+v\n", status.StringValue())

			fmt.Printf("contain lastModified: %+v\n", containsKey(doc, "lastModified"))
		}
	}

	{
		// Start Example 53
		result, _ := coll.UpdateMany(
			context.Background(),
			bson.D{
				{"qty", bson.D{
					{"$lt", 50},
				}},
			},
			bson.D{
				{"$set", bson.D{
					{"size.uom", "cm"},
					{"status", "P"},
				}},
				{"$currentDate", bson.D{
					{"lastModified", true},
				}},
			},
		)

		fmt.Printf("result.MatchedCount: %+v\n", result.MatchedCount)
		fmt.Printf("result.ModifiedCount: %+v\n", result.ModifiedCount)

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"qty", bson.D{
					{"$lt", 50},
				}},
			},
		)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			uom, _ := doc.LookupErr("size", "uom")
			fmt.Printf("uom.StringValue(): %+v\n", uom.StringValue())

			status, _ := doc.LookupErr("status")
			fmt.Printf("status.StringValue(): %+v\n", status.StringValue())

			fmt.Printf("contain lastModified: %+v\n", containsKey(doc, "lastModified"))
		}
	}

	{
		// Start Example 54
		result, _ := coll.ReplaceOne(
			context.Background(),
			bson.D{
				{"item", "paper"},
			},
			bson.D{
				{"item", "paper"},
				{"instock", bson.A{
					bson.D{
						{"warehouse", "A"},
						{"qty", 60},
					},
					bson.D{
						{"warehouse", "B"},
						{"qty", 40},
					},
				}},
			},
		)

		fmt.Printf("MatchedCount: %+v\n", result.MatchedCount)
		fmt.Printf("ModifiedCount: %+v\n", result.ModifiedCount)

		cursor, _ := coll.Find(
			context.Background(),
			bson.D{
				{"item", "paper"},
			},
			)

		for cursor.Next(context.Background()) {
			doc := cursor.Current

			fmt.Printf("contains _id: %+v\n", containsKey(doc, "_id"))
			fmt.Printf("contains item: %+v\n", containsKey(doc, "item"))
			fmt.Printf("contain instock: %+v\n", containsKey(doc, "instock"))

			instock, _ := doc.LookupErr("instock")
			vals, _ := instock.Array().Values()
			fmt.Printf("length of vals is: %+v\n", vals)
		}
	}
}
```


## 实例十一 删除

```
func main() {
	db := db()
	coll := db.Collection("inventory_delete")

	err := coll.Drop(context.Background())
	if err != nil {
		panic(err)
	}

	{
		// Start Example 55
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
				{"status", "P"},
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
		result, _ := coll.InsertMany(context.Background(), docs)
		fmt.Printf("InsertedIDs: %+v\n", result.InsertedIDs)
	}

	{
		// Start Example 57
		result, _ := coll.DeleteMany(
			context.Background(),
			bson.D{
				{"status", "A"},
			},
		)
		fmt.Printf("Deleted count: %+v\n", result.DeletedCount)
	}

	{
		// Start Example 58
		result, _ := coll.DeleteOne(
			context.Background(),
			bson.D{
				{"status", "D"},
			},
		)
		fmt.Printf("Deleted count: %+v\n", result.DeletedCount)
	}

	{
		// Start Example 59
		result, _ := coll.DeleteMany(context.Background(), bson.D{})
		fmt.Printf("Deleted count: %+v\n", result.DeletedCount)
	}
}
```


## 实例十二 事务

```
var log = logger.New(logger.DebugLevel)

func main() {
	db := db()
	coll := db.Collection("inventory_delete")

	err := coll.Drop(context.Background())
	if err != nil {
		panic(err)
	}

	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}

	employees := client.Database("hr").Collection("employees")
	events := client.Database("reporting").Collection("events")

	err = client.UseSession(context.Background(), func(sctx mongo.SessionContext) error {
		err := sctx.StartTransaction(options.Transaction().
			SetReadConcern(readconcern.Snapshot()).
			SetWriteConcern(writeconcern.New(writeconcern.WMajority())),
			)
		if err != nil {
			return err
		}

		_, err = employees.UpdateOne(sctx, bson.D{{"employee", 3}}, bson.D{{"$set", bson.D{{"status", "Inactive"}}}})
		if err != nil {
			sctx.AbortTransaction(sctx)
			log.Printf("caught exception during transaction, aborting.")
			return err
		}

		_, err = events.InsertOne(sctx, bson.D{{"employee", 3}, {"status", bson.D{{"new", "Inactive"}, {"old", "active"}}}})
		if err != nil {
			sctx.AbortTransaction(sctx)
			log.Printf("caught exception during transaction, aborting")
			return err
		}
		
		for {
			err = sctx.CommitTransaction(sctx)
			switch e := err.(type) {
			case nil:
				return nil
			case mongo.CommandError:
				if e.HasErrorLabel("UnknownTransactionCommitResult") {
					log.Printf("UnknownTransactionCommitResult, retrying commit operation...")
					continue
				}
				log.Printf("Error during commit...")
				return e
			default:
				log.Printf("Error during commit...")
				return e
			}
		}
	})
}
```


## 实例十三 事务 commit 重试

```
// RunTransactionWithRetry is an example function demonstrating transaction retry logic.
func RunTransactionWithRetry(sctx mongo.SessionContext, txnFn func(sessionContext mongo.SessionContext) error) error {
	for {
		err := txnFn(sctx) // Performs transaction.
		if err == nil {
			return nil
		}

		log.Printf("Transaction aborted. Caught exception during transaction.\n")

		// If transient error, retry the whole transaction
		if cmdErr, ok := err.(mongo.CommandError); ok && cmdErr.HasErrorLabel("TransientTransactionError") {
			log.Printf("TransientTransactionError, retrying transaction....")
			continue
		}
		return err
	}
}

// CommitWithRetry is an example function demonstrating transaction commit with retry logic.
func CommitWithRetry(sctx mongo.SessionContext) error {
	for {
		err := sctx.CommitTransaction(sctx)
		switch e := err.(type) {
		case nil:
			log.Printf("Transaction committed.")
			return nil
		case mongo.CommandError:
			// Can retry commit
			if e.HasErrorLabel("UnknownTransactionCommitResult") {
				log.Printf("UnknownTransactionCommitResult, retrying commit operation...")
				continue
			}
			log.Printf("Error during commit...")
			return e
		default:
			log.Printf("Error during commit...")
			return e
		}
	}
}

// TransactionsExamples contains examples for transaction operations.
func TransactionsExamples(ctx context.Context, client *mongo.Client) error {
	_, err := client.Database("hr").Collection("employees").InsertOne(ctx, bson.D{{"pi", 3.14159}})
	if err != nil {
		return err
	}

	_, err = client.Database("hr").Collection("employees").DeleteOne(ctx, bson.D{{"pi", 3.14159}})
	if err != nil {
		return err
	}

	_, err = client.Database("reporting").Collection("events").InsertOne(ctx, bson.D{{"pi", 3.14159}})
	if err != nil {
		return err
	}

	_, err = client.Database("reporting").Collection("events").DeleteOne(ctx, bson.D{{"pi", 3.14159}})
	if err != nil {
		return err
	}

	// Start Transactions Retry Example 3
	runTransactionWithRetry := func(sctx mongo.SessionContext, txnFn func(mongo.SessionContext) error) error {
		for {
			err := txnFn(sctx) // Performs transactions.
			if err == nil {
				return nil
			}

			log.Printf("Transaction aborted. Caught exception during transaction.")

			// If transient error, retry the whole transaction
			if cmdErr, ok := err.(mongo.CommandError); ok && cmdErr.HasErrorLabel("TransientTransactionError") {
				log.Printf("TransientTransactionError, retrying transaction...")
				continue
			}
			return err
		}
	}

	commitWithRetry := func(sctx mongo.SessionContext) error {
		for {
			err := sctx.CommitTransaction(sctx)
			switch e := err.(type) {
			case nil:
				log.Printf("Transaction committed.")
				return nil
			case mongo.CommandError:
				// Can retry commit
				if e.HasErrorLabel("UnknownTransactionCommitResult") {
					log.Printf("UnknownTransactionCommitResult, retrying commit operation...")
					continue
				}
				log.Printf("Error during commit...\n")
				return e
			default:
				log.Printf("Error during commit...\n")
				return e
			}
		}
	}

	// Updates two collections in a transaction.
	updateEmployeeInfo := func(sctx mongo.SessionContext) error {
		employees := client.Database("hr").Collection("employees")
		events := client.Database("reporting").Collection("events")

		err := sctx.StartTransaction(options.Transaction().
			SetReadConcern(readconcern.Snapshot()).
			SetWriteConcern(writeconcern.New(writeconcern.WMajority())),
		)
		if err != nil {
			return err
		}

		_, err = employees.UpdateOne(sctx, bson.D{{"employee", 3}}, bson.D{{"$set", bson.D{{"status", "Inactive"}}}})
		if err != nil {
			sctx.AbortTransaction(sctx)
			log.Printf("caught exception during transaction, aborting.")
			return err
		}

		_, err = events.InsertOne(sctx, bson.D{{"employee", 3}, {"status", bson.D{{"new", "Inactive"}, {"old", "Active"}}}})
		if err != nil {
			sctx.AbortTransaction(sctx)
			log.Printf("caught exception during transaction, aborting.")
			return err
		}

		return commitWithRetry(sctx)
	}

	return client.UseSessionWithOptions(
		ctx, options.Session().SetDefaultReadPreference(readpref.Primary()),
		func(sctx mongo.SessionContext) error {
			return runTransactionWithRetry(sctx, updateEmployeeInfo)
		},
	)
}
```


## 实例十四 ChangeStream

```
// ChangeStreamExamples contains examples of changestream operations.
func ChangeStreamExamples(t *testing.T, db *mongo.Database) {
	ctx := context.Background()

	coll := db.Collection("inventory_changestream")
	coll.Drop(context.Background())

	coll.InsertOne(ctx, bson.D{{"x", int32(1)}})

	var stop int32
	doInserts := func(coll *mongo.Collection) {
		for atomic.LoadInt32(&stop) == 0 {
			coll.InsertOne(ctx, bson.D{{"x", 1}})
			time.Sleep(10 * time.Microsecond)
			coll.DeleteOne(ctx, bson.D{{"x", 1}})
		}
	}
	go doInserts(coll)

	{
		// Start Changestream Example 1
		cs, err := coll.Watch(ctx, mongo.Pipeline{})
		defer cs.Close(ctx)

		ok := cs.Next(ctx)
		next := cs.Current

		fmt.Printf("%+v\n", err)
		fmt.Printf("%+v\n", ok)
		fmt.Printf("%+v\n", len(next))
	}

	{
		// Start Changestream Example 2
		cs, err := coll.Watch(ctx, mongo.Pipeline{}, options.ChangeStream().SetFullDocument(options.UpdateLookup))
		defer cs.Close(ctx)

		ok := cs.Next(ctx)
		next := cs.Current

		fmt.Printf("%+v\n", err)
		fmt.Printf("%+v\n", ok)
		fmt.Printf("%+v\n", len(next))
	}

	{
		original, err := coll.Watch(ctx, mongo.Pipeline{})
		defer original.Close(ctx)

		ok := original.Next(ctx)
		fmt.Printf("%+v\n", err)
		fmt.Printf("%+v\n", ok)

		resumeToken := original.ResumeToken()

		cs, err := coll.Watch(ctx, mongo.Pipeline{}, options.ChangeStream().SetResumeAfter(resumeToken))
		defer cs.Close(ctx)

		ok = cs.Next(ctx)
		result := cs.Current

		fmt.Printf("%+v\n", err)
		fmt.Printf("%+v\n", ok)
		fmt.Printf("%+v\n", len(result))
	}

	{
		pipeline := mongo.Pipeline{bson.D{{"$match", bson.D{{"$or",
			bson.A{
				bson.D{{"fullDocument.username", "alice"}},
				bson.D{{"operationType", "delete"}}}}},
		}}}
		cs, err := coll.Watch(ctx, pipeline)
		defer cs.Close(ctx)
		fmt.Printf("%+v\n", err)

		ok := cs.Next(ctx)
		next := cs.Current

		fmt.Printf("%+v\n", err)
		fmt.Printf("%+v\n", ok)
		fmt.Printf("%+v\n", len(next))
	}
}
```
