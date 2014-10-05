---
layout: post
title: "Book Listing App with Elixir, Phoenix, Postgres and Ecto"
date: 2014-10-04 23:22:42 +0800
comments: true
categories: 
---

### Create Elixir Book Store Project

From the phoenix installation folder

```bash
$ mix phoenix.new book_store ../
```

Now enter the project folder and get all the dependencies and start the phoenix project

```bash
$ cd ../book_store
$ mix do deps.get, compile
$ mix phoenix.start
```

Open the browser, and go to the url ``http://localhost:4000``

{% img left /images/build-web-app-with-elixir/phoenix_page.png 800 510 'image' 'images' %}

### Add Ecto To The Project

From the project root folder, open file ``mix.exs``, scroll down to the end of the file you will see ``defp deps do`` function definition. You will need to add in ``postgrex`` and ``ecto`` dependencies

- ``postgrex`` (https://github.com/ericmj/postgrex) is the PostgresSQL driver for Elixir
- ``ecto`` (https://github.com/elixir-lang/ecto) is a database wrapper and language integrated query for Elixir 

```elixir
defp deps do
  [
    {:phoenix, "0.4.1"},
    {:cowboy, "~> 1.0.0"},
    {:postgrex, "~> 0.5"},
    {:ecto, "~> 0.2.0"}
  ]
end
```

In the same file, you will also need to update the application function definition to include ``postgrex`` and ``ecto``

```elixir
def application do
  [ 
    mod: { BookStore, [] },
    applications: [:phoenix, :cowboy, :logger, :postgrex, :ecto]
  ] 
end 
```

Run the following commands in the terminal to get all the dependencies.

```bash
$ mix deps.get
```

### Create A Repo

A repo is a basic interfacte to a database (which is postgres). You will need to open ``web/models/repo.ex`` and add the following code

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

We have defined PostgreSQL connection with a URL format, what you will need to do is to change the ``postgresuser`` and ``password`` to be the real postgres username and password on your database

Since we are going to use the migration feature, we will need to have ``priv`` function. Inside this function, we will need to specify where is the migration script is saved, which is inside ``priv/repo`` directory

The next that we must do is to make sure that our Repo module is started with our application, and is supervised. Open ``lib/book_store.ex``.

```elixir
defmodule BookStore do
  use Application

  # See http://elixir-lang.org/docs/stable/elixir/Application.html
  # for more information on OTP Applications
  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      # Define workers and child supervisors to be supervised
      worker(BookStore.Repo, [])
    ]

    opts = [strategy: :one_for_one, name: BookStore.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

Note that ``line 11`` is the only line that we add inside this file

To make sure that everything is good, let's compile the project:

```bash
$ mix compile
```

Next, let's create the ``book_store`` database in postgres

```bash
$ createdb book_store 
```

### Create a model

Create model file ``web/models/books.ex`` with the following code:

```elixir
defmodule BookStore.Books do
  use Ecto.Model

  schema "books" do
    field :title, :string
    field :description, :string
    field :author, :string
    field :publisher, :string
  end
end
```

### Generate Migration Script

We will also need to create a database migration for books model by using the following command:

```bash
$ mix ecto.gen.migration BookStore.Repo create_book
Compiled lib/book_store.ex
Compiled web/models/repo.ex
Generated book_store.app
* creating priv/repo/migrations
* creating priv/repo/migrations/20141005013526_create_book.exs
```

Now open the just generated migration file ``priv/repo/migrations/20141005013526_create_book.exs`` and change with the following code

```elixir
defmodule BookStore.Repo.Migrations.CreateBook do
  use Ecto.Migration

  def up do
    ["CREATE TABLE books(id serial primary key, title varchar(125), description text, author varchar(255), publisher varchar(255))",
     "INSERT INTO books(title, description, author, publisher) VALUES ('Programming Elixir', 'Programming Elixir: Functional |> Concurrent |> Pragmatic |> Fun', 'Dave Thomas', 'The Pragmatic Bookshelf')"
    ]
  end

  def down do
    "DROP TABLE books"
  end
end
```

Inside this migration file, you will see up and down function. ``up`` function is run when you run the database migration, ``down`` function is run when you revert or rollback this database migration

Now run the migration

```bash
$ mix ecto.migrate BookStore.Repo
* running UP _build/dev/lib/book_store/priv/repo/migrations/20141005013526_create_book.exs``
```

### Create A Query

Create ``web/models/queries.ex`` folder. 

```elixir
defmodule BookStore.Queries do
  import Ecto.Query

  def books_query do
    query = from book in BookStore.Books,
            select: book 
    BookStore.Repo.all(query)
  end
end
```

### Route books index page to Book controller index action

Open file ``web/router.ex``, we will need to map the root route to ``BookController``. Note that in Phoenix, controller name is singular + ``Controller`` whereas in Rails, it is ``BookController``

```elixir
defmodule BookStore.Router do
  use Phoenix.Router

  get "/", BookStore.BookController, :index, as: :books

end
```

### Create BookController and get all books 

Create file ``book_controller.ex`` inside ``web/controllers`` folder with the following source code

```elixir
defmodule BookStore.BookController do
  use Phoenix.Controller

  def index(conn, _params) do
    books = BookStore.Queries.books_query
    render conn, "index", books: books 
  end
end
```

### Create books index page view

Next thing we will need to do is to create an index page for books listing. First, create folder book inside ``web/template``. 

```bash
mkdir web/templates/book
```

Second, create a book view file - ``web/views/book_view.ex`` with the following content

```elixir
defmodule BookStore.BookView do
  use BookStore.Views
end
```

Finally, create file ``web/template/book/index.html.eex`` and paste in the following code:

```html
<h1>Our Books</h1>

<table class='table table-bodered table-striped'>
  <thead>
    <tr>
      <th>#</th>
      <th>Title</th>
      <th>Description</th>
      <th>Author</th>
      <th>Publisher</th>
    </tr>
  </thead>
  <tbody>
    <%= for book <- @books do %>
      <tr>
        <td><%= book.id %></td>
        <td><%= book.title %></td>
        <td><%= book.description %></td>
        <td><%= book.author %></td>
        <td><%= book.publisher %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Refresh the browser and voila, this is what we will get:

{% img left /images/build-web-app-with-elixir/complete.png 800 346 'image' 'images' %}


### Common Pitfall

I seldom hit the following error when trying to restart phoenix although all the codes are correct 
```bash
=INFO REPORT==== 5-Oct-2014::09:57:47 ===
    application: logger
    exited: stopped
    type: temporary
** (Mix) Could not start application book_store: exited in: BookStore.start(:normal, [])
    ** (EXIT) an exception was raised:
        ** (UndefinedFunctionError) undefined function: BookStore.start/2 (module BookStore is not available)
            BookStore.start(:normal, [])
            (kernel) application_master.erl:272: :application_master.start_it_old/4
```

To fix this, you will need to clean compile all your elixir code: 

```bash
$ mix clean
$ mix compile
$ mix phoenix.start
```


