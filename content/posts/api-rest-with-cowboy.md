---
author: "Juan Carlos Martinez de la Torre"
date: 2019-01-09
linktitle: api-rest-cowboy
menu:
  main:
    parent: tutorials
next: /tutorials/github-pages-blog
prev: /tutorials/automated-deployments
title: Api Rest With Cowboy
description: This tutorial will show you how to create a simple theme in Hugo.
weight: 10
---
At Elixir world the well know framework **Phoenix** is the main tool to develop the client side interface in any project. This framework allows us to develop complex real-time web systems simplily with a lot of integrated features such as websockets. But in the case of just pretending to build a simple api Rest a good option is to use the **Cowboy** library, which makes quite more lightweight the final application than using Phoenix.

In this post I will expose how to build a simple Rest Api Service with **Cowboy**.


Create New project with Elixir
```bash
mix new cowboy_example
```

Add cowboy dependency at ```mix.exs```

``` elixir
  defp deps do
    [
      {:cowboy, "~> 2.0"}
    ]
  end
```

Let's get and compile this dependency:
```bash
mix deps.get && mix deps.compile
```

Let's use the ```lib/cowboy_rest.ex``` file as application entrypoint for supervision tree. We need to indicate this on our ```mix.exs``` file:

```elixir
```

And set ```lib/cowboy_rest.exs``` as an application module so his content will be:
``` Elixir
defmodule PostgameAnalyzer do
  use Application

  def start(_type, _args) do
    children = [
    # No childrent yet
    ]

    opts = [strategy: :one_for_one, name: PostgameAnalyzer.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```


We will create the folder ```lib/cowboy_rest``` where submodules will be included. In our case the only submodule will be the web one at ```lib/cowboy_rest/web```.

At this point we can start working on Cowboy on the Web submodule. We will need:

*   ```lib/cowboy_rest/web/supervisor```: This will be the Supervisor of the Web submodule, in carge of supervising the cowboy system.

*  ```lib/cowboy_rest/http_listener```: This file will hold the roueing to the api Rest handlers. This will be also in charge of launching Cowboy Listener at start.

* ```lib/cowboy_rest/rest_handler```: It is a Rest Handler in charge of handling and answering the incoming requests.

