---
layout: post
title: new, make and zero values in Go
date: 2021-08-06
---

`new(T)` returns a pointer to a newly allocated zero value of type `T`.

`make` returns an _initialised_ slice, map or channel. For example, a slice which points to an underlying array (an uninitialised slice does not).

In Go, the zero value is a default value assigned to an allocated but uninitialised variable. Each type has its own zero value. For slices and maps, the zero value is `nil`.

Once assigned to a variable, `nil` is typed and behaves accordingly. You can `append` to a `nil` slice and look up keys in a `nil` map, but not vice versa. You cannot compare `nil`s of different types.

- <https://golang.org/doc/effective_go#allocation_new>
- <https://golang.org/ref/spec#The_zero_value>
- <https://blog.golang.org/slices-intro>
