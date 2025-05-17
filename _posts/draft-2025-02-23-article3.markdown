---
layout: post
title: "Exploring Go: Channels"
categories: tips
featured_image: /img/article3-golang-logo.png
---

As a modern programming language, Go supports concurrent data transfer using a mechanism called **channels**. This may be new for anyone unfamiliar with Golang, so let's dive into it!

## Goroutines and how to spawn them

We can imagine a goroutine as a sequence of commands. It can be a calculation, or a command to spawn another sequence of commands. A Go runtime independently manages goroutines so they run alternately. That means that a goroutine doesn't have to wait for another goroutine to finish its sequence of commands if it is not intended. However, we cannot tell which goroutine execution finishes first.

Let's take a look at this code. [Go Playground](https://go.dev/play/p/cQDNcRaIUuo)

```go
package main

import "fmt"

func A() {
	fmt.Println("A")
}

func main() {
	go A()
	fmt.Println("Main")
}
```

There are three possible outputs: `Main` followed by `A`, `A` followed by `Main`, and `Main` only.
Here is what happened: `main()` itself is a goroutine--we can call it "main goroutine". It is a sequence of two commands: spawn new goroutine `A()` and print `Main`. New goroutine `A()` prints `A`, but we cannot tell which goroutine finishes first. If `main()` finishes first, the runtime finishes and all goroutines are killed. If `A()` finishes first, `main()` may continue executing without exiting the runtime.

To spawn a new goroutine, we use `go` followed by a function call.

## 1. Sync Goroutines with Channel

To make goroutines useful, we may want to make them communicate with each other. To do that, we can use a channel.

This sample program shows a simple `channel` implementation. [Go Playground](https://go.dev/play/p/T1pnIyARWd7).

```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan int) {
	x := <-ch
	fmt.Println(x)
}

func B(ch chan int) {
	y := 5
	ch <- y
}

func main() {
	ch := make(chan int)
	go A(ch)
	go B(ch)
	time.Sleep(time.Second)
}
```

Here we have `A()` receives `5` from `B()` through a channel `ch`. To make a new channel, we use `make(chan <data-type>)` with any data type or struct. This program prints `5` as long as `main()` is not finished before `A()` and `B()` (we use `time.Sleep` as a trick).

![Unbuffered channel](/img/article3-ch.png)

Note that A's execution is blocked until `ch` finds a sender (which is from goroutine B), and so is B blocked until `ch` finds a receiver. That means print operation will always be done after `x` receives value from `ch`.

Try to swap `go B(ch)` so it comes before `go A(ch)` and we will have the same result. Again, we cannot tell which goroutine (A or B) finishes its execution first.

## 2. Buffered channel

This is a common mistake when spawning a Goroutine. Take a look at this example. [Go Playground](https://go.dev/play/p/csGp0IDeBat)

```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan int) {
	x := <-ch
	fmt.Println(x)
}

func main() {
	ch := make(chan int)
	ch <- 3
	go A(ch)
	time.Sleep(time.Second)
}
```

Here we send `3` through a channel, but we never call the receiver. It is because the statement `ch <- 3` actually blocks the `main()` operation, so the program never reaches `go A()`. If we run this program, we will get the following error.

```shell
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
	/tmp/sandbox1789620925/prog.go:15 +0x2d

Program exited.
```

Here Go runtime detects that there are no goroutines ready to receive value from `ch`, so the sender goroutine deadlocks.

There are two ways to solve this problem. One is to simply swap `go A()` execution before `ch <- 3` so `A()` will receive the value through the channel.

Two, we can actually **store** the value in a channel. To do that, we need a buffered channel.

This example shows how it works. [Go Playground](https://go.dev/play/p/fnWROmxJVa9)


```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan int) {
	x := <-ch
	fmt.Println(x)
}

func main() {
	ch := make(chan int, 1)
	ch <- 3
	go A(ch)
	time.Sleep(time.Second)
}
```

Now we have `3` stored temporarily in `ch` so it won't block the next operation (spawning new goroutine A). `A` then receives from `ch`, frees up `ch` memory spaces, and print the value.

Another example of a buffered channel: [Go Playground](https://go.dev/play/p/FORpYDvIn1W)

```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan string) {
	x := <-ch
	fmt.Println(x)
	x = <-ch
	fmt.Println(x)
}

func B(ch chan string) {
	ch <- "Alice"
	ch <- "Bob"
}

func main() {
	ch := make(chan string, 2)
	go A(ch)
	go B(ch)
	time.Sleep(time.Second)
}
```

To define a buffered channel, we use `make` with buffer size as the second parameter. In this example, we create a string channel `ch` with size 2. Then, `ch` stores both `Alice` and `Bob` string before it is released by receiver `A()`. The output of this program is `Alice` followed by `Bob`.

As an exercise, try to modify this code to make avoid possible deadlocks.

```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan string) {
	ch <- "Foo"
	ch <- "Bar"
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}

func main() {
	ch := make(chan string)
	go A(ch)
	go A(ch)
	time.Sleep(time.Second)
}
```

[Solution](https://go.dev/play/p/UciGWhSTJP3)

A buffered channel blocks send operations when the buffer is full and blocks receive operations when the buffer is empty. To avoid deadlocks, we want to make a channel with enough capacity until any receive operation `fmt.Println` is reached.

![Buffered Channel](/img/article3-buffered-ch.png)

## 3. Channel and Select

Try to run this program!

```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan string) {
	time.Sleep(5 * time.Second)
	ch <- "A Done"
}

func B(ch chan string) {
	ch <- "B Done"
}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	go A(ch1)
	go B(ch2)
	fmt.Println(<-ch1)
	fmt.Println(<-ch2)
}
```

The output of this program is always `Done A` followed by `Done B`. Here we spawn two goroutines. However, `fmt.Println(<-ch1)` always blocks the next operations, even if it consumes a significant amount of time.

We can improve this. Because we cannot exactly know which goroutine finishes first, we can use `select` statement to process whatever channel receives a value. [Go Playground](https://go.dev/play/p/zvPc3JcxP7O)

```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan string) {
	time.Sleep(5 * time.Second)
	ch <- "A Done"
}

func B(ch chan string) {
	ch <- "B Done"
}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	go A(ch1)
	go B(ch2)

	for range 2 {
		select {
		case x := <-ch1:
			fmt.Println(x)
		case y := <-ch2:
			fmt.Println(y)
		}
	}
}
```

The output may be `B Done` followed by `A Done` after a while. Here we can process any channel that finishes first without waiting for each other.

## 4. Channel with Timeout

In the previous examples, we have used `time.Sleep` to delay a goroutine's execution. We can set a timeout to avoid waiting for data transfer too long.

For example, we can modify the above code to terminate the operation after 3 seconds. Note that we can utilize an infinite `for` loop and don't need to use `range` anymore. [Go Playground](https://go.dev/play/p/t_9IUuaYWDd)

```go
package main

import (
	"fmt"
	"time"
)

func A(ch chan string) {
	time.Sleep(5 * time.Second)
	ch <- "A Done"
}

func B(ch chan string) {
	ch <- "B Done"
}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	go A(ch1)
	go B(ch2)

	for {
		select {
		case x := <-ch1:
			fmt.Println(x)
		case y := <-ch2:
			fmt.Println(y)
		case <-time.After(3 * time.Second):
			fmt.Println("Timeout")
			return
		}
	}
}
```

Now we may have `B Done` followed by `Timeout`. We didn't see operations taking too much time.

## 5. Channel with Close

We can explicitly close a channel if it is not needed anymore. Try to run this! 

```go
package main

import "fmt"

func A(ch chan int) {
	for i := range 10 {
		ch <- i
	}
}

func main() {
	ch := make(chan int)
	go A(ch)
	for i := range ch {
		fmt.Println(i)
	}
}
```

Here we iterate over a channel using `for...range`. However, this code results in a deadlock, because `main()` routine keeps waiting for a value from the channel, whose sender stops after 10 times.

In this case, we need to explicitly close the channel so it won't send data anymore. [Go Playground](https://go.dev/play/p/pCcObNb4tMg)

```go
package main

import "fmt"

func A(ch chan int) {
	for i := range 10 {
		ch <- i
	}
	close(ch)
}

func main() {
	ch := make(chan int)
	go A(ch)
	for i := range ch {
		fmt.Println(i)
	}
}

```

## Additional: Explanation of the solution

In the problem above, we need to declare a buffered channel with a size of **at least** 3.

- If we use an unbuffered channel or a buffered channel with size 1, it will certainly deadlock, because we cannot get into the second statement `ch <- "Bar"` of any goroutines a with full buffer.

- If we use a buffered channed with size 2, it **may** deadlock when the first two strings stored are `Foo` from both goroutines.

## Reference

[Dave Cheney - Performance Without the Event Loop](https://dave.cheney.net/2015/08/08/performance-without-the-event-loop)

[Channel Select](https://dasarpemrogramangolang.novalagung.com/A-channel-select.html)

[Channel Timeout](https://dasarpemrogramangolang.novalagung.com/A-channel-timeout.html)

[Channel Range & Close](https://dasarpemrogramangolang.novalagung.com/A-channel-range-close.html)
