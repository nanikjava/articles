---
date: "2021-03-09"
title: "go-workers"
author : "Nanik Tolaram (nanikjava@gmail.com)" 

---

[go-workers](https://github.com/catmullet/go-workers) is an open source project that helped developers in making it easier to build concurrency into their app. The project is suitable for small to medium project. The project removes developers from the headache of worrying about channels, etc.

Developers just have to define the workers that it need to do work asynchronously and different workers can work together passing data in and out between them. We defined our worker as follows:

{{< highlight go >}}
type Worker struct {
}
{{</highlight>}}

and we need to have the implementation of `Work` like follows:

{{< highlight go >}}
func (wo *Worker) Work(w *worker.Worker, in interface{}) error {
	..
	..
}
{{</highlight>}}

Once we have implementation ready we can execute it using the library as follows:

{{< highlight go >}}
workerOne := worker.NewWorker(ctx,  &SingleWorker{}, 1).Work()
{{</highlight>}}

The worker object now ready to receive messages by using the following command:

{{< highlight go >}}
workerOne.send(<can_send_anything>)
{{</highlight>}}

The following example shows how 2 workers processing the same data independently.

{{< highlight go >}}
package main

import (
	"context"
	"fmt"
	worker "github.com/catmullet/go-workers"
	"os"
	"path/filepath"
	"time"
)

func main() {
	ctx := context.Background()
	t := time.Now()

	f, err := os.Create(filepath.Join(os.TempDir(), "output.txt"))

	if err != nil {
		os.Exit(1)
	}

	workerOne := worker.NewWorker(ctx, NewWorker(), 1).SetWriterOut(f).Work()
	workerTwo := worker.NewWorker(ctx, NewTestWorkerObject(), 10)
	workerTwo.InFrom(workerOne).Work()

	workerOne.SetWriterOut(f).Work()

	for i := 0; i <= 2000; i++ {
		workerOne.Send(i)
	}
	defer f.Close()
	workerOne.Close()
	totalTime := time.Since(t).Milliseconds()
	fmt.Printf("total time %dms\n", totalTime)
}

type Worker struct {
}

type NewTestWorker struct {
}

func NewWorker() *Worker {
	return &Worker{}
}

func NewTestWorkerObject() *NewTestWorker {
	return &NewTestWorker{}
}
func (wo *NewTestWorker) Work(w *worker.Worker, in interface{}) error {
	i := in.(int)
	w.Out(i)
	return nil
}

func (wo *Worker) Work(w *worker.Worker, in interface{}) error {
	i := in.(int)
	w.Println(i)
	return nil
}

{{</highlight>}}


The following is an example on using the library to GET data from [cat-fact](https://cat-fact.herokuapp.com/facts) every 5 seconds.

{{< highlight go >}}
package main

import (
	"context"
	worker "github.com/catmullet/go-workers"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {

	ctx := context.Background()
	workerOne := worker.NewWorker(ctx, NewSingleWorker(), 1).
		SetWriterOut(os.Stdout).
		Work()

	c := make(chan os.Signal)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)
	go func() {
		<-c
		workerOne.Close()
		os.Exit(0)
	}()

	for {
		workerOne.Send(nil)
		time.Sleep(5 * time.Second)
	}
}

type SingleWorker struct {
}

func NewSingleWorker() *SingleWorker {
	return &SingleWorker{}
}

func (wo *SingleWorker) Work(w *worker.Worker, in interface{}) error {
	download()
	return nil
}

func download() {
	resp, err := http.Get("https://cat-fact.herokuapp.com/facts")
	if err != nil {
		log.Fatalln(err)
	}
	//We Read the response body on the line below.
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatalln(err)
	}
	//Convert the body to type string
	sb := string(body)
	log.Printf(sb)
}

{{</highlight>}}
