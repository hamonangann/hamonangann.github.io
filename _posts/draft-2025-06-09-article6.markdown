---
layout: post
title: "Making Use of Go Channels #1: Indefinite Process, Graceful Shutdown"
categories: tips
featured_image: /img/article3-golang-logo.png
---

Concurrency in Go is quite tricky. We have `channel`, a simple type that can be used for sending and receiving data concurrently. We also have `goroutines`, a lightweight thread managed by Go runtime to make concurrent operations possible.

Fortunately, there are many implementations `sync.WaitGroup` such as `sync.Mutex` to make things related to concurrency easier. Nowadays, developers don't need to set channels and goroutines manually really often. However, there are several fundamental things that are common and can be done with simple `channel` implementation. In this article, we will discuss these cases.

## Make an indefinite process

The most basic use case for the unbuffered channel is blocking a process until data is supplied to the channel.

```go
package main

import "fmt"

func main() {
	ch := make(chan struct{})
	go func() {
		for i := range 10 {
			fmt.Println(i)
		}
		ch <- struct{}{}
	}()
	<-ch

	fmt.Println("Counting done")
}
```

In the code above, `ch` makes sure that the goroutine function will not exit before it finishes the loop. This is because the `main` goroutine waits for channel send/receive, which happens after the loop.

If the channel never receives any data, the program will run indefinitely until the user closes the program.


```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan struct{}{}) 

	go func() {
		for {
			fmt.Println("ping")
			time.Sleep(time.Second)
		}
	}()
	<- ch
}
```

This program prints `ping` every second indefinitely and will only exit with interrupts such as `Ctrl+C`. To be exact, the program runs until `SIGINT` or `SIGTERM` is sent by the operating system.

## Graceful shutdown

Furthermore, our program can respond to interrupts instead of directly exiting. Usually, it is related to cleanup or `defer` operations, making graceful shutdown possible.

To do this, we need a buffered channel of 1 and `signal.Notify`.

```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	go func() {
		for {
			fmt.Println("ping")
			time.Sleep(time.Second)
		}
	}()
	
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit

	fmt.Println("Prints before exiting")
}
```

Instead of exiting, the program listens to OS interrupts such as `SIGINT` and `SIGTERM`. That means, for example, if a user invokes `Ctrl+C` to close the program, the main goroutine will continue to the next operations after `<- quit`.

We can apply this pattern to make sure shutdown operations don't break a system or leak data. For example, we can implement a graceful shutdown to a web server as follows.

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	srv := http.Server{
		Addr: ":8080",
	}

	// Start a server at port 8080
	// This goroutine will run indefinitely
	// If any server error happens, the program will exit immediately
	go func() {
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("server error: %v", err)
		}
	}()

	quit := make(chan os.Signal, 1)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit

	// Graceful shutdown starts
	// At this point, further interrupts will not be handled and are considered as forced shutdown
	log.Println("shutdown server...")

	// If next shutdown operation doesn't finished after 3 second, just exit the program
	// This prevents indefinite shutdown time
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()

	if err := srv.Shutdown(ctx); err != nil {
		log.Fatalf("server shutdown error: %v", err)
	}

	<-ctx.Done()

	log.Println("Server exited gracefully")
}
```

Continue to part 2...

## Reference

[Channel Use Cases](https://go101.org/article/channel-use-cases.html)

[Signals](https://gobyexample.com/signals)
