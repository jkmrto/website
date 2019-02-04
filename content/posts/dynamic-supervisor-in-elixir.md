---
author: "Juan Carlos Martinez de la Torre"
date: 2019-02-03
linktitle: dynamic-supervisor
menu:
  main:
    parent: tutorials
next: /tutorials/github-pages-blog
prev: /tutorials/automated-deployments
title: Dynamic Supervisor Example
description: This post will expose how to build a bais dynamic supervisor in Elixir.
weight: 10
---

One of the main feature of [Elixir](https://elixir-lang.org/ ) is the ability to guarantee that if a supervised process fails or get crashed for any reason, other process with the same functinality will be started as soon as the supervisor process realizes the problem. This is related to the fault-tolerance capability.

Let's say that we have an application in which at start we dont know how many processes we will have, because they will be generated dinamically during the running of the application. For example if we have a game application which allows several games at the same time and we want to get associated to each game one process, then we will need to dinamically launch a process per each game.

Since Elixir 1.6, [Dynamic Supervisor](https://hexdocs.pm/elixir/DynamicSupervisor.html)  is the module that makes simpler this task. In this post it will be exposed how to build a basic dynamic supervisor, using Elixir [Registry](https://hexdocs.pm/elixir/master/Registry.html) to keep reachable all the launched processes.

# Setup

Let's create a new project using ```mix``` through the command line with:
```Bash
mix new dynamic_supervisor_example
```

We will use the module at ```lib/dynamic_supervisor_example``` as application entrypoint so his content at first would be:

```Elixir
# lib/dynamic_supervisor_example.ex
defmodule DynamicSupervisorExample do
  use Application

  def start(_type, _args) do
    children = [
      # No childrent yet
    ]

    opts = [strategy: :one_for_one, name: __MODULE__]
    Supervisor.start_link(children, opts)
  end
end
```

We need to modify our file ```mix.exs``` to indicate the new application entrypoint:
```Elixir
............
  def application do
    [
      mod: {DynamicSupervisorExample, []},
      extra_applications: [:logger]
    ]
  end
............
```

# Dynamic Supervisor under Supervision Tree

Our dynamic supervisor module will be placed at ``` lib/dynamic_supervisor_example/worker_supervisor.ex``` with this content at first:
```Elixir
# lib/dynamic_supervisor_example/workers_supervisor.ex
defmodule DynamicSupervisorExample.WorkersSupervisor do
  use DynamicSupervisor

  def start_link(_arg),
    do: DynamicSupervisor.start_link(__MODULE__, [], name: __MODULE__)

  def init(_arg),
    do: DynamicSupervisor.init(strategy: :one_for_one)

end
```

We will add this new module as a children to the application entrypoint:

``` Elixir
# lib/dynamic_supervisor_example.ex
......
    children = [
      { DynamicSupervisorExample.WorkersSupervisor, [] }
    ]
......
```






# Still doing