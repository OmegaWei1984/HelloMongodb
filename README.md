# Hello, Mongodb

[MongoDB Documentation](https://www.mongodb.com/docs/manual/tutorial/getting-started/) 的笔记和实践

[TOC]

# 基础

切换到某个 `db`，如果 `db` 不存在就会被创建。

```javascript
use examples;
db;
```

# CRUD

## 插入文档

插入单个

```javascript
db.movies.insertOne(
   {
      title: 'The Dark Knight',
      year: 2008,
      genres: [ 'Action', 'Crime', 'Drama' ],
      rated: 'PG-13',
      languages: [ 'English', 'Mandarin' ],
      released: ISODate("2008-07-18T00:00:00.000Z"),
      awards: {
         wins: 144,
         nominations: 106,
         text: 'Won 2 Oscars. Another 142 wins & 106 nominations.'
      },
      cast: [ 'Christian Bale', 'Heath Ledger', 'Aaron Eckhart', 'Michael Caine' ],
      directors: [ 'Christopher Nolan' ]
   }
)
```

插入多个

```javascript
db.movies.insertMany([
   {
      title: 'The Dark Knight',
      year: 2008,
      genres: [ 'Action', 'Crime', 'Drama' ],
      rated: 'PG-13',
      languages: [ 'English', 'Mandarin' ],
      released: ISODate("2008-07-18T00:00:00.000Z"),
      awards: {
         wins: 144,
         nominations: 106,
         text: 'Won 2 Oscars. Another 142 wins & 106 nominations.'
      },
      cast: [ 'Christian Bale', 'Heath Ledger', 'Aaron Eckhart', 'Michael Caine' ],
      directors: [ 'Christopher Nolan' ]
   },
   {
      title: 'Spirited Away',
      year: 2001,
      genres: [ 'Animation', 'Adventure', 'Family' ],
      rated: 'PG',
      languages: [ 'Japanese' ],
      released: ISODate("2003-03-28T00:00:00.000Z"),
      awards: {
         wins: 52,
         nominations: 22,
         text: 'Won 1 Oscar. Another 51 wins & 22 nominations.'
      },
      cast: [ 'Rumi Hiiragi', 'Miyu Irino', 'Mari Natsuki', 'Takashi Naitè' ],
      directors: [ 'Hayao Miyazaki' ]
   },
   {
      title: 'Casablanca',
      genres: [ 'Drama', 'Romance', 'War' ],
      rated: 'PG',
      cast: [ 'Humphrey Bogart', 'Ingrid Bergman', 'Paul Henreid', 'Claude Rains' ],
      languages: [ 'English', 'French', 'German', 'Italian' ],
      released: ISODate("1943-01-23T00:00:00.000Z"),
      directors: [ 'Michael Curtiz' ],
      awards: {
         wins: 9,
         nominations: 6,
         text: 'Won 3 Oscars. Another 6 wins & 6 nominations.'
      },
      lastupdated: '2015-09-04 00:22:54.600000000',
      year: 1942
   }
])
```

## 查找文档

查找全部内容。

```javascript
db.movies.find( { } )
```

也可以这样写，在 `collection` 的名字有特殊字符的时候可以用到。

```javascript
db["movies"].find()
```

### 条件查询

对查询进行过滤。

```javascript
db.movies.find( { "directors": "Christopher Nolan" } );
db.movies.find( { "released": { $lt: ISODate("2000-01-01") } } );
```

通过多个条件进行过滤。

```javascript
db.movies.find( { "released": { $lt: ISODate("2000-01-01") }, "directors": "James Cameron" } )
```

有以下关系运算符

| 名称    | 描述          |
| ----- | ----------- |
| \$eq  | 等于          |
| \$gt  | 大于          |
| \$gte | 大于等于        |
| \$in  | 在 array 中的  |
| \$lt  | 小于          |
| \$lte | 小于等于        |
| \$ne  | 不等于         |
| \$nin | 不在 array 中的 |

多个并列的条件为 `and`

```javascript
db.movies.find({
   rated: "PG",
   year: { $gte: 1942 }
})
```

使用 `$or` 来表示 `or` 的关系

```javascript
db.movies.find({
   $or: [{ rated: "PG" }, { year: { $gte: 1998 }}]
})
```

可以通过 `find()` 的第二个参数进行过滤控制要返回那些字段。1 代表需要这个字段，0 代表需要除了这个之外的其他字段，`_id` 比较特殊，需要用 0 去过滤否则就是默认需要返回，其他字段是不能 0 1 同时使用。

```javascript
db.movies.find( { }, { "title": 1, "directors": 1, "year": 1 } );
db.movies.find( { }, { "_id": 0, "title": 1, "genres": 1 } );
```

### 嵌套查询

通过点表示法（`.`）例如 `"xxx.yyy.zzz"` 可以嵌套查询 `{"xxx": {"yyy": {"zzz": 42 }}}`

tips：点表示法的字段和嵌套字段用需要用字符串表示

```javascript
db.movies.find({ "awards.wins": 52 })
```

下面这种形式的查找的效果是去匹配字段完全一样的数据，如果存在 ``{"xxx": {"yyy": {"zzz": 42, `zzzz`: 123}}}`` 就会匹配不到。

```javascript
db.movies.find({"xxx": {"yyy": {"zzz": 42 }}})
```

### 匹配数组

使用 `$all` 操作符可以匹配数组中是否含有这个值。

```javascript
db.movies.find({ genres: { $all: ['Animation'] } })
```

和嵌套的对象不一样，对于匹配数组中的某个元素，这样写是可以的。

```javascript
db.movies.find({ genres: 'Animation' })
```

使用 `$size` 操作符匹配数组的长度

```javascript
db.movies.find({ genres: { $size: 3 } })
```

通过点表示法（`.`）可以表示数组的索引

```javascript
db.movies.find({"genres.2": "Family"})
```

通过点表示法（`.`）匹配数组中的每个对象的其中一个指定字段

```javascript
db.inventory.insertMany( [
   { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
   { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
   { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
   { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);

db.inventory.find( { 'instock.qty': { $gte: 20 } } )
```

如果想匹配数组中的整个元素字段顺序必须保持一致

```javascript
// 不能匹配
db.inventory.find( { "instock": { qty: 5, warehouse: "A" } } )
// 能匹配
db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } )
```

在匹配数组时只有使用 `$elemMatch` 操作符才是按照元素进行匹配。

使用 `$elemMatch` 操作符同时匹配某一个元素的多个指定字段多个条件（AND 的关系）

```javascript
db.inventory.find( { "instock": { $elemMatch: { qty: {$gte: 50} , warehouse: "A" } } } )
```

不使用 `$elemMatch` 操作符对单个字段进行多条件匹配，只要多个元素中的某一个符合其中一个条件就会被匹配到（OR 的关系）

```javascript
db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 } } )
```

使用了 `$elemMatch` 操作符则是多个条件必须同时满足（AND 的关系）

```javascript
db.inventory.find( { "instock": { $elemMatch: { qty: 5, warehouse: "A" } } } )
```

但是不使用 `$elemMatch` 操作符去匹配多个字段时，数组对象字段的多个条件必须都符合才能被匹配到（AND 的关系）。虽然多个条件需要同时被满足，但是数组对象多个元素中只要其中有一个元素符合其中一条条件就算这一条条件被匹配到了，也就是说多个条件可能被数组中的不同元素分别满足其中的一部分。

```javascript
db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 }, "instock.warehouse": 'A' } )

db.inventory.find( { "instock.qty": 5, "instock.warehouse": "A" } )
```

### Null

查找某个字段为 `null` 意为找到某个字段的值为 `null` 或者不存在某个字段的文档

```javascript
db.inventory.insertMany([
   { _id: 1, item: null },
   { _id: 2 }
])

db.inventory.find( { item: null } )
```

### 类型检查

使用 `$type` 操作符匹配字段值的指定类型

```javascript
db.inventory.find( { item : { $type: 10 } } )
db.inventory.find( { item : { $type: 'null' } } )
```

[BSON Type](https://www.mongodb.com/docs/manual/reference/bson-types/#std-label-bson-types) 可以用 number 或者 字符串别名表示

### 字段检查

使用 `$exists` 操作符匹配存在或者不存在某个字段的文档

```javascript
db.inventory.find( { item : { $exists: false } } )
```

### 聚合数据

```javascript
db.movies.aggregate( [
   { $unwind: "$genres" },
   {
     $group: {
       _id: "$genres",
       genreCount: { $count: { } }
     }
   },
   { $sort: { "genreCount": -1 } }
] )
```

*   `$group` 按 `genres` 分组。
*   `$count` 对 `genres` 进行 `count`。
*   `$sort` 排序，升序为 `-1`（后项小于前项），反之降序为 `1`（后项大于前项）。

### 查询快照

# Databases and Collections

创建 `collection`，当 `collection` 不存在时，直接插入或者创建 `index` 都会进行创建。

```javascript
db.c1.insertOne({x:1})
db.c2.createIndex({y:1})



example> db.c1.find({})
[ { _id: ObjectId("63eefeb076d38ba98289dd05"), x: 1 } ]
example> db.c2.find({})
```

直接进行创建。

```javascript
db.createCollection("c3")
```

查看全部 `collection`。

```javascript
db.getCollectionInfos()
```

## view

`view` 是 `mongodb` 中的只读对象/视图，类似 `MySQL` 中的视图。标准视图（standard views）不会被保存到磁盘上，视图是进行查询的时候计算出来的。

创建一个 `view`。

```javascript
db.createView(
    "Nolan's Movie",
    "movies",
    [ { $match: { "directors": 'Christopher Nolan' } } ]
)
```

也可以使用 `db.createCollection()` 创建 `view`。

```javascript
db.createCollection(
   "Nolan's Moive II",
   {
      viewOn: "movies",
      pipeline: [ { $match: {"directors": 'Christopher Nolan'} } ]
    }
)



show collections;
Nolan's Moive II  [view]
Nolan's Movie     [view]
```

对 `view` 进行查看。

```javascript
db["Nolan's Movie"].find({})
```
