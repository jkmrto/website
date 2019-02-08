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


# Components

The objective si to have 

## Application entrypoint

We need to use an application entrypoint where starts the supervision tree, the file. This module will be on charge of supervise the `Registry` and the `DynamicSupervisorWithRegistry.WorkerSupervisor`.

``` Elixir
#lib/dynamic_supervisor_example.ex
defmodule DynamicSupervisorWithRegistry do
  use Application # Indicate this module is an application entrypoint

  @registry :worker_registry

  def start(_args, _opts) do
    children = [
      { DynamicSupervisorWithRegistry.WorkerSupervisor, [] },
      { Registry, [keys: :unique, name: @registry]}
    ]

    # :one_to_one strategy indicates only the crashed child will be restarted, without affecting the rest of children.
    opts = [strategy: :one_for_one, name: __MODULE__] 
    Supervisor.start_link(children, opts)
  end
end
```

As children of this module we have:
  * **Registry**: Allow to register the workers by a custom name, that will allow to acess the workers easily, without needing to know its *pid*
  * **DynamicSupervisorWithRegistry.WorkerSupervisor**. Dynamic supervisor en charge on supervising future workers. 

## Workers Supervisor
This module should just supervise the workers and allow to launch new workers.

``` Elixir
# lib/dynamic_supervisor_example/worker_supervisor.ex
defmodule DynamicSupervisorWithRegistry.WorkersSupervisor do
  use DynamicSupervisor
  alias DynamicSupervisorWithRegistry.Worker

  def start_link(_arg),
    do: DynamicSupervisor.start_link(__MODULE__, [], name: __MODULE__)

  def init(_arg),
    do: DynamicSupervisor.init(strategy: :one_for_one)

  def start_child(child_name) do
    DynamicSupervisor.start_child(
      __MODULE__,
      %{id: Worker, start: { Worker, :start_link,  [child_name]}, restart: :transient})
  end

end
```

It is important to note that workers will be launched with ``` restart: :transient ``` will be only restated if they terminate due to an error not it was a ```:normal``` termination

## Worker

```
# lib/dynamic_supervisor_example/worker.ex
defmodule DynamicSupervisorWithRegistry.Worker do
  use GenServer
  require Logger

  @registry :worker_registry

  ## API
  def start_link(name),
    do: GenServer.start_link(__MODULE__, name, name: via_tuple(name))

  def stop(name), do: GenServer.stop(via_tuple(name))

  def crash(name), do: GenServer.cast(via_tuple(name), :raise)

  ## Callbacks
  def init(name) do
    Logger.info("Starting #{inspect(name)}")
    {:ok, name}
  end

  def handle_cast(:work, name) do
    Logger.info("hola")
    {:noreply, name}
  end

  def handle_cast(:raise, name),
    do: raise RuntimeError, message: "Error, Server #{name} has crashed"

  def terminate(reason, name) do
    Logger.info("Exiting worker: #{name} with reason: #{inspect reason}")
  end

  ## Private
  defp via_tuple(name) ,
    do: {:via, Registry, {@registry, name} }

end
```


# Running it
```
DynamicSupervisorWithRegistry.WorkerSupervisor.start_child("worker_1")
DynamicSupervisorWithRegistry.Worker.
GenServer.cast(pid, :work)
Registry.lookup(:worker_registry, 345)

DynamicSupervisorWithRegistry.Worker.crash("worker_1")

Registry.count(:worker_registry)

 DynamicSupervisorWithRegistry.Worker.stop("worker_1")
```



# Still doing