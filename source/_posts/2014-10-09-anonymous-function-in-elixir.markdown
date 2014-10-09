---
layout: post
title: "Anonymous function in Elixir"
date: 2014-10-09 08:05:11 +0800
comments: true
categories: 
keywords: "anonymous function, elixir"
description: "Anonymous function in Elixir"
---

This is one of my favorite topic when talking about anonymous function. The reason why is that anonymous function in Elixir is a very simple way to express something without having to turn on a text editor. I can specify an anonymous function right away in an Elixir console and test it straight away. Here's how:

<!-- more -->

From your elixir terminal, you can quickly define an anonymous function like below:

```elixir
iex> itself = fn n -> n end
#Function<6.90072148/1 in :erl_eval.expr/5>
```

Anonymous function starts with ``fn`` keyword, following by a list of arguments, then the symbol ``->``, then the expression and ends with an ``end`` keyword. You can define an anonymous function with many arguments like below

```elixir
iex> sum = fn a, b -> a + b end
#Function<12.90072148/2 in :erl_eval.expr/5>
```

Now, in order to call this anonymous function, you will need to use the ``dot`` notation. This is a bit different from calling a function defined in ``defmodule``:

```elixir
iex> sum = fn a, b -> a + b end
#Function<12.90072148/2 in :erl_eval.expr/5>
iex> sum.(5, 10)
15
```

Some body like me will feel a bit ackward in the beginning when we need to use ``dot`` notation to call an anonymous function. However, it is true that this ``dot`` notation is used to differentiate between calling a function without any arguments and using a variable.  For instance:

```elixir
iex> my_self = fn -> 5 end
#Function<6.90072148/1 in :erl_eval.expr/5>
iex> other_self = 5
5
iex> my_self.() # I am calling an anonymous function
5
iex> other_self # I am using a variable
5
```

Like Javascript, function in Elixir is first class citizen. Specifically, this means the language supports passing functions as arguments to other functions, returning them as the values from other functions, and assigning them to variables or storing them in data structures. 

So let's see how we can pass a function as an argument to another function:

```elixir
defmodule ListUtils do
  def select([], _check), do: []
  def select([head | tail], check) do
    if check.(head) do
      [head | select(tail, check)]
    else
      select(tail, check)
    end
  end
end

is_odd = fn n -> rem(n, 2) == 1 end
ListUtils.select([1, 2, 3, 4], is_odd)
```

- Here is we define a ``ListUtils`` module which contains ``select`` function to filter based on the argument ``check`` function. 
- On line 12, we define an anonymous function and assign that function to variable ``is_odd``.
- On line 13, we pass the ``is_odd`` variable (now is bound to the anonymous) as an argument to ``select`` function.
- You can expect the result returning from the ``select`` function call is ``[1, 3]``

So this is how an anonymous function can be assigned to a variable and passed to another function. In addition, we can directly supply the anonymous function into the function call without first assigning to a variable like below and it still works!

```elixir
ListUtils.select([1, 2, 3, 4], fn n -> rem(n, 2) == 1 end)
```

Further more, in Elixir, there is a short hand syntax that can convert the line above into following:

```elixir
ListUtils.select([1, 2, 3, 4], &(rem(&1, 2) == 1))
```

We have transformed the anonymous function from ``fn n -> rem(n, 2) == 1 end`` to ``&(rem(&1, 2) == 1)``. In the later case, ``&1`` is actually ``n`` variable in first case. I usually do not recommend to use this for multiple argument function as it might confuse the reader
. However, it is still readable and understandable for just 1 argument. Anyway, for your information, multiple argument function can be rewritten as:

```elixir
sum = fn a, b -> a + b end
# is equivalent to 
sum = &(&1 + &2)
```

That's all for this article. I hope you enjoy this article if you reach this far. Please leave some comments if you have any questions or recommendations :-)

