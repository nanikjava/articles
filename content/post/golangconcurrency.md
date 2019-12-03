---
date: "2019-12-02"
title: "Concurrency Design Pattern in Go"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---

---------------
Design Patterns
---------------

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
    
<h3>Tee-Channel</h3>

Takes a single input channel and an arbitrary number of output channels and duplicates each input into every output. When the input channel is closed, all outputs channels are closed.



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

<h3>Queuing</h3>

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



<h1>References</h1>

* Golang
	* http://peter.bourgon.org/go-in-production/#testing-and-validation --> best practises
    * Lesson learned from contributing minikube
        * https://github.com/kubernetes/minikube/pull/5718#discussion_r339308904
            - "Generally isn't useful to repeat the type of an object in the object name."  See also https://github.com/golang/go/wiki/CodeReviewComments#variable-names
