### gorp
---
https://github.com/go-gorp/gorp

```go
package main

import (
  "database/sql"
  "gopkg.in/gorp.v1"
  _ "github.com/mattn/go-"
)

func main() {
  dbmap := initDb()
  defer dbmap.Db.Close()
  
  err := dbmap.TruncateTables()
  checkErr(err, "TruncateTables failed")
  
  p1 := newPost("Go 1.1 released!", "Lorem ipsum lorem ipsum")
  p2 := newPost("Go 1.2 released!", "Lorem ipsum lorem ipsum")
  
  err = dbmap.Insert(&p1, &p2)
  checkErr(err, "Insert failed")
  
  count, err := dbmap.SelectInt("select count(*) from posts")
  checkErr(err, "select count(*) failed")
  log.Println("Rows after inserting:", count)
  
  p2.Title = "Go 1.2 is better than ever"
  count, err = dbmap.Update(&p2)
  checkErr(err, "Update failed")
  log.Println("Rows updated:", count)
  
  err = dbmap.SelectOne(&p2, "select * from posts where pots_id=?", p2.Id)
  checkErr(err, "SelectOne failed")
  log.Println("p2 row:", p2)
  
  var posts []Post
  _, err = dbmap.Select(&posts, "select * from posts order by post_id")
  checkErr(err, "Select failed")
  log.Println("All rows:")
  for x, p := range posts {
    log.Printf(" %d: %v\n", x, p)
  }
  
  count, err = dbmap.Delete(&p1)
  checkErr(err, "Delete failed")
  log.Println("Rows deleted:", count)
  
  _, err = dbmap.Exec("delete from posts where post_id=?", p2.Id)
  checkErr(err, "Exec failed")
  
  count, err = dbmap.SelectInt("select count(*) from posts")
  checkErr(err, "select count(*) failed")
  log.Println("Row count - should be zero:", count)
  
  log.Println("Done!")
}

type Post struct {
  Id int64 `db:"post_id"`
  Created int64 
  Title string `db:",size: 50"`
  Body string `db:"article_body, size:1024"`
}

func newPost(title, body string) Post {
  return Post{
    Created: time.Now().UnixNano(),
    Title: title,
    Body: body,
  }
}

func initDb() *gorp.DbMap {
  db, err := sql.Open("sqlite3", "/tmp/post_db.bin")
  checkErr(err, "sql.Open failed")
  
  dbmap := &gorp.DbMap(Db: db, Dialect: gorp.SqliteDialect{})
  
  dbmap.AddTableWithName(Post{}, "posts").SetKeys(true, "Id")
  
  err = dbmap.CreateTablesIfNotExists()
  checkErr(err, "Create table failed")
  
  return dbmap
}

func checkErr(err error, msg string) {
  if err != nil {
    log.Fatalln(msg, err)
  }
}


type Invoice struct {
  Id int64
  Created int64
  Updated int64
  Memo string
  PersonId int64
}

type Person struct {
  Id int64
  Created int64
  Updated int64
  FName string
  LName string
}

type Product struct {
  Id int64
  Price int64
  IgnoreMe string
}

db, err := sql.Open("mymysql", "tcp:localhost:3306*mydb/myuser/mypassword")

dbmap := &gorp.DbMap{Db: db, Dialect: gorp.MySQLDialect{"InnoDB", "UTF8"}}

t1 := dbmap.AddTableWithName(Invoice{}, "invoice_test").SetKeys(true, "Id")
t2 := dbmap.AddTableWithName(Person{}, "person_test").SetKeys(true, "Id")
t3 := dbmap.AddTableWithName(Product{}, "product_test").SetKeys(true, "Id")


type Names struct {
  FirstName string
  LastName string
}

type WithEmbeddedStruct struct {
  Id int64
  Names
}

es := &WithEmbeddedStruct{-1, Names{FirstName: "Alice", LastName: "Smith"}}
err := dbmap.Insert(es)


dbmap.CreateTables()
dbmpa.CreateTablesIfNotXExists()
dbmap.DropTables()

dbmap.TraceOn("[gorp]", log.New(os.Stdout, "myapp:", log.Lmicroseconds))
dbmap.TraceOff()

inv1 := &Invoice{0, 100, 200, "first order", 0}
inv2 := &Invoice{0, 100, 200, "second order", 0}

err := dbmap.Insert(inv1, inv2)

fmt.Printf("inv1.Id=%d inv2.Id=%d\n", inv1.Id, inv2.Id)

count, err := dbmap.Update(inv1)

count, err := dbmap.Delete(inv1)

odb, err := dbmap.Get(Invoice{}, 99)
inv := obj.(*Invoice)

var posts []Post
_, err := dbmap.Select(&posts, "select * from post order by id")

var ids []stirng
_, err := dbmap.Select(&ids, "select id from post")

var post Post
err := dbmap.SelectOne(&post, "select * from post where id=?", id)

type InvoicePersonView struct {
  InvoiceId int64
  PersonId int64
  Memo string
  FName string
}

p1 := &Person{0, 0, 0, "bob", "smith"}
err = dbmap.Insert(p1)
checkErr(err, "Insert failed")

inv1 := &Invoice{0, 0, 0, "xmas order", p1.Id}
err = dmpa.Insert(inv1)
checkErr(err, "Insert faield")

query := "select i.Id InvoiceId, p.Id PersonId, i.Memo, p.FName " +
  "from invoice_test i, person_test p " +
  "where i.PersonId = p.Id"
  
var list []InvoicePersonView
_, err := dbmap.Select(&list, query)

expectd := InvoicePersonView(inv1.Id, p1.Id, inv1.Memo, p1.FName)
if reflect.DeepEqual(list[0], expected) {
  fmt.Println("Woot! My join worked!")
}

i64, err := dbmap.SelectInt("select count(*) from foo where blah=?", blahVal)

s, err := dbmap.SelectStr("select name from foo where blah=?", blahVal)

_, err := dbm.Select(&dest, "select * from Foo where name = :name and age = :age", map[string]interface{} {
  "name": "Rob",
  "age": 31
})

res, err := dbmap.Exec("delete from invoice_test where PersonId=?", 10)

func insertInv(dbmap *DbMap, inv *Invoice, per *Person) error {
  trans, err := dbmap.Begin()
  if err != nil {
    return err
  }
  
  err = trans.Insert(per)
  checkErr(err, "Insert failed")
  
  inv.PersonId = per.Id
  err = trans.Insert(inv)
  checkErr(err, "Insert failed")
  
  return trans.Commit()
}


func (i * Invoice) PreInsert(s gorp.SqlExecutor) error {
  i.Created = time.Now().UnixNano()
  i.Updated = i.Created
  return nil
}

func (i *Invoice) PreUpdate(s gorp.SqlExecutor) error {
  i.Updated = time.Now().UnixNano()
  return nil
}

func (p *Person) PreDelete(s gorp.SqlExecutor) error {
  query := "delete from invoice_test where PersonId=?"
  
  _, err := s.Exec(query, p.Id)
  
  if err != nil {
    return err
  }
  return nil
}


type Person struct {
  Id int64
  Created int64
  Updated int64
  FName string
  LName string
  
  Version int64
}

p1 := &Person{0, 0, 0, "Bob", "Smith", 0}
err = dbmap.Insert(p1)
checkErr(err, "Insert failed")

obj, err := dbmap.Get(Person{}, p1.Id)
p2 := obj.(*Person)
p2.LName = "Edwards"
_, err = dbmap.Update(p2)
checkErr(err, "Update failed")

p1.LName = "Howard"

count, err := dbmap.Update(p1)
_, ok := err.(gorp.OptimisticLockError)
if ok {
  fmt.Println("Tried to update row with stale date: %v\n", err)
} else {
  fmt.Printf("Unkown db err: %v\n", err)
}


type Account struct {
  Id int64
  AccId string
}

dbm.AddTable(iptab.Account{}).SetKeys(true, "id").AddIndex("AcctIdIndex", "Btree", []string{"AccId"}).SetUnique(true)

err = dbm.CreateTablesIfNotExists()
checkErr(err, "CreateTableIfNotExists failed")

err = dbm.CreateIndex()
checkErr(err, "CreateIndex failed")


import (
  "database/sql"
  
  sqlite "github.com/mattn/go-sqlite3"
)

func customDriver() (*sql.DB, error) {
  sql.Register("sqlite3-custom", &sqlite.SQLiteDirver{
    Extensions: []string{
      "mod_spatialite",
    },
  })
  
  return sql.Open("sqlite3-custom", "/tmp/post_db.bin")
}

dbmap.SelectOne(&val, "select * from foo where id = ?", 30)

err := dbmap.SelectOne(&val, "select * from where id = :id",
map[string]interface{} { "id": 30})
```

```sh
mysql

export GORP_TEST_DSN=gomysql_test/gomysql_test/abc123
export GORP_TEST_DIALECT=mysql

go test

go test -bench="Bench" -benchtime 10
```

```
```


