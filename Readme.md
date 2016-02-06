# UniversQL

A universal query language for databases.


## Introduction

A large number of database queries perform the same basic tasks: reads, writes,
updates and deletes. Yet each new NoSQL technology is accompanied by its own
query language. While this adds power, it also increases complexity. UniversQL
intends to provide a universal grammar (instruction set) that can subsequently
be adapted to different technologies and architectures.

This is not intended to replace individual query languages. As always, some queries
will require database-specific query expertise. However, in the spirit of the
Pareto Principle, we can cover many use cases. Furthermore, rather than expecting
each user to understand the nuances of a given query language, we can focus on
creating UniversQL-to-database transpilers/adapters that automatically embody
best practices.

A parsing expression grammar file is included in this repository, which can be
used as a starting point to create language-specific parsers. For an example
implementation, please visit https://github.com/brandoncarl/universql-js.


## Examples

#### Read

```coffee
# UniversQL
albums{name,year}[sort=-year,limit=2]?artist="Taylor*"
```

```sql
# SQL
SELECT name, year FROM albums WHERE artist LIKE 'Taylor*' ORDER BY year DESC LIMIT 2;
```

```coffee
# MongoDB
db.albums.find({ artist: { $regex: /^Taylor/ } }, { name : 1, year : 1 }).sort({ year : -1 }).limit(2);
```

```coffee
# RethinkDB
r.table('albums').filter(r.row('artist').match('Taylor.*')).withFields("name", "year").orderBy(r.desc('year')).limit(2);
```

```coffee
# CouchDB
# !!! How to query?
```

#### Create

```coffee
# UniversQL
+albums{name:'1989',artist:'Taylor Swift',year:2015}
```

```sql
# SQL
INSERT INTO albums (name, artist, year) VALUES ('1989', 'Taylor Swift', 2015);
```

```coffee
# MongoDB
db.albums.save({ name : '1989', artist : 'Taylor Swift', year : 2015 })
```

```coffee
# RethinkDB
r.table('albums').insert([{ name : '1989', artist : 'Taylor Swift', year : 2015 }]);
```

```coffee
# CouchDB
curl -X POST http://127.0.0.1:5984/db/ -d "{ _type : 'album', name : '1989', artist : 'Taylor Swift', year : 2015 }" -H "Content-Type: application/json"
```

#### Update

```coffee
# UniversQL
^albums{producer:'Max Martin'}?name='1989'
```

```sql
# SQL
UPDATE albums SET producer = 'Max Martin' WHERE name = '1989';
```

```coffee
# MongoDB
db.albums.update({ name : '1989' }, { $set : { producer : 'Max Martin' } });
```

```coffee
# RethinkDB
r.table('artists').filter(r.row('name').eq(1989)).update({ producer : 'Max Martin'})
```

```coffee
# CouchDB
# !!! Need to do a query first (and perhaps create a view)
curl -X PUT http://127.0.0.1:5984/db/ -d "{ _id : id, _rev : rev, producer : 'Max Martin' }" -H "Content-Type: application/json"
```

#### Delete

```coffee
# UniversQL
-albums{name:'1989'}
```

```sql
# SQL
DELETE FROM albums WHERE name = '1989';
```

```coffee
# MongoDB
db.albums.remove({ name : '1989' });
```

```coffee
# RethinkDB
r.table('artists').filter(r.row('name').eq(1989)).delete()
```

```coffee
# CouchDB
# !!! Need to do a query first (and perhaps create a view)
curl -X DEL http://127.0.0.1:5984/db/ -d "{ _id : id, _rev : rev }" -H "Content-Type: application/json"
```


## Advanced Elements

Multiple insert
Upsert
Embedded documents
Joins
Aggregation
Distinct
Hinting

## Goals

1. The language should be immediately intuitive to most developers.  
2. The language should be accessible to non-developers with minimal effort.  

## Implementation Guidelines

1. Implementations for languages should have repositories named "universql-<language>".
For example, "universql-ruby" or "universql-js".  
2. The package name for the core implementation in each language should be
"universql". This allows commands like `npm install universql` or `gem install universql`.  
3.
