---
date: "2021-03-11"
title: "Concurrent map in Go"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

From [golang faq](https://golang.org/doc/faq#atomic_maps) 

{{< rawhtml >}}
<h4>Why are map operations not defined to be atomic?</h4>
<div class="boxtextoverflow">
After long discussion it was decided that the typical use of maps did not require safe access from multiple goroutines, and in those cases where it did, the map was probably part of some larger data structure or computation that was already synchronized. Therefore requiring that all map operations grab a mutex would slow down most programs and add safety to few. This was not an easy decision, however, since it means uncontrolled map access can crash the program. 
<br></br>
The language does not preclude atomic map updates. When required, such as when hosting an untrusted program, the implementation could interlock map access.
<br></br>
Map access is unsafe only when updates are occurring. As long as all goroutines are only reading—looking up elements in the map, including iterating through it using a for range loop—and not changing the map by assigning to elements or doing deletions, it is safe for them to access the map concurrently without synchronization.
<br></br>
As an aid to correct map use, some implementations of the language contain a special check that automatically reports at run time when a map is modified unsafely by concurrent execution. 
</div>

{{< /rawhtml >}}

Other languages such as Java provides map implementation that supports concurrent read/write. 

Time will come when we will need to design apps that requires map to be read in different part of the app requiring concurrent read and write. Conceptually to implement a map that supports concurrent read/write is done by introducing some sort of synchronisation. This article goes through few ways on what to use if an app require to have concurrent read/write map functionality.

## concurrent-map

[concurrent-map](https://github.com/orcaman/concurrent-map) is quite an old project last updated 2months ago. In terms of implementation it still works.

Internally the framework is using struct called **_ConcurrentMapShared_**


{{< highlight go >}}
type ConcurrentMapShared struct {
	items        map[string]interface{}
	sync.RWMutex // Read Write mutex, guards access to internal map.
}
{{</highlight>}}

Internally the library creates 32 different buckets of the **_ConcurrentMapShared_** type and use sharding when it does reading/writing values.

Using the library is very simple in app as shown below.

{{< highlight go >}}
func main() {
	m := New()
	elephant := Animal{"elephant"}
	monkey := Animal{"monkey"}

	m.Set("elephant", elephant)
	m.Set("monkey", monkey)
}
{{</highlight>}}

## ristretto

Another option is to use cache library. Most cache libraries stores data in a map as follows

{{< highlight go >}}
map[string]interface{}
{{</highlight>}}

For this article we going to take a look at [Ristretto](https://github.com/dgraph-io/ristretto) project. According to the project

{{< rawhtml >}}
<div class="boxtextoverflow">
Ristretto is a fast, concurrent cache library built with a focus on performance and correctness.
</div>
{{< /rawhtml >}}

The library is not built specifically to address map based concurrency issue. However, if we look at the implementation it can pretty much do what a concurrent map can.

The function calls are not exactly the same like a map operation, nevertheless it is quite close.

{{< highlight go >}}
package main

import (
	"fmt"
	"github.com/dgraph-io/ristretto"
	"os"
	"time"
)

func main() {
	c, err := ristretto.NewCache(&ristretto.Config{
		NumCounters:        10,
		MaxCost:            1000,
		BufferItems:        64,
		IgnoreInternalCost: true,
	})

	if err != nil {
		os.Exit(1)
	}

	valid := c.Set("test", "value", 1000)

	if valid {
		fmt.Println("Stored successfully")
	}

	time.Sleep(100 * time.Millisecond)
	s, found := c.Get("test")

	if !found {
		fmt.Println("Get not successful")
	} else {
		str := fmt.Sprintf("%v", s)
		fmt.Println(str)
	}
}
{{</highlight>}}

Cache library are designed to be asynchronous in nature as there is no hurry for the data to be made available directly. Unlike map when you store something you will get it back directly. Hence, the `time.sleep(..)` in the code. If the `time.sleep` code is removed the app would never find the key.

This kind of cache library are very focused on performance and are memory efficient which helped if the app is going to store big sets of data. As a bonus concurrency is built into the library which is something the app does not have to worry about.

Good [reading material](https://dgraph.io/blog/post/introducing-ristretto-high-perf-go-cache/) about the project including performance comparison with other open source cache libraries.
