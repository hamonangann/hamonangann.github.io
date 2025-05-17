---
layout: post
title: "Exploring Go: Pointers"
categories: tips
---

For programmers not familiar with Golang (or C/C++ kind of language), pointers may be a new thing that is quite tricky. Therefore, let's explore Go pointers!

## Pointer and Memory

In every Go application, variable values are stored in memory. Because a memory has so many place to store data, our program keeps track of every data address, well, in memory.

The great thing about Go is, we can actually know the address using pointer data type. This is because, pointer's value is an address to another data value.

To show the power of the Go pointers, let's take a look at these examples!

## 1. Memory address of a variable

Try to run this program! [Go Playground](https://go.dev/play/p/zbGCvsI2GMb)

```go
package main

import "fmt"

func main() {
    var x int
    x = 5
	fmt.Println(x, &x)
}
```

Here we have a variable `x` as an integer. The output of this program is as follows.

```shell
5 0xc000012070
```

Besides the value `x` which is 5, in Go, we can actually know where `x` is stored in the memory. To do that, we can use the **reference** `&` operator before the variable name.

```
*Note*: the memory address shown in your experiment may be different, but its type is always a memory address.
```

## 2. Store memory address in a pointer

Now, try to run this! [Go Playground](https://go.dev/play/p/qHSsat8iqGu)

```go
package main

import "fmt"

func main() {
    var x int
    var y *int
    x = 5
    y = &x
	fmt.Println(x, y)
}
```

```shell
5 0xc000012070
```

We are getting similar result with (1). The difference is, we now have new variable `y` storing the address of `x`. This kind of type is called **pointer**.

To initialize (create new) pointers, we use `*` operator before the variable name.

## 3. Get the pointer's value

Remember that a pointer's value is a reference to a memory address? In fact, we can also use `*` operator to get a value of the variable referenced by a pointer. Now try to run this. [Go Playground](https://go.dev/play/p/KUFhi_Ej1Rg)

```go
package main

import "fmt"

func main() {
	var x int
	var y *int
	x = 5
	y = &x
	fmt.Println(y, *y)
}
```

We are getting the same result as (2)! This is because `*y` is actually the **value** of `x`, while `y` is the **reference** to `x`.

## 4. Pass by value

Function in Go is always pass-by-value. That means that every variable inside a function is actually a **new** variable that have the value assigned by the caller. Because of that, the memory address of the variable inside a function is **different** from outside the function.

```go
package main

import "fmt"

func passByValue(x int) {
	fmt.Printf("Inside function %d %p \n", x, &x)
}

func main() {
	x := 5
	passByValue(x)
	fmt.Printf("Outside function %d %p \n", x, &x)
}
```

The output of the program above is as follows. [Go Playground](https://go.dev/play/p/AvTJcDrUQ5P)

```shell
Inside function 5 0xc000012078 
Outside function 5 0xc000012070
```

The value of `x` inside and outside function is obviously the same, but it is stored in a different variable with a different address. 

One of the impacts of this is `x` modification **inside** function does not affect the `x` **outside** function.

```go
package main

import "fmt"

func passByValue(x int) {
	x = 10
	fmt.Printf("Inside function %d %p \n", x, &x)
}

func main() {
	x := 5
	passByValue(x)
	fmt.Printf("Outside function %d %p \n", x, &x)
}
```

[Go Playground](https://go.dev/play/p/fvyDgcxkLyB). We cannot change `x` in `passByValue` function above, because outside the function, `x` value is still 5.

Under the hood, Go creates a new **local** variable `x` (only applies inside the function) which its value is **copied** from `x` at the caller. 

## 5. Pass a pointer to a function

If so, can we actually modify values passed to a function? In Go, we can pass a pointer (with the dereference operator) and modify the value using that pointer.

```go
package main

import "fmt"

func passByReference(x *int) {
	*x = 10
	fmt.Printf("Inside function %d %p \n", *x, x)
}

func main() {
	x := 5
	passByReference(&x)
	fmt.Printf("Outside function %d %p \n", x, &x)
}
```

[Go Playground](https://go.dev/play/p/EkS-st9PrNx). In `passByReference` function, we change the values pointed by `x` from 5 to 10 with the dereference operator. Because `x` inside function is a pointer to `x` outside function, we can access and modify the value there. Therefore, when we print, we get the same result.

To pass a pointer to a function, we use the reference operator `&x` when we call `passByReference`

```shell
Inside function 10 0xc000012070 
Outside function 10 0xc000012070 
```

Now the value outside function is changed.

Please note that `x` pointer within a function with `&x` pointer is **different**. When a function is called, Go creates a local pointer `x` with value copied from `&x` (which points to our initial `x`). Therefore, it is still pass-by-value, actually. We just make the reference to `x` as a pointer value.

## 6. Setter dan getter

The most common pointer usage in Go is setter and getter for a struct. [Go Playground](https://go.dev/play/p/MBAVHyEPecy)

```go
package main

import (
	"fmt"
	"strings"
)

type Person struct {
	name string
}

func NewPerson(name string) Person {
	return Person{strings.Trim(name, " ")}
}

func (p Person) Name() string {
	return p.name
}

func (p *Person) SetName(name string) {
	p.name = strings.Trim(name, " ")
}

func main() {
	p := NewPerson(" Andy")
	fmt.Println(p.Name())
	p.SetName("Handy")
	fmt.Println(p.Name())
}
```

The name is first initialized as `Andy` and then modified to `Handy`.

There are details here. To change `p.name`, we don't have to use dereference `(*p).name = name` because the Go compiler can detect our intention here.

Of course, when we want to modify (setter) we must pass a pointer to make the value actually change (see (5)). On the other hand, we can choose to pass either pointer or value to getter methods.  

## 7. List/Map of Struct

Another example of pointer usage is struct modification. Let's take a look at this example. [Go Playground](https://go.dev/play/p/u7h3hYXOazk)

```go
package main

import "fmt"

type Person struct {
	FirstName string
	LastName  string
}

func main() {
	p0 := Person{"Andy", "Timmons"}
	p1 := Person{"Ardy", "Putra"}

	persons := []Person{p0, p1}

	persons[0].LastName = "Caroll"

	fmt.Println(p0.LastName)
}
```

Seems like we don't get what we want. When printed, `LastName` is still `Timmons`, not `Carrol`. It is because `persons[0]` is 
different from `p0`.

When we create `persons` list, the value of `p0` is copied to another memory address, and the copy is assigned to `persons[0]`. Because they are actually different, changes of `persons[0]` does not affect `p0` and vice versa.

We can tackle this issue using a pointer. [Go Playground](https://go.dev/play/p/sK8bkUFdYnk)

```go
package main

import "fmt"

type Person struct {
	FirstName string
	LastName  string
}

func main() {
	p0 := Person{"Andy", "Timmons"}
	p1 := Person{"Ardy", "Putra"}

	persons := []*Person{&p0, &p1}

	persons[0].LastName = "Caroll"

	fmt.Println(p0.LastName)
}
```

After we modified the code above, the LastName successfully changed to `Caroll`!

It is because `p0` and `persons[0]` are now pointers (which point to the same memory address). Now, change on `persons[0]` will affect the value pointed by `p0`

We can achieve the same result with Go map values.

## 8. Nil pointer

A pointer can also point to no data or variables. In this case, that pointer refers to a zero value called **nil**. [Go Playground](https://go.dev/play/p/Ya4F7UCMFgi)

```go
package main

import "fmt"

func main() {
	var p *int
	p = nil
	fmt.Println(p)
}
```

The output of the code above is "<nil>". How if we use `*` to expose the value? [Go Playground](https://go.dev/play/p/p-d5D1cYjDj)

```go
package main

import "fmt"

func main() {
	var p *int
	p = nil
	fmt.Println(*p)
}
```

Turns out we get a runtime error (panic). This is because <nil> pointer is not pointing to any data in memory, so printing the value will result in error.

```shell
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x48eb36]

goroutine 1 [running]:
main.main()
	/tmp/sandbox1296800962/prog.go:8 +0x16

Program exited.
```

## 9. Error object

Another common pointer application is the error object. In Go, we don't have stack trace or even try-catch functionality to detect errors. Therefore, the error object in Go is only an ordinary struct. [Go Playground](https://go.dev/play/p/xmszAs72wUS)

```go
package main

import (
	"errors"
	"fmt"
)

type CharError struct {
	Message string
}

func (e *CharError) Error() string {
	return e.Message
}

func firstChar(s string) (string, error) {
	if len(s) == 0 {
		return "", &CharError{"Length of string s is zero"}
	}
	return s[:1], nil

}

func main() {
	first, err := firstChar("")
	if err != nil {
		var charError *CharError

		if errors.As(err, &charError) {
			fmt.Println("Empty input certainly don't have first character")
			return
		}

		fmt.Println("Unexpected error: ", err.Error())
		return
	}
	fmt.Println("The first character of the input is", first)
}
```

We can see the error is passed from `firstChar` as a pointer. Next, `if err != nil` is used to catch errors. If the pointer actually points to an error, the program will handle the error inside `if` block, and vice versa.

If we run the program above, we will get `Empty input certainly doesn't have first character` as an output.

## 10. Pointer and Interface

A pointer (points to struct, for example), can also be used as an implementation of an interface. For example, we can code like this. [Go Playground](https://go.dev/play/p/4sxS0fxintD)

```go
package main

import "fmt"

type Vehicle interface {
	Start()
	Stop()
}

type Car struct {
	Power string
}

func (c *Car) Start() {
	c.Power = "On"
}

func (c *Car) Stop() {
	c.Power = "Off"
}

func main() {
	var jaguar Vehicle = &Car{}
	jaguar.Start()
	fmt.Println(jaguar.(*Car).Power)
}
```

Here we create a new variable `jaguar` as a point to a `Car`. Here, `*Car` is the one who implements the `Vehicle` interface, not the `Car` itself.

One of the common mistakes in Go is using a pointer to interface to initialize certain interface implementations. For example, running the code below will result in an error. [Go Playground](https://go.dev/play/p/5PSDVBMbfDK)

```go
package main

type Vehicle interface {
	Start()
	Stop()
}

type Car struct {
	Power string
}

func (c *Car) Start() {
	c.Power = "On"
}

func (c *Car) Stop() {
	c.Power = "Off"
}

func main() {
	var jaguar *Vehicle = &Car{}
}
```

```shell
./prog.go:21:24: cannot use &Car{} (value of type *Car) as *Vehicle value in variable declaration: *Car does not implement *Vehicle (type *Vehicle is pointer to interface, not interface)
```

Please note that a pointer to an interface is **not** the interface (well, it is, a pointer). It is a concrete type storing the memory address of an interface, and it is uncommon.

To fix the error above, just declare `jaguar` as `Vehicle`, not `*Vehicle`.


## Reference

[Pointers Explained](https://yourbasic.org/golang/pointers-explained/)

[Go Pointer](https://wahyu-ehs.medium.com/go-pointer-47f1ed099682)

[Menggunakan Pointer di Golang dengan Mudah](https://husniadil.com/menggunakan-pointer-di-golang-dengan-mudah/)