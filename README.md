# go-simple-cache

go-simple-cache is an in-memory key:value store/cache similar to memcached that is suitable for applications running on a single machine. Its major advantage is that, being essentially a thread-safe map[string]interface{} with expiration times, it doesn't need to serialize or transmit its contents over the network.

Any object can be stored, for a given duration or forever, and the cache can be safely used by multiple goroutines.

Source code originally forked from: 
https://github.com/patrickmn/go-cache

This forked version of go-cache is a simplified version of the original. This version 
has the following changes from the original:

1. expired items stay in the cache, but a user provided callback function is executed when the cache expires
2. cache expiration is applied to the entire cache, rather than to individual keys
3. removed increment/decrement feature
4. removed ability to persist cache to disk
5. removed auto eviction of expired items

### Use Case

This version of the library is ideal as a simple lookup cache that is populated by a relatively static list 
of items that have a limited life or that change infrequently. For example, you fetch a list of records from 
a lookup table in a database and put them in the cache with a timeout of 24hrs. The data in the database changes 
infrequently, which justifies the 24hr timeout, and performing the lookup from the cache is faster than going 
back to the database. When the 24hr timeout is reached, a callback function is executed and the cache is refreshed 
with the latest data from the database. 

### Installation

`go get github.com/jfarleyx/go-simple-cache`

### Usage

```go
import (
	"fmt"
	"github.com/jfarleyx/go-simple-cache"
	"time"
)

func main() {
	// Create a cache with a default expiration time of 5 hours
	c := cache.New(5*time.Hour)

	// Provide a callback function that is called when the cache expires
	myfunc := func() {
		// e.g. fetch current data and refill cache
	}
	c.cache.OnExpired(myfunc)

	// Set the value of the key "foo" to "bar"
	c.Set("foo", "bar")

	// Get the string associated with the key "foo" from the cache
	foo, found := c.Get("foo")
	if found {
		fmt.Println(foo)
	}

	// Since Go is statically typed, and cache values can be anything, type
	// assertion is needed when values are being passed to functions that don't
	// take arbitrary types, (i.e. interface{}). The simplest way to do this for
	// values which will only be used once--e.g. for passing to another
	// function--is:
	foo, found := c.Get("foo")
	if found {
		MyFunction(foo.(string))
	}

	// This gets tedious if the value is used several times in the same function.
	// You might do either of the following instead:
	if x, found := c.Get("foo"); found {
		foo := x.(string)
		// ...
	}
	// or
	var foo string
	if x, found := c.Get("foo"); found {
		foo = x.(string)
	}
	// ...
	// foo can then be passed around freely as a string

	// Want performance? Store pointers!
	c.Set("foo", &MyStruct)
	if x, found := c.Get("foo"); found {
		foo := x.(*MyStruct)
			// ...
	}
}
```
