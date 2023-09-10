---
published: false
title: Understanding Closures in Python
description: Closures are a powerful concept in programming that exist in both Python and many other programming languages. They allow functions to retain access to their enclosing scope's variables even after the outer function has completed execution.
tags:
  - python
  - javascript
  - tutorial
  - programming
cover_image: 'https://i.imgur.com/r30OEKR.png'
canonical_url: null
---

Closures are a powerful concept in programming that exist in both Python and many other programming languages. They allow functions to retain access to their enclosing scope's variables even after the outer function has completed execution.

In Python, a closure is a nested function that captures and remembers the values of variables in the enclosing (containing) function's local scope, even if the enclosing function has finished executing. This means that a closure "closes over" the variables it needs, allowing them to persist beyond the lifetime of the enclosing function.

To create a closure, you typically define a nested function inside another function and return that nested function.

## How Closures Work

Let's start with a simple example to illustrate how closures work in Python:

```python
def outer_function(x):
    def inner_function(y):
        return x + y
    return inner_function

closure = outer_function(10)
result = closure(5)
print(result)  # Output: 15
```

In this example, `inner_function` is defined inside `outer_function`. When `outer_function` is called with an argument of 10, it returns `inner_function`, which is then assigned to the closure variable. Later, when `closure` is called with an argument of 5, it still has access to the `x` variable from its enclosing scope, resulting in the output 15.

## Why Use Closures?

Closures can be incredibly useful in various scenarios, such as:

**Function Factories**: Closures can be used to generate specialized functions based on some initial configuration or parameters. This is particularly handy when you need to create multiple similar functions with different settings.

**Callbacks**: Closures are commonly used in event-driven programming to create callback functions that carry additional context or state information.

**Data Encapsulation**: You can use closures to encapsulate data within a function, creating a private scope for that data. This helps in maintaining data integrity and preventing unintended modifications.

**Decorator Pattern**: Modifying the behaviour of functions without altering their code.

**Memoization**: Closures can be used to cache the results of expensive function calls, improving performance by avoiding unnecessary re-computation.

Now, let's dive into some Python code examples to understand closures better.

### Function Factory

```python
def exponentiate(power):
    def inner(base):
        return base ** power
    return inner

square = exponentiate(2)
cube = exponentiate(3)

print(square(4))  # Output: 16
print(cube(4))    # Output: 64
```
Here, `exponentiate` is a function factory that generates functions for calculating squares and cubes.

### Callbacks

```python
def event_handler(event_name):
    def callback(*args, **kwargs):
        print(f"Event '{event_name}' triggered with args: {args}, kwargs: {kwargs}")
    return callback

button_click = event_handler("button_click")
button_click("left_click", x=100, y=200)
# Event 'button_click' triggered with args: ('left_click',), kwargs: {'x': 100, 'y': 200}
```

In this example, `event_handler` creates a callback function tailored for a specific event.

### Data Encapsulation

```python
def counter():
    count = 0

    def increment():
        nonlocal count  # Use nonlocal to modify the count variable in the enclosing scope
        count += 1
        return count

    def decrement():
        nonlocal count
        count -= 1
        return count

    return increment, decrement

inc, dec = counter()  # Get closures for increment and decrement
print(inc())  # Output: 1
print(inc())  # Output: 2
print(dec())  # Output: 1
```

In this example, counter returns two closures: `increment` and `decrement`. These closures share access to the count variable and allow us to modify it without exposing it to the global scope.

### Decorators

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()

# Output:
# Something is happening before the function is called.
# Hello!
# Something is happening after the function is called.
```

This code defines a decorator `my_decorator`, which wraps the `say_hello` function with additional behaviour.

### Memoization

```python
import time


def memoize(func):
    cache = {}

    def wrapper(n):
        if n not in cache:
            cache[n] = func(n)
        return cache[n]

    return wrapper


@memoize
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n - 1)


start_time = time.time()
factorial(300)
print((time.time() - start_time)*1000, "ms in 1st time")

start_time = time.time()
factorial(300)
print((time.time() - start_time)*1000, "ms in 2nd time")

# Output:
# 1.6314983367919922 ms in 1st time
# 0.004291534423828125 ms in 2nd time
```

In this example, we have created a `memoize` closure that takes a function `func` as its input. The `cache` dictionary within the closure stores previously computed results to avoid redundant calculations.

The `wrapper` function is the actual closure, which takes an argument `n`. It checks if the result for `n` is already in the `cache`. If it is, it returns the cached value; otherwise, it computes the result using the original `func` and caches it before returning.

The `@memoize` decorator is applied to the `factorial` function, which enables memoization for the `factorial` function. Now, when you call `factorial(n)` for any value of `n`, the function will check if the result is already cached before performing the actual factorial computation. This dramatically reduces redundant calculations and improves performance for large values of `n`.

Using closures and memoization in this way can significantly optimize recursive functions and is a useful technique in many algorithmic scenarios.

## Conclusion

Closures are a powerful and versatile feature in Python. They enable you to create functions that retain access to their surrounding context, making them useful for a wide range of programming tasks. Understanding closures is essential for writing clean, maintainable, and expressive Python code.
