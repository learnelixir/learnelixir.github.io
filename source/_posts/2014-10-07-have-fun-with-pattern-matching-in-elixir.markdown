---
layout: post
title: "Have Fun with Pattern Matching in Elixir"
date: 2014-10-07 22:28:03 +0800
comments: true
categories: 
keywords:  "pattern matching, fun, elixir"
description:  "Have fun with pattern matching in Elixir"
---

In this article, we will look deeper into pattern matching in Elixir. Coming from prologue background, I think pattern matching is quite a fun thing to do. Prologue is a too hard core language for this kind of thing, whereas Elixir is mixing just well between normal programming language and pattern matching...

<!-- more -->

First, let's look at the very basic pattern matching in Elixir:

```elixir
iex> variable = 1
1
```

In a common programming language like Ruby or Java, this is simple an assignment. However, in Elixir, this assignment operator is called the match operator. Here is why

```elixir
iex> variable = 1
1
iex> 1 = variable
1
```

In Ruby or Java, the expression ``1 = variable`` will give you a syntax error because 1 cannot be assigned to a variable. But in Elixir, it is just fine, because 1 and variable are matching in value. However, you cannot match an undefined variable with a number when the variable in on the right hand side

```elixir
iex> 2 = x
** (RuntimeError) undefined function: x/0
```

Without pre-definining ``x`` variable, Elixir will think that you are trying to call a function ``x`` (``x/0``) without any argument. Now, let's go further to matching few variables at the same time

```elixir
iex> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex> a
1
iex> b
2
iex> c
3
```

After this matching operation, ``a``, ``b`` and ``c`` will have value ``1``, ``2`` and ``3`` respectively. This is extremely useful when you need to assign a list of variables or when you return a function as a tuple. We can now go a bit further by first trying to understand a definition of a list:

__A list is a series of element which can have 0 elements.__

So based on this definition, I can write a list as following

```bash
a list is [] # empty list
# or
a list is [element1, element2, element3, element4, etc...]
```

The first is easy to understand, a list can be empty. The second list basically shows that it can have many elements, which can be understood that a list is combined from ``element1`` and the rest of the element:

```bash
list is [element1, and the remaining]
```

In mathematics, we can say

```bash
list = [head | tail]
```

Where ``head`` is ``element1`` and ``tail`` is the remaining. So now we apply this into Elixir pattern matching


```elixir
iex> list = [1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
iex> [head | tail] = list 
[1, 2, 3, 4, 5]
```
Because of patter matching, ``head`` is now carrying value ``1``

```elixir
iex> head
1
```

And tail is now carrying the remaining of the list which is:

```elixir
iex> tail
[2, 3, 4, 5]
```

So if we want to get our all elements of this list, we can just simply continue to assign like following:

```elixir
iex> [head1 | tail1] = tail
[2, 3, 4, 5]
iex> head1
2
iex> tail1
[3, 4, 5]
```

So on so forth we will be able to extract every single element of this list using pattern matching. You can note that we never talk about a for loop, yet we still can go through the loop easily using Pattern Matching.

The applications of pattern matching is endless, but I am going to show you a classic pattern matching by trying to calculate a sum of a list. 

```elixir
defmodule Utils do
  def sum([]), do: 0
  def sum([head | tail]), do: head + sum(tail)
end

Utils.sum([]) # return 0
Utils.sum([9, 8, 7, 6]) # returns 30
```

Let's go through quickly how this work:

- On line 2, we definied a sum function for an empty list (pattern matching an empty list in sum function's argument) and return 0.
- On line 3, we defined another sum function but with a non empty list, this function then returns the result as the value of ``head`` + the sum of the remaining inside the list (``sum(tail)``). It is actually a recursion by calling the function itself in its evaluation.

So by now you should have a basic understanding how Elixir pattern matching works. Eventually, you will see more and more this type of pattern matching throughout Elixir Programming.

