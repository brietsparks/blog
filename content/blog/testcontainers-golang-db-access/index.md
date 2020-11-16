---
title: Testing Golang database access functions with testcontainers
date: "2020-10-15T15:00:00.000Z"
---

[Testcontainers-go](https://github.com/testcontainers/testcontainers-go) is a library that allows your test code to create, use, and destroy Docker containers. This is helpful when tests need to run against a stateful application such as a database. This article demonstrates basic usage of testcontainers to test functions that run against a postgres database.

First, I'll walk through a tutorial of building a simple database repository layer with accompanying test code. Afterwards, I'll provide a couple next steps for adaptation usage to a real world application. 
The source code of both:
- [Tutorial](https://github.com/brietsparks/testcontainer-go-pract/tree/master/tutorial)
- [Next steps](https://github.com/brietsparks/testcontainer-go-pract)

# Tutorial

## Application code

First lets write the application code, which has a domain entity and two database access functions.

```go
package main

import (
	"context"
	"database/sql"
)

type Item struct {
	Id          string
	Description string
}

type ItemRepository struct {
	db *sql.DB
}

func (r *ItemRepository) CreateItem(ctx context.Context, description string) (*Item, error) {
	// insert into items(id, description) values ($1, $2) 
}

func (r *ItemRepository) GetItem(ctx context.Context, id string) (*Item, error) {
	// select id, description from items where id = $1
}
```

For brevity's sake I've omitted the implementation details, but they can be found in the [source](https://github.com/brietsparks/testcontainer-go-pract/blob/master/tutorial/main.go).

## Test code

We have two testable database access functions: `CreateItem` and `GetItem`. To test them, we need to:
 1. create and connect to a postgres testcontainer
 2. connect to the database in the container
 3. run table migrations in the database
 4. run the tests
 5. close the database
 6. destroy the container
 
There are three dependencies here: container, database, and migration. To take care of these, we can write some helper functions.  
 
This one creates and connects to the container and database and returns handles to both.

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
	"github.com/docker/go-connections/nat"
	"github.com/testcontainers/testcontainers-go"
	"github.com/testcontainers/testcontainers-go/wait"
	"log"
	"time"
)

func CreateTestContainer(ctx context.Context, dbname string) (testcontainers.Container, *sql.DB, error) {
	var env = map[string]string{
		"POSTGRES_PASSWORD": "password",
		"POSTGRES_USER":     "postgres",
		"POSTGRES_DB":       dbname,
	}
	var port = "5432/tcp"
	dbURL := func(port nat.Port) string {
		return  fmt.Sprintf("postgres://postgres:password@localhost:%s/%s?sslmode=disable", port.Port(), dbname)
	}

	req := testcontainers.GenericContainerRequest{
		ContainerRequest: testcontainers.ContainerRequest{
			Image:        "postgres:latest",
			ExposedPorts: []string{port},
			Cmd:          []string{"postgres", "-c", "fsync=off"},
			Env:          env,
			WaitingFor:   wait.ForSQL(nat.Port(port), "postgres", dbURL).Timeout(time.Second*5),
		},
		Started: true,
	}
	container, err := testcontainers.GenericContainer(ctx, req)
	if err != nil {
		return container, nil, fmt.Errorf("failed to start container: %s", err)
	}

	mappedPort, err := container.MappedPort(ctx, nat.Port(port))
	if err != nil {
		return container, nil, fmt.Errorf("failed to get container external port: %s", err)
	}

	log.Println("postgres container ready and running at port: ", mappedPort)

	url := fmt.Sprintf("postgres://postgres:password@localhost:%s/%s?sslmode=disable", mappedPort.Port(), dbname)
	db, err := sql.Open("postgres", url)
	if err != nil {
		return container, db, fmt.Errorf("failed to establish database connection: %s", err)
	}

	return container, db, nil
}

```

Migrations can be done with a library such as [golang-migrate](https://github.com/golang-migrate/migrate). Here is a helper function that takes a pg connection, migration files, and returns an object that can apply the migrations.

```go
package main

import (
	"database/sql"
	"github.com/golang-migrate/migrate/v4"
	"github.com/golang-migrate/migrate/v4/database/postgres"
	_ "github.com/golang-migrate/migrate/v4/source/file"
	_ "github.com/lib/pq"
	"log"
	"path/filepath"
	"runtime"
)

func NewPgMigrator(db *sql.DB) (*migrate.Migrate, error) {
	_, path, _, ok := runtime.Caller(0)
	if !ok {
		log.Fatalf("failed to get path")
	}

	sourceUrl := "file://" + filepath.Dir(path) + "/migrations"

	driver, err := postgres.WithInstance(db, &postgres.Config{})

	if err != nil {
		log.Fatalf("failed to create migrator driver: %s", err)
	}

	m, err := migrate.NewWithDatabaseInstance(sourceUrl, "postgres", driver)

	return m, err
}
``` 

The helper functions (container+database, migration) written thus far allow us to set up and tear down a containerized database. Let's write the test.

```go
package main

import (
	"context"
	"testing"
)

func TestExample(t *testing.T)  {
	ctx := context.Background()

	// container and database
	container, db, err := CreateTestContainer(ctx, "testdb")
	if err != nil {
		t.Fatal(err)
	}
	defer db.Close()
	defer container.Terminate(ctx)

	// migration
	mig, err := NewPgMigrator(db)
	if err != nil {
		t.Fatal(err)
	}

	err = mig.Up()
	if err != nil {
		t.Fatal(err)
	}

	// test
	r :=  &ItemRepository{db}
	created, err := r.CreateItem(ctx, "desc")
	if err != nil {
		t.Errorf("failed to create item: %s", err)
	}
	retrieved, err := r.GetItem(ctx, created.Id)
	if err != nil {
		t.Errorf("failed to retrieve item: %s", err)
	}
	if created.Id != retrieved.Id {
		t.Errorf("created.Id (%s) != retrieved.Id (%s)", created.Id, retrieved.Id)
	}
	if created.Description != retrieved.Description {
		t.Errorf("created.Description != retrieved.Description (%s != %s)", created.Description, retrieved.Description)
	}
}
```

Running `go test -v` will perform all the steps required to test the database access functions against a containerized database. With a testcontainer, you are afforded the convenience of testing against a live database without having to manage it manually. The final product of this tutorial has [source code which can be found here](https://github.com/brietsparks/testcontainer-go-pract/tree/master/tutorial).

# Next Steps

For next steps, you would probably want to abstract the setup and teardown. The exact approach could vary depending on whether your tests use a framework that provides setup/teardown conventions. Also, if you can implement package-level setup/teardown, your testcontainer and database can be reused across multiple files and won't need to be recreated on a per-file basis.

My parting note is to leave you with an example of how to implement the next steps, available at [the root of the source code repo](https://github.com/brietsparks/testcontainer-go-pract). The code is refactored into the directories `/db`, `/entities`, and `/repositories`. I have abstracted the setup and teardown into a package-level `TestMain` function. Additionally, I expanded the application code to include a `list` entity and a many-to-many relationship between items and lists. I hope this can be useful to anyone looking to learn and incorporate testcontainers in their go codebase. 
