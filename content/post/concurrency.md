---
date: "2024-04-19"
title: "Go Concurrency Notes"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---
# Table of Contents  
- [Restricting Writers](#restricting-writers)
- [Architecture](#architecture)
  - [A1](#a1)
  - [A2](#a2)


## Restricting Writers


### Scenario

Building library that perform expensive I/O operation (eg: writing to a file). The library uses a writer code to write the data and it limits the number of writers to the number of CPU cores. This will safeguard the library write operation to work correctly and consistently in the event if the caller is calling the writer in an uncontrollable manner. Imagine an app that calls the writer in 100 goroutines in the hope it speeds up the write operation, which in reality does not happen as I/O operation are bound to the speed of the I/O peripherals.


### Solution

The caller calls the writer function in a normal way. This will works normal in a sequential manner, the number of channels does not make any difference. All 10 calling will be processed.

```
package main

import (
	"log"
	"time"
)

type storage struct {
	workersLimitCh chan struct{}
}

func (s *storage) InsertRows(i int) error {
	insert := func() error {
		j := i
		defer func() { <-s.workersLimitCh }()
		log.Println("Sleeping...", j)
		time.Sleep(2 * time.Second)
		return nil
	}

	log.Println("Selecting...")

	select {
	case s.workersLimitCh <- struct{}{}:
		log.Println("insert()....")
		return insert()
	default:
		log.Println("defaulting...")
	}

	return nil
}

func main() {
	s := storage{}
	s.workersLimitCh = make(chan struct{}, 10)
	for i := 0; i <= 10; i++ {
		s.InsertRows(i)
	}
	for {
	}
}
```




The caller spins up goroutine to call the writer function. Even when the client spins up 10 goroutine to call the function, the function will limit it to only the number of channels available (which is 4). The rest of the 6 goroutine will be ignored.

```
package main

import (
	"log"
	"time"
)

type storage struct {
	workersLimitCh chan struct{}
}

func (s *storage) InsertRows(i int) error {
	insert := func() error {
		j := i
		defer func() { <-s.workersLimitCh }()
		log.Println("Sleeping...", j)
		time.Sleep(2 * time.Second)
		log.Println("Done Sleeping...", j)

		return nil
	}

	log.Println("Selecting...")

	select {
	case s.workersLimitCh <- struct{}{}:
		log.Println("insert()....")
		return insert()
	default:
		log.Println("defaulting...")
	}

	return nil
}

func main() {
	s := storage{}
	s.workersLimitCh = make(chan struct{}, 4)
	for i := 0; i <= 10; i++ {
		go func(i int) {
			s.InsertRows(i)

		}(i)
	}
	for {
	}
}
```