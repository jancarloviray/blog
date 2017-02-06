---
layout: post
title: Go and MongoDB simple example
permalink: blog/go-mongodb-simple-example/
comments: True
excerpt_separator: <!--more-->
---

Both Golang and MongoDB are a perfect pair for rapid application development and
building things for scale. They both have native features that allows you to
focus on your application. When choosing a database, make sure that MongoDB fits 
your requirements as it is not a one-size-fits all solution. Read the official 
docs on its design and use cases. In this post, we'll go from Installation to 
building a CRUD.

<!--more-->

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

**NOTE: this section is still in development**

Get mongo drivers for go.

`go get gopkg.in/mgo.v2`

Sample CRUD

```go
package main

import (
	"fmt"
	"time"

	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

type Game struct {
	Winner       string    `bson:"winner"`
	OfficialGame bool      `bson:"official_game"`
	Location     string    `bson:"location"`
	StartTime    time.Time `bson:"start"`
	EndTime      time.Time `bson:"end"`
	Players      []Player  `bson:"players"`
}

type Player struct {
	Name   string    `bson:"name"`
	Decks  [2]string `bson:"decks"`
	Points uint8     `bson:"points"`
	Place  uint8     `bson:"place"`
}

func NewPlayer(name, firstDeck, secondDeck string, points, place uint8) Player {
	return Player{
		Name:   name,
		Decks:  [2]string{firstDeck, secondDeck},
		Points: points,
		Place:  place,
	}
}

var isDropMe = true

func main() {
	Host := []string{
		"127.0.0.1:27017",
		// replica set addrs...
	}
	const (
		Username   = "YOUR_USERNAME"
		Password   = "YOUR_PASS"
		Database   = "YOUR_DB"
		Collection = "YOUR_COLLECTION"
	)
	session, err := mgo.DialWithInfo(&mgo.DialInfo{
		Addrs: Host,
		// Username: Username,
		// Password: Password,
		// Database: Database,
		// DialServer: func(addr *mgo.ServerAddr) (net.Conn, error) {
		// 	return tls.Dial("tcp", addr.String(), &tls.Config{})
		// },
	})
	if err != nil {
		panic(err)
	}
	defer session.Close()

	game := Game{
		Winner:       "Dave",
		OfficialGame: true,
		Location:     "Austin",
		StartTime:    time.Date(2015, time.February, 12, 04, 11, 0, 0, time.UTC),
		EndTime:      time.Now(),
		Players: []Player{
			NewPlayer("Dave", "Wizards", "Steampunk", 21, 1),
			NewPlayer("Javier", "Zombies", "Ghosts", 18, 2),
			NewPlayer("George", "Aliens", "Dinosaurs", 17, 3),
			NewPlayer("Seth", "Spies", "Leprechauns", 10, 4),
		},
	}

	if isDropMe {
		err = session.DB("test").DropDatabase()
		if err != nil {
			panic(err)
		}
	}

	// Collection
	c := session.DB(Database).C(Collection)

	// Insert
	if err := c.Insert(game); err != nil {
		panic(err)
	}

	// Find and Count
	player := "Dave"
	gamesWon, err := c.Find(bson.M{"winner": player}).Count()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s has won %d games.\n", player, gamesWon)

	// Find One (with Projection)
	var result Game
	err = c.Find(bson.M{"winner": player, "location": "Austin"}).Select(bson.M{"official_game": 1}).One(&result)
	if err != nil {
		panic(err)
	}
	fmt.Println("Is game in Austin Official?", result.OfficialGame)

	// Find All
	var games []Game
	err = c.Find(nil).Sort("-start").All(&games)
	if err != nil {
		panic(err)
	}
	fmt.Println("Number of Games", len(games))

	// Update
	newPlayer := "John"
	selector := bson.M{"winner": player}
	updator := bson.M{"$set": bson.M{"winner": newPlayer}}
	if err := c.Update(selector, updator); err != nil {
		panic(err)
	}

	// Update All
	info, err := c.UpdateAll(selector, updator)
	if err != nil {
		panic(err)
	}
	fmt.Println("Updated", info.Updated)

	// Remove
	info, err = c.RemoveAll(bson.M{"winner": newPlayer})
	if err != nil {
		panic(err)
	}
	fmt.Println("Removed", info.Removed)
}
```
