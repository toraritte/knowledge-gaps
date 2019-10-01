Bottom line is to use the following:

  + MIGRATION: **`timestamptz`** aka. `timestamp with timezone`
    (the Ecto default is `timestamp` and `:naive_datetime`)

  + SCHEMA: **`:utc_datetime`**
    (the Ecto default is `:naive_datetime`

Basically none of the recommended types are the default.

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

  These   articles  should   be  used   together  (the
  first  shows  how to  set  up  Ecto schemas  to  use
  `:utc_datetime`,  but  no  word of  migrations  with
  `timestamptz`, and vice versa)

  A very handy table from the first article:

  <table>
    <thead>
      <tr>
        <th>Ecto 3 type</th>
        <th>Elixir type</th>
        <th style="text-align: center">Supports microseconds?</th>
        <th style="text-align: center">Supports <a href="https://hexdocs.pm/elixir/DateTime.html#to_unix/2">DateTime functions?</a></th>
        <th style="text-align: center">Supports NaiveDateTime functions?</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><code class="highlighter-rouge">:utc_datetime_usec</code></td>
        <td><code class="highlighter-rouge">DateTime</code></td>
        <td style="text-align: center">✓</td>
        <td style="text-align: center">✓</td>
        <td style="text-align: center">✓</td>
      </tr>
      <tr>
        <td><code class="highlighter-rouge">:utc_datetime</code></td>
        <td><code class="highlighter-rouge">DateTime</code></td>
        <td style="text-align: center">No</td>
        <td style="text-align: center">✓</td>
        <td style="text-align: center">✓</td>
      </tr>
      <tr>
        <td><code class="highlighter-rouge">:naive_datetime_usec</code></td>
        <td><code class="highlighter-rouge">NaiveDateTime</code></td>
        <td style="text-align: center">✓</td>
        <td style="text-align: center">No</td>
        <td style="text-align: center">✓</td>
      </tr>
      <tr>
        <td><code class="highlighter-rouge">:naive_datetime</code></td>
        <td><code class="highlighter-rouge">NaiveDateTime</code></td>
        <td style="text-align: center">No</td>
        <td style="text-align: center">No</td>
        <td style="text-align: center">✓</td>
      </tr>
    </tbody>
  </table>

-------

+ Discussions and advice specific to PostgreSQL

  + [Don't use timestamp (without time zone) (PostgreSQL Wiki)](https://wiki.postgresql.org/wiki/Don%27t_Do_This#Don.27t_use_timestamp_.28without_time_zone.29)

  + [Difference between timestamps with/without time zone in PostgreSQL](https://stackoverflow.com/questions/5876218/difference-between-timestamps-with-without-time-zone-in-postgresql)

  + [Ignoring time zones altogether in Rails and PostgreSQL (accepted answer)](https://stackoverflow.com/questions/9571392/ignoring-time-zones-altogether-in-rails-and-postgresql/9576170#9576170)

  + [8.5. Date/Time Types (PostgreSQL manual)](https://www.postgresql.org/docs/current/datatype-datetime.html)

  + [9.9.3. AT TIME ZONE (PostgreSQL manual)](https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT)
