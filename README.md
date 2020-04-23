  # 🚀 GocqlX [![GoDoc](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](http://godoc.org/github.com/scylladb/gocqlx) [![Go Report Card](https://goreportcard.com/badge/github.com/scylladb/gocqlx)](https://goreportcard.com/report/github.com/scylladb/gocqlx) [![Build Status](https://travis-ci.org/scylladb/gocqlx.svg?branch=master)](https://travis-ci.org/scylladb/gocqlx)

GocqlX makes working with Scylla easy and error prone without sacrificing performance.
It’s inspired by [Sqlx](https://github.com/jmoiron/sqlx), a tool for working with SQL databases, but it goes beyond what Sqlx provides.

## Features

* Binding query parameters from struct fields, map, or both
* Scanning query results into structs based on field names
* Convenient functions for common tasks such as loading a single row into a struct or all rows into a slice (list) of structs
* Making any struct a UDT without implementing marshalling functions
* GocqlX is fast. Its performance is comparable to raw driver. You can find some benchmarks [here](#performance).

Subpackages provide additional functionality:

* CQL query builder ([package qb](https://github.com/scylladb/gocqlx/blob/master/qb))
* CRUD operations based on table model ([package table](https://github.com/scylladb/gocqlx/blob/master/table))
* Database migrations ([package migrate](https://github.com/scylladb/gocqlx/blob/master/migrate))

## Installation

    go get -u github.com/scylladb/gocqlx

## Getting started

Wrap gocql Session:

```go
// Create gocql cluster.
cluster := gocql.NewCluster(hosts...)
// Wrap session on creation, gocqlx session embeds gocql.Session pointer. 
session, err := gocqlx.WrapSession(cluster.CreateSession())
if err != nil {
	t.Fatal(err)
}
```

Specify table model:

```go
// metadata specifies table name and columns it must be in sync with schema.
var personMetadata = table.Metadata{
	Name:    "person",
	Columns: []string{"first_name", "last_name", "email"},
	PartKey: []string{"first_name"},
	SortKey: []string{"last_name"},
}

// personTable allows for simple CRUD operations based on personMetadata.
var personTable = table.New(personMetadata)

// Person represents a row in person table.
// Field names are converted to camel case by default, no need to add special tags.
// If you want to disable a field add `db:"-"` tag, it will not be persisted.
type Person struct {
	FirstName string
	LastName  string
	Email     []string
}
```

Bind data from a struct and insert a row:

```go
p := Person{
	"Michał",
	"Matczuk",
	[]string{"michal@scylladb.com"},
}
q := session.Query(personTable.Insert()).BindStruct(p)
if err := q.ExecRelease(); err != nil {
	t.Fatal(err)
}
```

Load a single row to a struct:

```go
p := Person{
	"Michał",
	"Matczuk",
	nil, // no email
}
q := session.Query(personTable.Get()).BindStruct(p)
if err := q.GetRelease(&p); err != nil {
	t.Fatal(err)
}
t.Log(p)
// stdout: {Michał Matczuk [michal@scylladb.com]}
```

Load all rows in to a slice:

```go
var people []Person
q := session.Query(personTable.Select()).BindMap(qb.M{"first_name": "Michał"})
if err := q.SelectRelease(&people); err != nil {
	t.Fatal(err)
}
t.Log(people)
// stdout: [{Michał Matczuk [michal@scylladb.com]}]
```

## Examples

You can find lots of examples in [example_test.go](https://github.com/scylladb/gocqlx/blob/master/example_test.go).

Go and run them locally:

```bash
make run-scylla
make run-examples
```

## Performance

GocqlX performance is comparable to the raw `gocql` driver.
Below benchmark results running on my laptop.

```
BenchmarkBaseGocqlInsert            2392            427491 ns/op            7804 B/op         39 allocs/op
BenchmarkGocqlxInsert               2479            435995 ns/op            7803 B/op         39 allocs/op
BenchmarkBaseGocqlGet               2853            452384 ns/op            7309 B/op         35 allocs/op
BenchmarkGocqlxGet                  2706            442645 ns/op            7646 B/op         38 allocs/op
BenchmarkBaseGocqlSelect             747           1664365 ns/op           49415 B/op        927 allocs/op
BenchmarkGocqlxSelect                667           1877859 ns/op           42521 B/op        932 allocs/op
```

See the benchmark in [benchmark_test.go](https://github.com/scylladb/gocqlx/blob/master/benchmark_test.go).

## License

Copyright (C) 2017 ScyllaDB

This project is distributed under the Apache 2.0 license. See the [LICENSE](https://github.com/scylladb/gocqlx/blob/master/LICENSE) file for details.
It contains software from:

* [gocql project](https://github.com/gocql/gocql), licensed under the BSD license
* [sqlx project](https://github.com/jmoiron/sqlx), licensed under the MIT license

Apache®, Apache Cassandra® are either registered trademarks or trademarks of 
the Apache Software Foundation in the United States and/or other countries. 
No endorsement by The Apache Software Foundation is implied by the use of these marks.

GitHub star is always appreciated!
