---
date: "2019-12-02"
title: "Concurrency Design Pattern in Go"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---

<h1>Design Patterns</h1>


<h3>Concurreny Design Pattern</h3>

* Resources 
 * https://github.com/golang/go/wiki/LearnConcurrency
 * https://golang.org/ref/spec#Send_statements

Both the channel and the value expression are evaluated before communication begins. Communication blocks until the send can proceed. A send on an unbuffered channel can proceed if a receiver is ready. A send on a buffered channel can proceed if there is room in the buffer. A send on a closed channel proceeds by causing a run-time panic. Communication on nil channels can never proceed, a select with only nil channels and no default case blocks forever. 
{{< highlight go >}}
repeatFn := func(
    done <-chan interface{},
    fn func() interface{},
) <-chan interface{} {
    valueStream := make(chan interface{})

    go func() {
        defer close(valueStream)

        for {
            select {
            case <-done:
                return
            case valueStream <- fn():
            }
        }
    }()

    return valueStream
}
{{< /highlight >}}

One of repeatFn(..) argument is to receive 'done' variable that the repeatFn(..) will use as a flag to exit from the function
The value of the receive operation <-ch is the value received from the channel ch. The expression blocks until a value is available. Receiving from a nil channel blocks forever. A receive operation on a closed channel can always proceed immediately, yielding the element type's zero value after any previously sent values have been received.

<h3>Pipeline</h3>

The cascading effect of closing channel (from  close(done)) will ensure that all the other channels inside generator(..), add(..) and multiply (..) will also closed.
{{< highlight go >}}
func main(){
    generator := func(done <-chan interface{}, integers ...int) <-chan int {
        intStream := make(chan int)
        go func() {
            defer close(intStream)
            for _, i := range integers {
                select {
                case <-done:
                    return
                case intStream <- i:
                }
            }
        }()
        return intStream
    }

    multiply := func(
        done <-chan interface{},
        intStream <-chan int,
        multiplier int,
    ) <-chan int {
        multipliedStream := make(chan int)
        go func() {
            defer close(multipliedStream)
            for i := range intStream {
                select {
                case <-done:
                    return
                case multipliedStream <- i*multiplier:
                }
            }
        }()
        return multipliedStream
    }

    add := func(
        done <-chan interface{},
        intStream <-chan int,
        additive int,
    ) <-chan int {
        addedStream := make(chan int)
        go func() {
            defer close(addedStream)
            for i := range intStream {
                select {
                case <-done:
                    return
                case addedStream <- i+additive:
                }
            }
        }()
        return addedStream
    }

    done := make(chan interface{})
    defer close(done)
    intStream := generator(done, 1, 2, 3, 4)
    pipeline := multiply(done, add(done, multiply(done, intStream, 2), 1), 2)
    for v := range pipeline {
        fmt.Println(v)
    } 
}
{{< /highlight >}}

Resources:

* https://blog.golang.org/pipelines
    
<h3>Tee-Channel</h3>

Takes a single input channel and an arbitrary number of output channels and duplicates each input into every output. When the input channel is closed, all outputs channels are closed. The [eapache](https://github.com/eapache) project provide this kind of pattern. Following code snippet is the implementation.

{{< highlight go >}}
func tee(input SimpleOutChannel, outputs []SimpleInChannel, closeWhenDone bool) {
	cases := make([]reflect.SelectCase, len(outputs))
	for i := range cases {
		cases[i].Dir = reflect.SelectSend
	}
	for elem := range input.Out() {
		for i := range cases {
			cases[i].Chan = reflect.ValueOf(outputs[i].In())
			cases[i].Send = reflect.ValueOf(elem)
		}
		for _ = range cases {
			chosen, _, _ := reflect.Select(cases)
			cases[chosen].Chan = reflect.ValueOf(nil)
		}
	}
	if closeWhenDone {
		for i := range outputs {
			outputs[i].Close()
		}
	}
}
{{< /highlight >}}

The library uses Golang's internal [SelectCase](https://golang.org/pkg/reflect/#SelectCase) package to process multiple channels.

{{< highlight go >}}
type SelectCase struct {
    Dir  SelectDir // direction of case
    Chan Value     // channel to use (for send or receive)
    Send Value     // value to send (for send)
}
{{< /highlight >}}

The SelectCase structure is like a container of channels depending on the direction specified (Dir).

<h3>Fan-in-Fan-out channel</h3> 

This pattern collate results from goroutines that have been spawned to do background processing.

{{< highlight go >}}
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := fanIn(generator("Hello"), generator("Bye"))
    for i := 0; i < 10; i++ {
        fmt.Println(<- ch)
    }
}

// fanIn is itself a generator
func fanIn(ch1, ch2 <-chan string) <-chan string { // receives two read-only channels
    new_ch := make(chan string)
    go func() { for { new_ch <- <-ch1 } }() // launch two goroutine while loops to continuously pipe to new channel
    go func() { for { new_ch <- <-ch2 } }()
    return new_ch
}

func generator(msg string) <-chan string { // returns receive-only channel
    ch := make(chan string)
    go func() { // anonymous goroutine
        for i := 0; ; i++ {
            ch <- fmt.Sprintf("%s %d", msg, i)
            time.Sleep(time.Second)
        }
    }()
    return ch
}    
{{< /highlight >}}	
            
<h3>Bridge Channel</h3>

<h3>Context</h3>

Good for cancelling / terminating goroutine by propagation. In order for this to work effectively the context must be send as part of argument parameters to each of the goroutines. The Done() inside locale(..) method is called as the context.WithTimeout(..) got triggered after 1 second and it propagates all the way to the root context 
{{< highlight go >}}
func main() {
    var wg sync.WaitGroup
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    wg.Add(1)
    go func() {
        defer wg.Done()
        if err := printGreeting(ctx); err != nil {
            fmt.Printf("cannot print greeting: %v\n", err)
            cancel()
        }
    }()
    wg.Add(1)
    go func() {
        defer wg.Done()
        if err := printFarewell(ctx); err != nil {
            fmt.Printf("cannot print farewell: %v\n", err)
        }
    }()
    wg.Wait()
}

func printGreeting(ctx context.Context) error {
    greeting, err := genGreeting(ctx)
    if err != nil {
        return err
    }
    fmt.Printf("%s world!\n", greeting)
    return nil
}

func printFarewell(ctx context.Context) error {
    farewell, err := genFarewell(ctx)
    if err != nil {
        return err
    }
    fmt.Printf("%s world!\n", farewell)
    return nil
}

func genGreeting(ctx context.Context) (string, error) {
    ctx, cancel := context.WithTimeout(ctx, 1*time.Second)
    defer cancel()
    switch locale, err := locale(ctx); {
    case err != nil:
        return "", err
    case locale == "EN/US":
        return "hello", nil
    }
    return "", fmt.Errorf("unsupported locale")
}

func genFarewell(ctx context.Context) (string, error) {
    switch locale, err := locale(ctx); {
    case err != nil:
        return "", err
    case locale == "EN/US":
        return "goodbye", nil
    }
    return "", fmt.Errorf("unsupported locale")
}

func locale(ctx context.Context) (string, error) {
    select {
    case <-ctx.Done():
        return "", ctx.Err()
    case <-time.After(1 * time.Minute):
        fmt.Println("Expired")
    }
    return "EN/US", nil
}
{{< /highlight >}}


<h3>Error Propagation</h3>

{{< highlight go >}}

.........

..........

.......

{{< /highlight >}}


<h1>Concurrency Project</h1>

This section will discuss in detail about the [eapache project](https://github.com/eapache). This project is a very useful project to learn more in-depth about concurrency. There are several different implementations and patterns it implemented that are useful to use in an application. The article will talk in detail about the _**go-resilliency**_

<h3>Blackhole</h3>

The function of the class is to receive data and discard it and calculated the total number of data it received. This is useful in situation where we want to check if our app concurrency app is receiving the same amount of data as expected. 

{{< highlight go >}}

package channels

// BlackHole implements the InChannel interface and provides an analogue for the "Discard" variable in
// the ioutil package - it never blocks, and simply discards every value it reads. The number of items
// discarded in this way is counted and returned from Len.
type BlackHole struct {
	input  chan interface{}
	length chan int
	count  int
}

func NewBlackHole() *BlackHole {
	ch := &BlackHole{
		input:  make(chan interface{}),
		length: make(chan int),
	}
	go ch.discard()
	return ch
}

func (ch *BlackHole) In() chan<- interface{} {
	return ch.input
}

func (ch *BlackHole) Len() int {
	val, open := <-ch.length
	if open {
		return val
	} else {
		return ch.count
	}
}

func (ch *BlackHole) Cap() BufferCap {
	return Infinity
}

func (ch *BlackHole) Close() {
	close(ch.input)
}

func (ch *BlackHole) discard() {
	for {
		select {
		case _, open := <-ch.input:
			if !open {
				close(ch.length)
				return
			}
			ch.count++
		case ch.length <- ch.count:
		}
	}
}
{{< /highlight >}}

Following is a code sample on how to use Blackhole

{{< highlight go >}}

func TestBlackHole(t *testing.T) {
	discard := NewBlackHole()

	for i := 0; i < 1000; i++ {
		discard.In() <- i
	}

	discard.Close()

	if discard.Len() != 1000 {
		t.Error("blackhole expected 1000 was", discard.Len())
	}
}
{{< /highlight >}}


<h3>Deadline</h3>

There are instances where we want our application to do a background process but there is a time limit imposed on it. Following is a sample snippet on how to use this functionality 


{{< highlight go >}}

func ExampleDeadline() {
	dl := New(1 * time.Second)

	err := dl.Run(func(stopper <-chan struct{}) error {
		// do something possibly slow
		// check stopper function and give up if timed out
		return nil
	})

	switch err {
	case ErrTimedOut:
		// execution took too long, oops
	default:
		// some other error
	}
}
{{< /highlight >}}

<h3>Circuit Breaker</h3>

<h3>Batcher</h3>

