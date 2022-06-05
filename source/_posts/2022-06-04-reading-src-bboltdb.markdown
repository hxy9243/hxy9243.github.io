layout: post
title: 'Reading BBoltDB'
date: 2022-06-04 22:00
comments: true
categories: "CodeReading"
tags: ['Learning', 'Golang', 'DistributedSystems', 'Database']
---

By trying to understand databases in general, I've recently dived into the source code
of [BBoltDB](https://github.com/etcd-io/bbolt) for a deeper understanding of its implementation.
It's a fairly small, nifty local KV store for source-code reading and studying, especially you're a Golang
programmer. At the same time it's widely used and tested, embeded as the powerful storage engine for 
systems like [Etcd](https://github.com/etcd-io/etcd) and [Consul](https://github.com/hashicorp/consul). 
Though it's only backed by one file, it's capable of serving data size up to 
[TB level](https://github.com/etcd-io/bbolt/#project-status).

Below is my notes for the very quick reading from its source code, hopefully providing anyone 
interested in data systems a quick glimpse of what's under the hood. It's not meant as an exhaustive
and deep analysis.

<!--more-->

# Overview

BBoltDB is a local, embedded database backed by a single file `mmap`ped into the memory,
meaning it's backed by only one file, and it's meant to be built into other data systems
as the storage engine.
The data file is locked by the application, with only one process access at a time.

There's no write-ahead-log. Storage engine uses B+ tree for managing storage.
And all operations are written to the database file. 

# User Interface

Reading source code from v1.3.5 release:

[https://github.com/etcd-io/bbolt/tree/v1.3.5](https://github.com/etcd-io/bbolt/tree/v1.3.5)

BBoltDB is a key-value store, so data format is as simple as key-value pairs, organized into "Buckets".
Buckets provides namespace for a range of key-value pairs, and they can be nested.

Using BBoltDB in your application is as simple as importing its library in yoru Golang project,
and opening a database: See more at [BBoltDB](https://github.com/etcd-io/bbolt/#opening-a-database).

```go
import (
	"log"

	bolt "go.etcd.io/bbolt"
)

func main() {
	// Open the my.db data file in your current directory.
	// It will be created if it doesn't exist.
	db, err := bolt.Open("my.db", 0600, nil)
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	...
}
```

DB operations are carried out in transactions. For example, a Read-write transaction:

```go
err := db.Update(func(tx *bolt.Tx) error {
	...
	return nil
})
```

Read-only transactions:

```go
err := db.View(func(tx *bolt.Tx) error {
	...
	return nil
})
```

See more at:

[https://github.com/etcd-io/bbolt#read-write-transactions](https://github.com/etcd-io/bbolt#read-write-transactions)

And for more features like range query, auto-incrementing: 

[https://github.com/etcd-io/bbolt#autoincrementing-integer-for-the-bucket](https://github.com/etcd-io/bbolt#autoincrementing-integer-for-the-bucket)

# Opening The Database

BBoltDB organize `mmap`ped memory as `page`s. The first two pages are saving metadata for the database,
like version info, configurations, etc. The third page saves the `freelist` of pages, which is
the addresses of memory that's not allocated yet. The rest of the pages are 

See source code from [https://github.com/etcd-io/bbolt/blob/v1.3.5/db.go#L178](https://github.com/etcd-io/bbolt/blob/v1.3.5/db.go#L178).

When opening the database process, BBoltDB carries out the following major steps:

- Opens the file, and locks file (`flock`) access to single process.
- `mmap` the file into memory.
- Read in the meta pages from the memory, construct meta info if not exists. Meta page includes the info:
    - magic number.
    - version number.
    - pagesize.
    - freelist: which keeps a list of free pages for lookup/allocate/recycle.

# Managing Memory

BBoltDB manages the mapped in memory by pages of 4KB size. And it keeps all the pages besides the metapages in a freelist for allocation. Freelist can be of default array type, or a map type by option.

- It initializes meta pages in total at the time of the first start. It contains metadata information about the database, including the freelist info.
  
  See at [https://github.com/etcd-io/bbolt/blob/v1.3.5/db.go#L426](https://github.com/etcd-io/bbolt/blob/v1.3.5/db.go#L426).

- The `freelist` is initialized either by loading the metadata from the page, 
or constructed by scanning through all the free pages in the DB and get their ids.
- Allocation: uses `sync.Pool` for one page, or allocates memory for longer contiguous pages. 
All allocation is backed by a page (could be a multiple of page size) with a page id.

  By default, `freelist` of pages uses a list implementation, and multiple pages of allocation
  is guaranteed to be contiguous.

  Implementation is in `arrayAllocate()`: https://github.com/etcd-io/bbolt/blob/v1.3.5/freelist.go#L107
- Once a page is allocated, it’s serialized into the memory as a `node`, which represents the node in a B+tree.

# B+Tree

B+Tree is the higher level abstraction from the memory pages in BBoltDB. The database manages
key spaces by B+Tree and its nodes. The `node`s are initialized from memory pages from the `freelist`, 
and organized as a B+Tree. Each `node` has its own index key, with a list 
of `inode`s (for internal nodes) that saves the actual keys and values as bytes.

See https://github.com/etcd-io/bbolt/blob/v1.3.5/node.go

```go
// node represents an in-memory, deserialized page.
type node struct {
	bucket     *Bucket
	isLeaf     bool
	unbalanced bool
	spilled    bool
	key        []byte
	pgid       pgid
	parent     *node
	children   nodes
	inodes     inodes
}

// inode represents an internal node inside of a node.
// It can be used to point to elements in a page or point
// to an element which hasn't been added to a page yet.
type inode struct {
	flags uint32
	pgid  pgid
	key   []byte
	value []byte
}

type inodes []inode
```

The `node.go` file contains the whole B+Tree implementation, from indexing, reading/writing, 
spilling, splitting node, rebalancing, to deletion.

# Transaction

Each transaction will write the node data to the actual memory that's backed by the page. And if
there are more data than the configured size of the node, the node will spill data to a new
node, and recursively split the parents if necessary.

All R/W operations in BBoltDB are managed by transactions:

  - Create transaction with `tx = db.Begin()`.
  - Perform actual View/Update.
  - Possible spill of nodes, which splits tree nodes.
  - Possible rebalance of tree nodes.
  - Write the dirty pages to the mmapped memory region and syncs to disk in `tx.write()`. See at https://github.com/etcd-io/bbolt/blob/v1.3.5/tx.go#L514.
  - In case of error, Tx.Rollback().
  - Commit transaction with `tx.Commit()`.

Example from user's perspective, from the README for [BBoltDB](https://github.com/etcd-io/bbolt/tree/v1.3.5#managing-transactions-manually):

```go
// Start a writable transaction.
tx, err := db.Begin(true)
if err != nil {
    return err
}
defer tx.Rollback()

// Use the transaction...
_, err := tx.CreateBucket([]byte("MyBucket"))
if err != nil {
    return err
}

// Commit the transaction and check for error.
if err := tx.Commit(); err != nil {
    return err
}
```

# Conclusion

BBoltDB has a simple (compared to other database systems) yet powerful capability 
supporting the operations of 
huge volume of data in many well-tested online systems in many companies. Reading the source
code provides a hint of insights on how a B+tree based simple Key-Value store works,
yet I've only scratched the surface. If you're willing to maintain/extend
BBoltDB, this could maybe be a first step to understanding its detailed design.