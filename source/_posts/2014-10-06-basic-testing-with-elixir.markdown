---
layout: post
title: "Basic Testing with Elixir"
date: 2014-10-06 22:31:16 +0800
comments: true
categories: 
description: "Basic Testing concept in Elixir"
keywords: "elixir, test, TDD"
---

In this article, we will go through some basic testing syntax with elixir. Elixir is a language that very well supports unit test and I totally love it when I first see it on the video. Let's start by writing a simple assertion
<!--more-->
You can use your favorite editor to edit an ``.ex`` file. In my case, I create a test folder and edit my ``sample.ex`` inside

```bash
$ mkdir test
$ vim test/sample.ex
```

Now, inside this ``sample.ex``, I can immediately write my first assertion like below

```elixir
ExUnit.start                                                      
                                                                  
defmodule TestIt do                                               
  use ExUnit.Case                                                 
                                                                  
  test "1 + 1 is 2" do                                            
    assert 1 + 1 == 2                                             
  end                                                             
end
```

As you can see, the test case can be written in a very simple manner. It starts with ``ExUnit.start`` and then make use of ``ExUnit.Case`` in ``use ExUnit.Case``

You can then run this file by this command

```bash
$ elixir sample.ex
.

Finished in 0.03 seconds (0.03s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 322828
```

That's for the start, now we can following a Test Driven Development method (TDD) to develop something basic just for learning purpose. Let's write fibonacci generate. 

Giving a sequence number, the program will generate out a fibonacci number. Before we start writing, let's review the definition of fibonacci number 

```bash
fibonacci(0) = 1
fibonacci(1) = 1
fibonacci(n) = fibonacci(n - 1) + fibonacci(n - 2) 
```

With this definition in mind, let start by writing the first test:

```elixir
ExUnit.start                                                      
                                                                  
defmodule TestIt do                                               
  use ExUnit.Case                                                 
                                                                  
  test "fibonacci number 0 is 1" do
    assert Sequence.fibonacci(0) == 1
  end
end
```

Run this test and you can expect a failure:

```bash
$ elixir sample.ex

  1) test fibonacci number 0 is 1 (TestIt)
     sample.ex:6
     ** (UndefinedFunctionError) undefined function: Sequence.fibonacci/1 (module Sequence is not available)
     stacktrace:
       Sequence.fibonacci(0)
       sample.ex:7



Finished in 0.03 seconds (0.03s on load, 0.00s on tests)
1 tests, 1 failures

Randomized with seed 490020
```

As you can expect, the failure is because we do not have any module ``Sequence`` and function ``fibonacci``. Let go to ``sample.ex`` and add in a function fibonacci:

```elixir
ExUnit.start                                                      
                                                                  
defmodule Sequence do
  def fibonacci(n) do
    1
  end
end

defmodule TestIt do
  use ExUnit.Case
  
  test "fibonacci number 0 is 1" do 
    assert Sequence.fibonacci(0) == 1
  end
end
```

Now we have ``fibonacci`` function in ``Sequence`` module, which takes in a number and always return 1. Run the test again

```bash
elixir sample.ex
.

Finished in 0.04 seconds (0.04s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 311294
```

Great! It passed. We are at green state now. We can now refactor a bit to make our source code a bit shorter:

```elixir
ExUnit.start                                                      
                                                                  
defmodule Sequence do
  def fibonacci(n), do: 1
end

defmodule TestIt do
  use ExUnit.Case
  
  test "fibonacci number 0 is 1" do
    assert Sequence.fibonacci(0) == 1
  end
end
```

I have moved the block inside ``def fibonacci`` to become something shorter. In fact, ``def`` is a function call in Elixir, which takes in a pattern (``fibonacci(n)``) and ``do`` something. This something needs to be a regular expression.

Now run the test again and you would expect it to pass

```bash
$ elixir sample.ex
sample.ex:4: warning: variable n is unused
.

Finished in 0.04 seconds (0.04s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 999642
```

You can ignore the warning of unused variable n for now. We will come back to that shortly. Now, let's write some more test cases:

```elixir
ExUnit.start 

defmodule Sequence do
  def fibonacci(n), do: 1
end

defmodule TestIt do
  use ExUnit.Case

  test "fibonacci number 0 is 1" do
    assert Sequence.fibonacci(0) == 1
  end

  test "fibonacci number 1 is 1" do
    assert Sequence.fibonacci(1) == 1
  end

  test "fibonacci number 2 is 2" do
    assert Sequence.fibonacci(2) == 2 
  end

  test "fibonacci number 3 is 3" do
    assert Sequence.fibonacci(2) == 2 
  end

  test "fibonacci number 4 is 5" do
    assert Sequence.fibonacci(2) == 2 
  end

  test "fibonacci number 10 is 89" do
    assert Sequence.fibonacci(10) == 89 
  end
end
```

And when run the test, it will fail as expected

```bash
$ elixir sample.ex
  1) test fibonacci number 2 is 2 (TestIt)
     sample.ex:18
     Assertion with == failed
     code: Sequence.fibonacci(2) == 2
     lhs:  1
     rhs:  2
     stacktrace:
       sample.ex:19



  2) test fibonacci number 3 is 3 (TestIt)
     sample.ex:22
     Assertion with == failed
     code: Sequence.fibonacci(2) == 2
     lhs:  1
     rhs:  2
     stacktrace:
       sample.ex:23



  3) test fibonacci number 4 is 5 (TestIt)
     sample.ex:26
     Assertion with == failed
     code: Sequence.fibonacci(2) == 2
     lhs:  1
     rhs:  2
     stacktrace:
       sample.ex:27

..

  4) test fibonacci number 10 is 89 (TestIt)
     sample.ex:30
     Assertion with == failed
     code: Sequence.fibonacci(10) == 89
     lhs:  1
     rhs:  89
     stacktrace:
       sample.ex:31



Finished in 0.06 seconds (0.06s on load, 0.00s on tests)
6 tests, 4 failures

Randomized with seed 982355
```

Now let try to write the code to pass these cases using the definition

```elixir
defmodule Sequence do
  def fibonacci(n) when n <= 1, do: 1
  def fibonacci(n), do: fibonacci(n - 1) + fibonacci(n - 2)
end
```

Note that ``when n <= 1`` in the first function definition is Elixir guard conidtion``. Now, run the test and everything is passed

```bash
$ elixir sample.ex
......

Finished in 0.06 seconds (0.06s on load, 0.00s on tests)
6 tests, 0 failures

Randomized with seed 456516
```

Since elixir function definition is patterned matching. You also can write as following to match according to the mathematic definition

```elixir
defmodule Sequence do
  def fibonacci(0), do: 1
  def fibonacci(1), do: 1
  def fibonacci(n), do: fibonacci(n - 1) + fibonacci(n - 2)
end
```

Here we do not need a guard condition for n in third function as it will match first and second if n is 0 or 1. The function matching is top to bottom. Run the test again

```bash
$ elixir sample.ex
......

Finished in 0.06 seconds (0.06s on load, 0.00s on tests)
6 tests, 0 failures

Randomized with seed 163178
```

We concluded this article here and hope you have learned and enjoyed the following from Elixir. Thank you for your time

- TDD in Elixir
- ExUnit Test
- Function Pattern Matching
- Guard Conditions
