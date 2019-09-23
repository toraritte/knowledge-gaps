```text
connection
|> endpoint()
|> router()
|> pipelines()
|> controller()
  |> common_services() # i.e., plugs
  |> action()
```

> Models   access  data,   views  present   data,  and
> controllers coordinate between the  two. In a sense,
> the purpose  of a web  server is to get  requests to
> functions that perform the right task
(*action*, in Phoenix parlance)

> Plugs are functions.
> Your web applications are pipelines of plugs.

```text
```

```text
```
