---
layout: post
title: "Playing with Model in Elixir Phoenix Console"
date: 2014-10-08 07:20:21 +0800
comments: true
categories: 
keywords: "model, iex, console, phoenix"
description: "Playing with model in elixir phoenix console"
---

You will need to have the source code of this article http://learnelixir.com/blog/2014/10/04/build-web-app-with-elixir/ before being able to follow this article. So basically, after you finish that article, you will have a model Book. Let turn on elixir console to play around with this model. To turn on elixir console with phoenix, type the following command

<!-- more -->

```bash
$ iex -S mix
```

Now you are inside elixir console mode, let's start with retrieve our first book

```elixir
iex> BookStore.Repo.get(BookStore.Books, 1)
%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun",
  id: 1, publisher: "The Pragmatic Bookshelf", title: "Programming Elixir"}
```

As you can see, we can use ``BookStore.Repo`` to retrieve a book. Here is the review of ``BookStore.Repo`` code

```elixir
defmodule BookStore.Repo do
  use Ecto.Repo, adapter: Ecto.Adapters.Postgres
  
  def conf do
    parse_url "ecto://postgresuser:password@localhost/book_store"
  end 
    
  def priv do
    app_dir(:book_store, "priv/repo")
  end
end
```

From the elixir console, we also can retrieve all books just like what we did in the controller

```elixir
iex> BookStore.Repo.all(BookStore.Books)
[%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun",
  id: 1, publisher: "The Pragmatic Bookshelf", title: "Programming Elixir"}]
```

As you can see, this time round, it returns you an array of Books. Up to this point, I feel a bit annoying every time I need to type ``BookStore.Books``, ``BookStore.Repo``, then I find out that ``alias`` in Elixir can help to shorten this syntax by just calling ``Books`` and ``Repo``. So now, what you need to do is

```elixir
iex> alias BookStore.Repo, as: Repo
nil
iex> alias BookStore.Books, as: Books
```

By default ``alias``, if you call ``alias`` without supplying ``as``, it will automatically set the alias to the last part of the module name. Hence in this case, you can just simply call

```elixir
iex> alias BookStore.Repo
nil
iex> alias BookStore.Books
nil
```

So now, it will be much pleasure to write the command in the console

```elixir
iex> Repo.all(Books)
[%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun",
  id: 1, publisher: "The Pragmatic Bookshelf", title: "Programming Elixir"}]

iex> Repo.get(Books, 1)
%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun",
  id: 1, publisher: "The Pragmatic Bookshelf", title: "Programming Elixir"}
```

Next, you can assign a book from the database call inside the console:

```elixir
iex> book = Repo.get(books, 1)
%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun",
  id: 1, publisher: "The Pragmatic Bookshelf", title: "Programming Elixir"}

iex> book.author
"Dave Thomas"

iex> book.description
"Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun"
```

To update this book, you will need to reassign the book with itself plus the updated properties and let ``Repo`` handle the update by calling ``update`` function on Repor

```elixir
iex>  book = %{book | description: "Programming Elixir: a lot more fun", title: "Programming Elixir with fun"}
%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: a lot more fun", id: 1,
  publisher: "The Pragmatic Bookshelf", title: "Programming Elixir with fun"}

iex> Repo.update(book)
:ok

iex> Repo.get(Books, 1)
%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: a lot more fun", id: 1,
  publisher: "The Pragmatic Bookshelf", title: "Programming Elixir with fun"}
```

Next, you also can create a new book by calling ``insert`` command on ``Repo``. Pretty straight forward.

```elixir
iex> Repo.insert(%Books{author: "Simon St. Laurent, J. David Eisenberg", description: "Elixir is an excellent language if you want to learn about functional programming, and with this hands-on introduction", publisher: "O'Reilly", title: "Introducing Elixir"})
%BookStore.Books{author: "Simon St. Laurent, J. David Eisenberg",
 description: "Elixir is an excellent language if you want to learn about functional programming, and with this hands-on introduction",
  id: 18, publisher: "O'Reilly", title: "Introducing Elixir"}
```

Now try to get all books again

```elixir
iex> Repo.all(Books)
[%BookStore.Books{author: "Dave Thomas",
  description: "Programming Elixir: a lot more fun", id: 1,
  publisher: "The Pragmatic Bookshelf", title: "Programming Elixir with fun"},
 %BookStore.Books{author: "Simon St. Laurent, J. David Eisenberg",
  description: "Elixir is an excellent language if you want to learn about functional programming, and with this hands-on introduction",
  id: 2, publisher: "O'Reilly", title: "Introducing Elixir"}]
```

Note this book is created with id ``2``. Now we can delete this book by calling ``delete`` command on ``Repo``. 

```elixir
iex> introducing_elixir_book = Repo.get(Books, 2)
%BookStore.Books{author: "Simon St. Laurent, J. David Eisenberg",
 description: "Elixir is an excellent language if you want to learn about functional programming, and with this hands-on introduction",
  id: 2, publisher: "O'Reilly", title: "Introducing Elixir"}

iex> Repo.delete(introducing_elixir_book)
:ok
```

Retrieving all books again, you will see the deleted book is really deleted.

```elixir
iex> Repo.all(Books)
[%BookStore.Books{author: "Dave Thomas",
 description: "Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun",
  id: 1, publisher: "The Pragmatic Bookshelf", title: "Programming Elixir"}]
```

That's all for this article. I really hope you enjoy it. These basic command on the model will play a very important role when you need to build a RESTful controller, which will be cover in the near future article :-) 
