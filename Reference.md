## Reference

In general, queries follow the format: `TABLES(JOINS){FIELDS}[OPTIONS]?FILTERS`.

#### Prefixes

Prefixes allow us to quickly create, update, delete and describe entities.

|Prefix|Meaning|Example|
|---|---|---|
| None | Perform a query|`users?name=Peter`|
| + | Insert a record, document, row, etc. |`+users{name:"Peter"}`|
| ^ | Update a record, document, row, etc. |`^users{active:true}`|
| - | Delete a record, document, row, etc. |`-users?name=Peter`|
| ? | Explain a query (if available) |`?users?name=Peter`|
| ++ | Add a table, collection, etc. |`++posts`|
| -- | Drop a table, collection, etc. |`--posts!!`|
| ^^ | Alter a table, collection, etc. |`^^users{quote:String}`|
| ?? | Describe a table, collection, etc. |`??users`|
| +++ | Add a database |`+++placebook`|
| --- | Remove a database |`---placebook!!!`|
| ??? | Describe a database|`???placebook`|

* Note that we require ! after deletion operators as a safeguard.


#### Examples

These examples originated in RethinkDB's [excellent cookbook](https://www.rethinkdb.com/docs/sql-to-reql/javascript/).

**Select all fields**
```coffee
# UniversQL
users
```
```sql
# SQL
SELECT * FROM users;
```
```coffee
# MongoDB
db.users.find()
```
```coffee
# RethinkDB
r.table("users")
```

**Select specific fields**
```coffee
# UniversQL
users{user_id,name}
```
```sql
# SQL
SELECT user_id, name FROM users
```
```coffee
# MongoDB
db.users.find(
  {},
  { user_id: 1, name: 1 }
)
```
```coffee
# RethinkDB
r.table("users")
 .pluck("user_id", "name")
```


**Select with single condition**
```coffee
# UniversQL
users?name=Peter
```
```sql
# SQL
SELECT * FROM users WHERE name = "Peter"
```
```coffee
# MongoDB
db.users.find(
  { name: "Peter" }
)
```
```coffee
# RethinkDB
r.table("users").filter({
    "name": "Peter"
})
```


**Select with multiple conditions**
```coffee
# UniversQL
users?name=Peter&age=30
```
```sql
# SQL
SELECT * FROM users WHERE name = "Peter" AND age = 30
```
```coffee
# MongoDB
db.users.find(
  { name: "Peter", age : 30 }
)
```
```coffee
# RethinkDB
r.table("users").filter({
    "name": "Peter",
    "age": 30
})
```


**Select with regex condition**
```coffee
# UniversQL
users?name=P*
users?name=/^P/
```
```sql
SELECT * FROM users WHERE name LIKE "P%"
```
```coffee
# MongoDB
db.users.find(
  { name : /^P/ }
)
```
```coffee
# RethinkDB
r.table("users").filter(
    r.row['name'].match("^P")
)
```

**Select with sort**
```coffee
# UniversQL
users[sort=name]
```
```sql
# SQL
SELECT * FROM users ORDER BY name ASC
```
```coffee
# MongoDB
db.users.find().sort({ name : 1 })
```
```coffee
# RethinkDB
r.table("users").order_by("name")
```

**Select with sort descending**
```coffee
# UniversQL
users[sort=-name]
```
```sql
# SQL
SELECT * FROM users ORDER BY name DESC
```
```coffee
# MongoDB
db.users.find().sort({ name : -1 })
```
```coffee
# RethinkDB
r.table("users").order_by(
    r.desc("name")
)
```

**Select with fields, conditions, and sorts**
```coffee
# UniversQL
users{user_id}(sort=-name)?name=Peter
```
```sql
# SQL
SELECT user_id FROM users WHERE name = "Peter" ORDER BY name DESC
```
```coffee
# MongoDB
db.users.find({ name : "Peter "}, { user_id : 1 }).sort({ name : -1 })
```
```coffee
# RethinkDB
r.table("users").filter({
    "name": "Peter"
}).order_by(
    r.desc("name")
).pluck("user_id")
```


**Select with limit and skip**
```coffee
# UniversQL
users[limit=5,skip=10]
```
```sql
# SQL
SELECT * FROM users LIMIT 5 SKIP 10
```
```coffee
# MongoDB
db.users.find().limit(5).skip(10)
```
```coffee
# RethinkDB
r.table("users").skip(10).limit(5)
```


**Select with set inclusion**
```coffee
# UniversQL
users?name=[Peter,John]
```
```sql
# SQL
SELECT * FROM users WHERE name IN ('Peter', 'John')
```
```coffee
# MongoDB
db.users.find(
  { name : { $in : ["Peter", "John"] } }
)
```
```coffee
# RethinkDB
r.table("users").filter(
  function (doc) {
    return r.expr(["Peter","John"])
            .contains(doc("name"));
  }
)
```


**Select with set exclusion**
```coffee
# UniversQL
users?name!=[Peter,John]
```
```sql
# SQL
SELECT * FROM users WHERE name NOT IN ('Peter', 'John')
```
```coffee
# MongoDB
db.users.find(
  { name : { $nin : ["Peter", "John"] } }
)
```
```coffee
# RethinkDB
r.table("users").filter(
  function (doc) {
    return r.expr(["Peter","John"])
            .contains(doc("name"))
            .not();
  }
)
```


**Count**
```coffee
# UniversQL
count(users)
users.count()
users{count()}
```
```sql
# SQL
SELECT COUNT(*) FROM users
```
```coffee
# MongoDB
db.users.count()
```
```coffee
# RethinkDB
r.table("users").count()
```


**Count field with condition**
```coffee
# UniversQL
users.count(name)?age>18
```
```sql
# SQL
SELECT COUNT(name) FROM users WHERE age > 18
```
```coffee
# MongoDB
db.users.find( { age: { $gt: 18 }, name: { $exists: true } } ).count()
```
```coffee
# RethinkDB
r.table("users").filter(
    r.row.hasFields("name")
    .and(r.row("age").gt(18))
).count()
```


**Average**
```coffee
# UniversQL
users.avg(age)
```
```sql
# SQL
SELECT AVG("age") FROM users
```
```coffee
# MongoDB
db.users.aggregate([
  { $group : { _id : null, avg : { $avg : "$age" } } }
])
```
```coffee
# RethinkDB
r.table("users")
 .avg("age")
```

**Maximum**
```coffee
# UniversQL
users.max(age)
```
```sql
# SQL
SELECT MAX("age") FROM users
```
```coffee
# MongoDB
db.users.aggregate([
  { $group : { _id : null, max : { $max : "$age" } } }
])
```
```coffee
# RethinkDB
r.table("users")["age"]
 .max()
```

**Unique (distinct) elements**
```coffee
# UniversQL
users.distinct(name)
users.uniq(name)
users.unique(name)
```
```sql
# SQL
SELECT DISTINCT(name) FROM users
```
```coffee
# MongoDB
db.runCommand({ distinct: "users", key: "name" })
```
```coffee
# RethinkDB
r.table("users").pluck("name").distinct()
```

**Between**
```coffee
# UniversQL
users?age>=18&age<=65
```
```sql
# SQL
SELECT * FROM users WHERE age BETWEEN 18 AND 65;
```
```coffee
# MongoDB
db.users.find(
  { age: { $gte: 18, $lte: 65 } }
)
```
```coffee
# RethinkDB
r.table("users").filter(
    (r.row["age"] >= 18)
    & (r.row["age"] <= 65))
```

**Variable Mapping (e.g CASE WHEN)**
```coffee
# UniversQL
users{name,is_adult:if(age>18,"yes","no")}
```
```sql
# SQL
SELECT name, 'is_adult' = CASE WHEN age>18 THEN 'yes' ELSE 'no' END FROM users
```
```coffee
# MongoDB
db.users.aggregate(
 [
   { $project: { name: 1, is_adult: { $cond: [ { $gt: ["$age", 18] }, "yes", "no" ] } } }
 ]
)
```
```coffee
# RethinkDB
r.table("users").map({
    "name": r.row["name"],
    "is_adult": r.branch(
        r.row["age"] > 18,
        "yes",
        "no"
    )
})
```

**Where Exists**
```
??? TBD
SELECT * FROM posts WHERE EXISTS (SELECT * FROM users WHERE posts.author_id = users.id)

```

**Update**
```coffee
# UniversQL
^users{age:18}?age<18
```
```sql
# SQL
UPDATE users SET age = 18 WHERE age < 18
```
```coffee
# MongoDB
db.users.update({ age: { $lt: 18 } }, { $set: { age: 18 } }, { multi: true })
```
```coffee
# RethinkDB
r.table("users").filter(
    r.row["age"] < 18
).update({
    "age": 18
})
```

**Update via Incrementing**
```coffee
# UniversQL
^users{age:+=1}
```
```sql
# SQL
UPDATE users SET age = age+1
```
```coffee
# MongoDB
db.users.update({}, { $inc: { age: 1 } }, { multi: true })
```
```coffee
# RethinkDB
r.table("users").update(
    { "age": r.row["age"]+1 }
)
```

**Deleting All Records**
```coffee
# UniversQL
# Note that we require an exclamation point given the severity of the operation
-users!
```
```sql
# SQL
DELETE FROM users
```
```coffee
# MongoDB
db.users.remove({})
```
```coffee
# RethinkDB
r.table("users").delete()
```

**Delete With Conditions**
```coffee
# UniversQL
-users?age<18
```
```sql
# SQL
DELETE FROM users WHERE age < 18
```
```coffee
# MongoDB
db.users.remove({ age : { $lt : 18 } })
```
```coffee
# RethinkDB
r.table("users")
    .filter( r.row["age"] < 18)
    .delete()
```

**Inner Join**
```coffee
# UniversQL
p:posts=u:users(p.user_id=u.id){post_id:p.id,u.name,user_id:u.id}
p:posts,u:users(p.user_id=u.id){post_id:p.id,u.name,user_id:u.id}
```
```sql
# SQL
SELECT posts.id AS post_id, user.name, users.id AS user_id FROM posts INNER JOIN users ON posts.user_id = users.id
```
```coffee
# MongoDB
# This query depends on your data model: the Mongo adapter should allow
# for data model specification.

# Case 1: If `posts` are included as subdocuments in `users`
db.users.aggregate(
  [
    { $unwind: "$posts" },
    { $project: { name: 1, "user_id": "$_id", "post_id": "$posts._id" } }
  ]
)

# Case 2: If `posts` are included as subdocuments in `users`, doesn't need unwind
# Note that this example doesn't map fields properly
db.users.find({ posts: { $exists: true } }, { name: 1, "posts._id" : 1 })

# Case 3: If `posts` collection is separate from `users`
# This goes against the intent of MongoDB and is an anti-pattern.
var mapPost = function() { emit(this.user_id, { post_id: this._id }); }
var mapUser = function() { emit(this._id, { name: this.name, _id: this._id }); }
var reduceUserPost = function(key, entries) {
  var out = {};
  entries.forEach(function(entry) {
    if (entry.post_id) {
      out.posts = out.posts || [];
      out.posts.push(entry.post_id);
    } else {
      for (var key in entry)
        if (entry[key] !== null) out[key] = entry[key];      
    }
  });
  return out;
}

db.posts.mapReduce(mapPost, reduceUserPost, { out : { reduce : "user_post" } })
db.users.mapReduce(mapUser, reduceUserPost, { out : { reduce : "user_post" } })
db.user_post.aggregate(
  [
    { $unwind: "$value.posts" },
    { $project: { name: "$value.name", user_id: "$_id", post_id: "$value.posts" }}
  ]
)
```
```coffee
# RethinkDB
r.table("posts").innerJoin(
  r.table("users"),
  function (post, user) {
    return post("userId").eq(user("id"));
}).zip()
```


#### Below here, incomplete!

```
p:posts=|u:users(p.user_id=u.id)
SELECT * FROM posts RIGHT OUTER JOIN users ON posts.user_id = users.id
```

```
p:posts|=u:users(p.user_id=u.id)
SELECT * FROM posts LEFT JOIN users ON posts.user_id = users.id
```

```
posts{category}[group=category]
SELECT category FROM posts GROUP BY category
```

```
posts{category,sum(num_comments)}[group=category]
SELECT category, SUM('num_comments') FROM posts GROUP BY category
```

```
posts{category,status,sum(num_comments)}[group=(category,status)]
SELECT category, status, SUM('num_comments') FROM posts GROUP BY category, status
```

```
posts{category,sum(num_comments)}[group=category]?num_comments>7
SELECT category, SUM(num_comments) FROM posts WHERE num_comments > 7 GROUP BY category
```

```
??? TBD
SELECT category, SUM(num_comments) FROM posts GROUP BY category HAVING num_comments > 7
```

```
??? TBD
SELECT title, COUNT(title) FROM movies GROUP BY title HAVING COUNT(title) > 1
```

```
+++my_database
CREATE DATABASE my_database;
```

```
---my_database!!!
```

```
++users(id:Integer!,name:String,age:Integer)
CREATE TABLE users (id INT IDENTITY(1,1) PRIMARY KEY, name VARCHAR(50), age INT);
```
