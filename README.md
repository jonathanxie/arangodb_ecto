# ArangoDB.Ecto 

Ecto 2.x adapter for [ArangoDB](https://www.arangodb.com/).

It converts `Ecto.Query` structs to [AQL](https://docs.arangodb.com/3.1/AQL).
At the moment the `from`, `where`, `order_by`, `limit`
`offset` and `select` clauses are supported.

For example, this Ecto query:
```elixir
from u in "users",
  where: like(u.name, "A%") and u.age > 18,
  select: [u.name, u.age],
  order_by: [desc: u.name],
  limit: 10
```
results in the following AQL query:
```
FOR u0 IN `users`
  FILTER ((u0.`name` LIKE 'A%') && (u0.`age` > 18))
  SORT u0.`name` DESC
  LIMIT 10
  RETURN { `name`: u0.`name`, `age`: u0.`age` }
```
      
## Usage

In the repository configuration, you need to specify the `:adapter`:

```elixir
config :my_app, MyApp.Repo,
  adapter: ArangoDB.Ecto,
  database_name: "my_app"
  ...
```
   
The following options can be defined to configure the connection to ArangoDB.
Unless specified otherwise, the show default values are used.
```elixir
host: "localhost",
port: 8529,
scheme: "http",
database_name: "_system",
arrango_version: 30_000,
headers: %{"Accept": "*/*"},
use_auth: :basic,
username: nil,
password: nil,
```

When defining your schema you should use `ArangoDB.Ecto.Schema` instead of `Ecto.Schema`.
This implicitly defines the primary key to be the autogenerated `_key` field as required 
by ArangoDB. For that reason you unfortunately also have to manually specify the `foreign_key`
when using associations.
```elixir
defmodule User do
  use ArangoDB.Ecto.Schema

  schema "users" do
    field :name, :string
    field :age, :integer
    has_many :posts, Post, foreign_key: :author__key
    timestamps()
  end
end

defmodule Post do
  use ArangoDB.Ecto.Schema

  schema "posts" do
    field :title, :string
    field :text, :binary
    belongs_to :author, User
    timestamps()
  end
end
```
## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `arangodb_ecto` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [{:arangodb_ecto, "~> 0.1.0"}]
end
```

## Features not yet implemented
* subqueries
* joins
* on conflict
* upserts

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at [https://hexdocs.pm/arangodb_ecto](https://hexdocs.pm/arangodb_ecto).

