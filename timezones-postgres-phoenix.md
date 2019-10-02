Bottom line is to use the following:

  + MIGRATION: **`timestamptz`** aka. `timestamp with timezone`
    (the Ecto default is `timestamp` and `:naive_datetime`)

  + SCHEMA: **`:utc_datetime`**
    (the Ecto default is `:naive_datetime`

Basically none of the recommended types are the default.

## ECTO MIGRATIONS: Switch to `timestamptz` and `:utc_datetime`

**Note**: [Ecto.Migration.timestamps/1](https://hexdocs.pm/ecto_sql/Ecto.Migration.html#timestamps/1) ([source](https://github.com/elixir-ecto/ecto_sql/blob/v3.2.0/lib/ecto/migration.ex#L925)) global configuration can always be overridden locally.

### 1. Global configuration

Using the `:migration_timestamps` configuration option from the [`Ecto.Migration` docs](https://hexdocs.pm/ecto_sql/Ecto.Migration.html#module-repo-configuration):

```elixir
# in ./config/dev.exs (for example)

config :app, App.Repo, migration_timestamps: [type: :timestamptz]
```

and one can use `Ecto.Migration.timestamps/1` in migrations as usual:

```elixir
# ./priv/repo/migrations/20190718195828_create_users.exs

create table(:users) do
  add :username, :string, null: false

  timestamps()
end
```


The  `Postgres`  adapter will  automatically  switch
the   Elixir  representation   to  `DateTime`   from
`NaiveDateTime`.

### 2. Local configuration

Use [Ecto.Migration.timestamps/1](https://hexdocs.pm/ecto_sql/Ecto.Migration.html#timestamps/1)'s `:type` option:

```elixir
defmodule App.Repo.Migrations.CreateUsers do

  use Ecto.Migration

  def change do
    create table(:users) do
      add :username, :string, null: false

      timestamps(type: :timestamptz)
    end
  end
end
```

## ECTO SCHEMAS: Switch to `:utc_datetime`

### 1. Global configuration

The  Ecto  schemas  also  need  to  be  modified  to
use  `:utc_datetime`,  otherwise  they  will  expect
`NaiveDateTime` by  default. Slightly  modifying the
example in the
[`Ecto.Schema` docs](https://hexdocs.pm/ecto/Ecto.Schema.html#module-schema-attributes):

> ```elixir
> # Define a module to be used as base
> defmodule MyApp.Schema do
>   defmacro __using__(_) do
>     quote do
>       use Ecto.Schema
>
>       # In case one uses UUIDs
>       @primary_key {:id, :binary_id, autogenerate: true}
>       @foreign_key_type :binary_id
>
>       # ------------------------------------
>       @timestamps_opts [type: :utc_datetime]

>     end
>   end
> end
>
> # Now use MyApp.Schema to define new schemas
> defmodule MyApp.Comment do
>   use MyApp.Schema
>
>   schema "comments" do
>     belongs_to :post, MyApp.Post
>
>     timestamps()
>   end
> end
> ```

### 2. Local configuration

```elixir
defmodule ANV.Accounts.User do

  use Ecto.Schema

  # -- EITHER --------------------------
  @timestamps_opts [type: :utc_datetime]

  schema "users" do

    field :username, :string

    # -- OR -----------------------
    timestamps(type: :utc_datetime)
  end
```

## Resources

+ [Difference in between :utc_datetime and :naive_datetime in Ecto](https://elixirforum.com/t/difference-in-between-utc-datetime-and-naive-datetime-in-ecto/12551)

-------

+ [lau/tzdata](https://github.com/lau/tzdata)

  [`DateTime`](https://hexdocs.pm/elixir/DateTime.html#from_naive/3)
  in   Elixir  "_only   handles  "Etc/UTC"
  datetimes_" but  it can be configured  with a custom
  time  zone  database,  which is  what  the  `tzdata`
  library is

-------

+ [Time zones in PostgreSQL, Elixir and Phoenix](https://www.amberbit.com/blog/2017/8/3/time-zones-in-postgresql-elixir-and-phoenix/) and
  [How to set timestamps to UTC DateTimes in Ecto](http://www.creativedeletion.com/2019/06/17/utc-timestamps-in-ecto.html)

  A very handy table from the first article:

```
+----------------------+------------------+------------------------+------------------------------+-----------------------------------+
|    Ecto 3 type       |    Elixir type   | Supports microseconds? | Supports DateTime functions? | Supports NaiveDateTime functions? |
+----------------------+------------------+------------------------+------------------------------+-----------------------------------+
| :utc_datetime_usec   | DateTime         |    YES                 |   YES                        |   YES                             |
| :utc_datetime        | DateTime         |    NO                  |   YES                        |   YES                             |
| :naive_datetime_usec | NaiveDateTime    |    YES                 |   NO                         |   YES                             |
| :naive_datetime      | NaiveDateTime    |    NO                  |   NO                         |   YES                             |
+----------------------+------------------+------------------------+------------------------------+-----------------------------------+

```

-------

+ Discussions and advice specific to PostgreSQL

  + [Don't use timestamp (without time zone) (PostgreSQL Wiki)](https://wiki.postgresql.org/wiki/Don%27t_Do_This#Don.27t_use_timestamp_.28without_time_zone.29)

  + [Difference between timestamps with/without time zone in PostgreSQL](https://stackoverflow.com/questions/5876218/difference-between-timestamps-with-without-time-zone-in-postgresql)

  + [Ignoring time zones altogether in Rails and PostgreSQL (accepted answer)](https://stackoverflow.com/questions/9571392/ignoring-time-zones-altogether-in-rails-and-postgresql/9576170#9576170)

  + [8.5. Date/Time Types (PostgreSQL manual)](https://www.postgresql.org/docs/current/datatype-datetime.html)

  + [9.9.3. AT TIME ZONE (PostgreSQL manual)](https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT)
