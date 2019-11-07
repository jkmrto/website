---
author: "Juan Carlos Martinez de la Torre"
date: 2019-09-11
linktitle: docker-with-erlang-distributed
menu:
  main:
    parent: tutorials
next: /tutorials/github-pages-blog
prev: /tutorials/automated-deployments
title: Docker with Erlang Distributed
description: This tutorial will show you how to create a simple theme in Hugo.
weight: 10
---

# Intro 

Last days I have been diving a little into the wonderful world of Erlang distributed. Elixir, thanks to that it is based on Erlang, has some built-in language constructs for distributed systems which make it easier to distribute systems in comparison with other programming paradigms.

Since Erlang is based on the actor model (where each actor is a process) it is transparent to erlang if the actor is local or if it is in a remote host. At the end all we have to guarante is that the destiny process is connected on the same cluster that our node.

Another tool I love to user is Docker, so I started to dive into how to connect two Elixir instances dockerized, it has some tricks that we will discover along this posts.

# Connecting using --net flag

In this first attempt we are going to launch two dockers using the latest elixir image, we will have opened two `iex` terminals where we can play with the nodes.

```console
docker run -i -t --net=host elixir iex --sname node1 --cookie cookie
```
And our second node:
``` console
docker run -i -t --net=host elixir iex --sname node2 --cookie cookie
```

 At using [--net=host](https://docs.docker.com/network/host/) option we are making using of the same network interface between the two dockers, so at some point this scenario is quite similar to the one we have if we launch the two elixir instances locally.

Let's check if our two nodes are connected:

```Elixir
iex(node2@jkmrto-XPS-15-9570)2> Node.list()
[]
```
This means that the nodes are not connected yet. it is needed to require to get the connection:

```Elixir
iex(node2@jkmrto-XPS-15-9570)2> Node.connect()
true
```

Now our nodes are connected, we can check it:

```Elixir
iex(node2@jkmrto-XPS-15-9570)4> Node.list()   
[:"node1@jkmrto-XPS-15-9570"]
```

Another way to check the connectivity with other node is using [:net_admin:ping()](http://erlang.org/doc/man/net_adm.html#ping-1):

```elixir
# From node1
iex(node2@jkmrto-XPS-15-9570)1> :net_adm.ping(:node1@jkmrto-XPS-15-9570)
Pong

iex(node2@jkmrto-XPS-15-9570)1> Node.list() 
[:"node1@jkmrto-XPS-15-9570"]
```

# Connecting nodes with Libcluster

One useful library when trying to get connected our nodes is [libcluster](https://github.com/bitwalker/libcluster). As his own documentation exposes:
> `This library provides a mechanism for automatically forming clusters of Erlang nodes`.

Let's build a simple application making use of this library. 
- Create the project:

```Bash
iex -S mix
```

- Add the dependency;
```Elixir
#./mix.exs
...
defp deps do
  [
    {:libcluster, "~> 3.1.1"}
  ]
end
...
```

Add the entrypoint for the application: 

```Elixir
#./mix.exs
...
def application do
  [
    extra_applications: [:logger],
    mod: {LibclusterPoc, []}
  ]
end
...
```

Let's add the module that is called at entrypoing:

```Elixir
defmodule LibclusterPoc do
  use Application

  def start(_type, _args) do
    topologies = [
      example: [
        strategy: Cluster.Strategy.Epmd,
        config: [
          hosts: [
            :"node1@jkmrto-XPS-15-9570",
            :"node2@jkmrto-XPS-15-9570",
            :"node3@jkmrto-XPS-15-9570"
          ]
        ]
      ]
    ]

    children = [
      {Cluster.Supervisor, [topologies, [name: :cluster_supervisor]]}
    ]

    Supervisor.start_link(children, strategy: :one_for_one, name: LibclusterPoc)
  end
end
```

It is important to note this:

- The strategic used by libcluster is `Cluster.Strategy.Epmd` which relies on epmd to get connected the different hosts. 

- They have been specified three nodes to get connected.

Let's launch the three instances in local and to check how the nodes are automatically connected.


```Elixir
jkmrto:libcluster_poc/ $ iex --sname node1 --cookie cookie -S mix
[libcluster:example] unable to connect to :"node2@jkmrto-XPS-15-9570"
[libcluster:example] unable to connect to :"node3@jkmrto-XPS-15-9570"
```

```Elixir
jkmrto:libcluster_poc/ $ iex --sname node2 --cookie cookie -S mix                         
[libcluster:example] connected to :"node1
[libcluster:example] unable to connect to :"node3@jkmrto-XPS-15-9570"
```

```Elixir
jkmrto:libcluster_poc/ $ iex --sname node3 --cookie cookie -S mix                         
[libcluster:example] connected to :"node1@jkmrto-XPS-15-9570"
[libcluster:example] connected to :"node2@jkmrto-XPS-15-9570"
```

### Checking the epmd

## On going

2. Autoconnect using libcluster.

 - Add docker compose with custom network and libcluster.  

 3.Docker DNS

 - Example  with the first kind of connection without using docker dns

 - Example using docker DNS
 
 - What is docker dns
 
 - Using custom network 
 
 - Docker compose ?? (autoscale with docker compose up).
