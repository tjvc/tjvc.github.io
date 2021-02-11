---
layout: post
title:  Growing buffers in Go
date:   2021-02-10
---


I recently reached for Go's `ioutil.ReadAll` utility function to read some data from a HTTP request body. I'd read that this function should be used with care because it can lead to large values being read into memory, potentially causing crashes. I was curious to find out how that could happen.

[Looking at the source for `ioutil.Readall`](https://golang.org/src/io/ioutil/ioutil.go), we can see that it reads data into a buffer, implemented as a byte slice, with `bytes.Buffer.ReadFrom`. [Reading the source for `ReadFrom`](https://golang.org/src/bytes/buffer.go), we see that it will read data up to the buffer's capacity and then attempt to grow the buffer, calling the private `grow` function.

From the docs for `grow`: `if the buffer can't grow it will panic with ErrTooLarge`. Within `grow` itself, the condition that triggers this panic is `c > maxInt-c-n`, where `c` is the capacity of the current buffer and `n` is the minimum slice size passed to a `Read` call by `Buffer.ReadFrom`. This is because, assuming re-slicing or dropping previously read bytes are not possible, Go will grow the buffer to twice its current capacity, plus `n`.

`maxInt` is defined as `int(^uint(0) >> 1)`. Unary `^` is the bitwise `NOT` operator: it inverts every bit of the unsigned integer `0`, yielding the maximum possible integer. `>>`, the right shift operator, here shifts bits right by one position, equivalent to dividing by two. So this expression returns the maximum possible signed integer (the leftmost bit being the sign bit).

On a 64-bit architecture then, `grow` will refuse to create a slice longer than `9223372036854775807`. Practically speaking, we're always going to exhaust the available memory before reaching that length, so where else can `ErrTooLarge` be thrown? The answer is in `makeSlice`, called from `grow`, which recovers from a panic thrown by `make` to throw `ErrTooLarge`. `make` itself is [mapped to a type-specific implementation when compiling](https://stackoverflow.com/a/18513087), in this case `makeslice`, which panics when asked to create a slice larger than the maximum memory allocation.
