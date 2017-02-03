---
layout: post
title: Golang and MySQL (or PostgreSQL)
permalink: blog/go-and-mysql-postgres/
comments: True
excerpt_separator: <!--more-->
---

Let's use a toolkit from one of Go's standard libraries: `database/sql`. It is an abstraction and you need to import a database driver also. We will be as simple and "down to the metal" as possible. Not a fan of ORMs, so this should be fun. Whatever SQL database you use, code should be similar or the same. Read my quick start post on the actual databases themselves [MySQL](http://www.jancarloviray.com/blog/mysql-quickstart-cheat-sheet/) and [PostgreSQL](http://www.jancarloviray.com/blog/postgres-quick-start-and-best-practices/) to get started with either.
<!--more-->

## Quick Start

### Import Driver

Import `database/sql` and a driver for the database. Don't use drivers directly - only refer to types defined in package if possible. This is to avoid dependency so you can easily change the driver or database later.

The underscore in the import `_` means we are just running the `Init` function of the package. No exported package names are visible. Internally, the driver registers itself as being available to the `database/sql` package.

```go 
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)
```

### Access the Database

```go 
func main() {
    // [user[:pass]@][protocol[(addr)]]/dbname[?p1=v1&...]
    db, err := sql.Open("mysql", "user:pass@/dbname")
    if err != nil { 
        log.Fatal(err) 
    }
    defer db.Close()

    err = db.Ping()
    if err != nil {
        log.Fatal(err.Error())
    }
}
```

Note that `sql.DB` **is not a connection**. When you use `Open()` you get a handle for the database and connections do not open until you need them. This means that it does not return an error if the server is not available or user/pass is invalid. If you want to check before making queries, use `db.Ping()`

- this does not establish a connection immediately - the first connection will be established lazily when needed
- remember to always check and handle errors returned on all operations

#### Connection Pooling

It is important to note that the `sql.DB` object is designed to be long-lived, so don't open and close databases frequently. Create **one** `sql.DB` object for each distinct database you need to access and keep it open until the program is done. Pass it around as needed, or make it available globally, but make sure you keep it open. If you don't you will experience connection problems or sporadic failures.

#### Are you running out of database connections? 

If you get a "too many connections" problem, you may need to set [func (*DB) SetMaxOpenConns](https://golang.org/pkg/database/sql/#DB.SetMaxOpenConns) to prevent infinite number of connections. This limits the number of total open connections to the database.

If you perform a query like `db.Query("SELECT * FROM table")`, a connection is opened and will not return until you either `Close()` the connection explicitly or iterate over the rows with `Next()`. Additionally, doing a `sql.Tx` will only return the connection after a `Commit` or `Rollback()` is called. **If you forget to completely iterate the rows and forget to close it, then your connection will NEVER go back to the pool.**

Additionally, keeping a connection idle for a long time can cause problems. Try setting `db.SetMaxIdleConns(0)` if you get connection timeouts because a connection is idle for too long.

### Retrieve Result Sets

Idiomatic ways to retrieve results from the datastore:

- execute a query that returns rows.
- prepare a statement for repeated use. Execute it multiple times. Destroy it.
- execute a statement in a one-off fashion, without preparing statements.
- execute a query that returns a single row (we use a shortcut func here).

If a function name includes `Query` it is designed to ask a question to the database and return a set of rows. Statements that don't return rows must use `Exec()` function.

```go
var (
    id int 
    name string
)

rows, err := db.Query("SELECT id, name from users where id = ?", 1)
if err != nil {
    log.Fatal(err)
}
defer rows.Close() // Important to close! This is no-op if already closed

// Iterate rows. Make sure you don't skip/exit the loop early 
// or the connection remains open. Don't defer inside loop also.
for rows.Next() {
    err := rows.Scan(&id, &name)
    if err != nil {
        log.Fatal(err)
    }
    log.Println(id, name)
}

err = rows.Err() // check at end of the loop
if err != nil {
    log.Fatal(err)
}
```

When you iterate over rows and scan them into variables, **Go converts the type for you!** If you have a column that is a string type, ie: `varchar` but you know they are ints, just pass in an `int` in your `Scan()` and Go will call `strconv.ParseInt()`. This is the idiomatic Go way. Awesome!

If your query returns at most one row, you can use a shortcut:

```go
var name string 
err = db.QueryRow("SELECT name from users where id = ?" 1).Scan(&name)
if err != nil {
    log.Fatal(err)
}
fmt.Println(name)
```

## Modify Data 

Use `Exec()` to `INSERT`, `UPDATE`, `DELETE`. Do not use `db.Query` to execute these statements, otherwise the connection will never be released.

```go 
_, err := db.Exec("DELETE FROM users")
```

Using Prepared Statements

```go
stmt, err := db.Prepare("INSERT INTO squareNum VALUES( ?, ? )")
if err != nil {
    panic(err.Error()) // don't do this! just an example
}
defer stmt.Close()

for i := 0; i < 25; i++ {
    _, err = stmt.Exec(i, (i * i)) // insert tuples
    if err != nil {
        panel(err.Error())
    }
}
```

## NULL

**Nulls can lead to a lot of ugly code, so if you can avoid them - do so.** You will have to use a special type from the package to handle them. For example:

```go 
for rows.Next() {
    var s sql.NullString 
    err := rows.Scan(&s)
    if s.Valid { } else { }
}
```

Note that there is another workaround in the database layer to work with NULLs.

```go
rows, err := db.Query(`
	SELECT name, COALESCE(other_field, '') as other_field WHERE id = ?
`, 42)

for rows.Next() {
	err := rows.Scan(&name, &otherField)
	// If `other_field` was NULL, `otherField` is now an empty string. This works with other data types as well.
}
```

There's another work around mentioned in [go-sql-driver wiki](https://github.com/go-sql-driver/mysql/wiki/Examples#ignoring-null-values).

## Full Example 

Check out this example from [thewhitetulip.gitbooks.io](https://thewhitetulip.gitbooks.io/webapp-with-golang-anti-textbook/content/manuscript/2.3example.html). Also, here is the [full project](https://github.com/thewhitetulip/Tasks) the author references.

```go 
package main

import (
    "database/sql"
    "fmt"
    "log"
    "net/http"
    "time"

    _ "github.com/mattn/go-sqlite3"
)

var database *sql.DB
var err error

//Task is the struct used to identify tasks
type Task struct {
    Id      int
    Title   string
    Content string
    Created string
}

//Context is the struct passed to templates
type Context struct {
    Tasks      []Task
    Navigation string
    Search     string
    Message    string
}

func init() {
    database, err = sql.Open("sqlite3", "./tasks.db")
    if err != nil {
        fmt.Println(err)
    }
}

func main() {
    http.HandleFunc("/", ShowAllTasksFunc)
    http.HandleFunc("/add/", AddTaskFunc)
    fmt.Println("running on 8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

//ShowAllTasksFunc is used to handle the "/" URL which is the default one
func ShowAllTasksFunc(w http.ResponseWriter, r *http.Request) {
    if r.Method == "GET" {
        context := GetTasks() //true when you want non deleted notes
        w.Write([]byte(context.Tasks[0].Title))
    } else {
        http.Redirect(w, r, "/", http.StatusFound)
    }
}

func GetTasks() Context {
    var task []Task
    var context Context
    var TaskID int
    var TaskTitle string
    var TaskContent string
    var TaskCreated time.Time
    var getTasksql string

    getTasksql = "select id, title, content, created_date from task;"

    rows, err := database.Query(getTasksql)
    if err != nil {
        fmt.Println(err)
    }
    defer rows.Close()
    for rows.Next() {
        err := rows.Scan(&TaskID, &TaskTitle, &TaskContent, &TaskCreated)
        if err != nil {
            fmt.Println(err)
        }
        TaskCreated = TaskCreated.Local()
        a := Task{Id: TaskID, Title: TaskTitle, Content: TaskContent,
            Created: TaskCreated.Format(time.UnixDate)[0:20]}
        task = append(task, a)
    }
    context = Context{Tasks: task}
    return context
}

//AddTaskFunc is used to handle the addition of new task, "/add" URL
func AddTaskFunc(w http.ResponseWriter, r *http.Request) {
    title := "random title"
    content := "random content"
    truth := AddTask(title, content)
    if truth != nil {
        log.Fatal("Error adding task")
    }
    w.Write([]byte("Added task"))
}

//AddTask is used to add the task in the database
func AddTask(title, content string) error {
    query:="insert into task(title, content, created_date, last_modified_at)\ 
    values(?,?,datetime(), datetime())"
    restoreSQL, err := database.Prepare(query)
    if err != nil {
        fmt.Println(err)
    }
    tx, err := database.Begin()
    _, err = tx.Stmt(restoreSQL).Exec(title, content)
    if err != nil {
        fmt.Println(err)
        tx.Rollback()
    } else {
        log.Print("insert successful")
        tx.Commit()
    }
    return err
}
```

## References 

- [https://github.com/thewhitetulip/web-dev-golang-anti-textbook](https://github.com/thewhitetulip/web-dev-golang-anti-textbook)
- [https://github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
- [http://go-database-sql.org/](http://go-database-sql.org/)
