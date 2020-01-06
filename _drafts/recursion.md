---
layout: post
title:  "Recursion"
date:   2020-01-06
---

To solve a problem recursively, we repeatedly break it down into and solve smaller instances of the same problem, until we arrive at some base case that is small enough to be solved directly. A recursive function calls itself until the base case is reached, with each call returning a solution to a subproblem. The first function call (at the bottom of the stack) returns a complete solution to the problem.

Take the factorial function, where `n!` is equal to `n * (n - 1) * (n - 2) ... * 2 * 1`, as an example. Where `n = 0` we know that `n! = 0`; this is our base case. For all greater values of `n`, we can say that `n! = n * (n - 1)!`. To find the factorial of `n`, we solve the subproblem `(n - 1)!` repeatedly until we arrive at our base case.

Here's an example Ruby implementation of a recursive factorial function. The first line tests for our base case, the second is the recursive function call.

```ruby
def factorial(n)
  return 1 if n.zero?

  n * factorial(n - 1)
end
```
