---
layout: post
title: "Build Web App with Elixir"
date: 2014-10-04 23:22:42 +0800
comments: true
categories: 
---

### Create Elixir Jobs Project

From the phoenix installation folder

```bash
$ mix phoenix.new elixir_jobs ../
```

Now enter the project folder and get all the dependencies and start the phoenix project

```bash
$ cd ../elixir_jobs
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
    mod: { ElixirJobs, [] },
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
defmodule ElixirJobs.Repo do
  use Ecto.Repo, adapter: Ecto.Adapters.Postgres

  def conf do
    parse_url "ecto://postgresuser:password@localhost/elixir_jobs"
  end

  def priv do
    app_dir(:elixir_jobs, "priv/repo")
  end
end
```

We have defined PostgreSQL connection with a URL format, what you will need to do is to change the ``postgresuser`` and ``password`` to be the real postgres username and password on your database

Since we are going to use the migration feature, we will need to have ``priv`` function. Inside this function, we will need to specify where is the migration script is saved, which is inside ``priv/repo`` directory

The next that we must do is to make sure that our Repo module is started with our application, and is supervised. Open ``lib/elixir_jobs.ex``.

```elixir
defmodule ElixirJobs do
  use Application

  # See http://elixir-lang.org/docs/stable/elixir/Application.html
  # for more information on OTP Applications
  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      # Define workers and child supervisors to be supervised
      worker(ElixirJobs.Repo, [])
    ]

    opts = [strategy: :one_for_one, name: ElixirJobs.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

Note that ``line 11`` is the only line that we add inside this file

To make sure that everything is good, let's compile the project:

```bash
$ mix compile
```

Next, let's create the ``elixir_jobs`` database in postgres

```bash
$ createdb elixir_jobs
```

### Create a model

Create model file ``web/models/jobs.ex`` with the following code:

```elixir
defmodule ElixirJobs.Jobs do
  use Ecto.Model

  schema "jobs" do
    field :title, :string
    field :job_type, :string
    field :description, :string
    field :status, :string
  end
end
```

### Generate Migration Script

We will also need to create a database migration for jobs model by using the following command:

```bash
$ mix ecto.gen.migration ElixirJobs.Repo create_job
Compiled lib/elixir_jobs.ex
Compiled web/models/repo.ex
Compiled web/models/jobs.ex
Generated elixir_jobs.app
* creating priv/repo/migrations
* creating priv/repo/migrations/20141004162815_create_job.exs
```

Now open the just generated migration file ``priv/repo/migrations/20141004162815_create_job.exs`` and change with the following code

```elixir
defmodule ElixirJobs.Repo.Migrations.CreateJob do
  use Ecto.Migration

  def up do
    ["CREATE TABLE jobs(id serial primary key, title varchar(125), job_type varchar(50), description text, status varchar(50))",
     "INSERT INTO jobs(title, job_type, description, status) VALUES ('Elixir Expert Needed', 'Remote', 'Elixir expert needed for writing article about Elixir every single week or two.', 'Part Time')"
    ]
  end

  def down do
    "DROP TABLE jobs"
  end
end
```

Inside this migration file, you will see up and down function. ``up`` function is run when you run the database migration, ``down`` function is run when you revert or rollback this database migration

Now run the migration

```bash
$ mix ecto.migrate ElixirJobs.Repo
* running UP _build/dev/lib/elixir_jobs/priv/repo/migrations/20141004162815_create_job.exs
```

### Create A Query

Create a file called ``queries.ex`` inside the ``web/models`` folder. 

```elixir
defmodule ElixirJobs.Queries do
  import Ecto.Query

  def jobs_query do
    query = from job in ElixirJobs.Jobs,
            select: job
    ElixirJobs.Repo.all(query)
  end
end
```

### Route jobs index page to Jobs controller index action

Open file ``web/router.ex``, we will need to map the root route to ``JobController``. Note that in Phoenix, controller name is singular + ``Controller``. In Rails, it is ``JobsController``

```elixir
defmodule ElixirJobs.Router do
  use Phoenix.Router

  get "/", ElixirJobs.JobController, :index, as: :jobs

end
```

### Create JobController and get all jobs

Create file ``job_controller.ex`` inside ``web/controllers`` folder with the following source code

```elixir
defmodule ElixirJobs.JobController do
  use Phoenix.Controller

  def index(conn, _params) do
    jobs = ElixirJobs.Queries.jobs_query
    render conn, "index", jobs: jobs
  end
end
```

### Create jobs index page view

Next thing we will need to do is to create an index page for jobs listing. First, create folder job inside ``web/template``. 

```bash
mkdir web/templates/job
```

Second, create a job view file - ``web/views/job_view.ex`` with the following content

```elixir
defmodule ElixirJobs.JobView do
  use ElixirJobs.Views
end
```

Finally, create file ``web/template/job/index.html.eex`` and paste in the following code:

```html
<h1>List of Jobs</h1>

<table class='table table-bodered table-striped'>
  <thead>
    <tr>
      <th>#</th>
      <th>Title</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <%= for job <- @jobs do %>
      <tr>
        <td><%= job.id %></td>
        <td><%= job.title %></td>
        <td><%= job.description %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Refresh the browser and this is what we get:

{% img left /images/build-web-app-with-elixir/complete.png 800 346 'image' 'images' %}

