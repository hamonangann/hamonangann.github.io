---
layout: post
title: "Making Use of Go Channels #2: Responding to Asynchronous Operations"
categories: tips
featured_image: /img/article3-golang-logo.png
---

In [part 1](https://hamonangann.github.io/tips/2025/06/09/article6.html) we already discuss how Go channels is used to make indefinite process. Now, we will discuss another common use case for channels.

## Responding to Asynchronous Operation

We know that goroutines are meant to be run asynchronously, but we may want a goroutine to respond to another goroutine operation's result. For example, in Javascript we have `Promise` object, that can either be resolved or rejected, and responded accordingly. In Go, this can be more flexible using `select`.

For instance, given a code making API call.

```

```



## Reference

[Channel Use Cases](https://go101.org/article/channel-use-cases.html)

[Signals](https://gobyexample.com/signals)
