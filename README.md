# Hello, Mongodb

- [MongoDB Documentation](https://www.mongodb.com/docs/manual/tutorial/getting-started/)

##  起步

切换到某个 `db`，如果 `db` 不存在就会被创建。
```mongosh
use examples;
db;
```
插入文档。
```mongosh
db.movies.insertMany([
   {
      title: 'Titanic',
      year: 1997,
      genres: [ 'Drama', 'Romance' ],
      rated: 'PG-13',
      languages: [ 'English', 'French', 'German', 'Swedish', 'Italian', 'Russian' ],
      released: ISODate("1997-12-19T00:00:00.000Z"),
      awards: {
         wins: 127,
         nominations: 63,
         text: 'Won 11 Oscars. Another 116 wins & 63 nominations.'
      },
      cast: [ 'Leonardo DiCaprio', 'Kate Winslet', 'Billy Zane', 'Kathy Bates' ],
      directors: [ 'James Cameron' ]
   },
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
查找全部内容
```mongosh
db.movies.find( { } )
```
也可以这样写，在 `collection` 的名字有特殊字符的时候可以用到。
```mongosh
db["movies"].find()
```
对查询进行过滤。
```mongosh
db.movies.find( { "directors": "Christopher Nolan" } );
db.movies.find( { "released": { $lt: ISODate("2000-01-01") } } );
```
多个条件
```mongosh
db.movies.find( { "released": { $lt: ISODate("2000-01-01") }, "directors": "James Cameron" } )
```
有以下关系运算符
|名称|描述|
|---|---|
|$eq|等于，Matches values that are equal to a specified value.|
|$gt|大于，Matches values that are greater than a specified value.|
|$gte|大于等于，Matches values that are greater than or equal to a specified value.|
|$in|在 arr 中的，Matches any of the values specified in an array.|
|$lt|小于，Matches values that are less than a specified value.|
|$lte|小于等于，Matches values that are less than or equal to a specified value.|
|$ne|不等于，Matches all values that are not equal to a specified value.|
|$nin|不在 arr 中的，Matches none of the values specified in an array.|

控制返回的字段
```mongosh
db.movies.find( { }, { "title": 1, "directors": 1, "year": 1 } );
db.movies.find( { }, { "_id": 0, "title": 1, "genres": 1 } );
```
聚合数据
```mongosh
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
- `$group` 按 `genres` 分组。
- `$count` 对 `genres` 进行 `count`。
- `$sort` 排序，升序为 `-1`（后项小于前项），反之降序为 `1`（后项大于前项）。
