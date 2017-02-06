---
layout: post
title: Go and MongoDB simple example
permalink: blog/go-mongodb-simple-example/
comments: True
excerpt_separator: <!--more-->
---

## Install and Start MongoDB

### Install MongoDB on Mac 

```sh
brew update
brew install mongodb
```

### Create Data Directory

Before you start MongoDB for the first time, create the directory where 
`mongod` process will write. By default this is in **/data/db**. If you 
create a different one, specify that in the `dbpath` option when starting a 
`mongod` process.

### Setup Permissions

Before running `mongod`, make sure the user account that will run it has read 
and write permissions for the directory.

```sh
# change owner of everything in /data to current user
sudo chown -R $USER /data/db

# allow user to have read/write access
sudo chmod -R u+rw /data/db
```

### Run MongoDB Server

`mongod`

That's it! However, if you have changed the directory from the default, you will
have to specify it, like this: `mongod --dbpath /path/to/dir`

### Connect to the Shell

Typically, you'd connect through your application, but if you'd like to connect
to the MongoDB server you just ran, execute the MongoDB shell client:

`mongo`

## Go

Get mongo drivers for go.

`go get gopkg.in/mgo.v2`

Sample CRUD

```go
package main

import (
	"fmt"
	"time"

	"strconv"

	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

type Person struct {
	ID        bson.ObjectId `bson:"_id,omitempty"`
	Name      string
	Phone     string
	Timestamp time.Time
}

var isDropMe = true

func main() {
	Host := []string{
		"127.0.0.1:27017",
		// more replica set addrs...
	}
	const (
		Username = "YOUR_USERNAME"
		Password = "YOUR_PASS"
		Database = "YOUR_DB"
	)
	session, err := mgo.DialWithInfo(&mgo.DialInfo{
		Addrs:    Host,
		// Username: Username,
		// Password: Password,
		// Database: Database,
	})
	if err != nil {
		panic(err)
	}
	defer session.Close()

	session.SetMode(mgo.Strong, true)

	if isDropMe {
		err = session.DB("test").DropDatabase()
		if err != nil {
			panic(err)
		}
	}

	// Collection People
	c := session.DB("test").C("people")

	// Create Index
	index := mgo.Index{
		Key:        []string{"name", "phone"},
		Unique:     true,
		DropDups:   true,
		Background: true,
		Sparse:     true,
	}
	err = c.EnsureIndex(index)
	if err != nil {
		panic(err)
	}

	// Insert
	err = c.Insert(
		&Person{Name: "Ale", Phone: "+55 55 9090 8989", Timestamp: time.Now()},
		&Person{Name: "Cla", Phone: "+77 77 9090 8989", Timestamp: time.Now()},
	)
	if err != nil {
		panic(err)
	}

	// Bulk Insert
	var data []interface{}
	for i := 0; i < 10000; i++ {
		data = append(data, Person{Name: "Ale" + strconv.Itoa(i), Phone: "+55 55 9090 8989", Timestamp: time.Now()})
	}
	bulk := c.Bulk()
	bulk.Unordered()
	bulk.Insert(data...)
	_, err = bulk.Run()
	if err != nil {
		panic(err)
	}

	// Query All
	var results []Person
	err = c.Find(bson.M{"name": "Ale"}).Sort("-timestamp").All(&results)
	if err != nil {
		panic(err)
	}
	fmt.Println("Query All: ", results)

	// Get All
	var allPeople []Person
	if err := c.Find(nil).Sort("-timestamp").All(&allPeople); err != nil {
		panic(err)
	}
	fmt.Println("Get All", results)

	// Find One
	result := Person{}
	if err := c.Find(bson.M{"name": "Ale"}).Select(bson.M{"phone": 0}).One(&result); err != nil {
		panic(err)
	}
	fmt.Println("Phone", result)

	// Update
	selector := bson.M{"name": "Ale"}
	updator := bson.M{"$set": bson.M{"phone": "86 44 9999 8888", "timestamp": time.Now()}}
	if err := c.Update(selector, updator); err != nil {
		panic(err)
	}

	// Remove
	if err := c.Find(bson.M{"name": "Cla"}).One(&result); err != nil {
		panic(err)
	}
	if err := c.Remove(bson.M{"_id": result.ID}); err != nil {
		panic(err)
	}

	// Count
	count, err := c.Count()
	if err != nil {
		panic(err)
	}
	fmt.Println("Total Documents:", count)
}
```
