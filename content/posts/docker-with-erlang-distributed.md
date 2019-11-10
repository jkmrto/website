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

Last days I have been diving a little into the wonderful world of Erlang distributed. Elixir, has some built-in constructs for distributed systems which make it easier to distribute systems in comparison with other programming paradigms.

Since Erlang is based on the actor model (where each actor is a process) it is transparent to erlang if the actor is local or if it is in a remote host, as long as the nodes are connected.

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
#Node2
iex(node2@jkmrto-XPS-15-9570)2> Node.list()
[]
```
This means that the nodes are not connected yet. it is needed to require to get the connection:

```Elixir
Node2
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

## Libcluster POC

Let's build a simple proof of concept with libcluster to run some test over it.

- Create the project:


```Bash
mix new libcluster_poc
```

Let's add the dependency on the mix file, remember to check the latest version at [hex](https://hex.pm/packages/libcluster)

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

Let's add the application entrypoint:

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
# ./lib/libcluster_poc.ex
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

## Executing three instances locally

Let's launch the three instances in local and to check how the nodes are automatically connected.


```Elixir
# First node
jkmrto:libcluster_poc/ $ iex --sname node1 --cookie cookie -S mix
[libcluster:example] unable to connect to :"node2@jkmrto-XPS-15-9570"
[libcluster:example] unable to connect to :"node3@jkmrto-XPS-15-9570"
```

```Elixir
# Second node
jkmrto:libcluster_poc/ $ iex --sname node2 --cookie cookie -S mix                         
[libcluster:example] connected to :"node1
[libcluster:example] unable to connect to :"node3@jkmrto-XPS-15-9570"
```

```Elixir
# Third node
jkmrto:libcluster_poc/ $ iex --sname node3 --cookie cookie -S mix                         
[libcluster:example] connected to :"node1@jkmrto-XPS-15-9570"
[libcluster:example] connected to :"node2@jkmrto-XPS-15-9570"
> Node.list()
[:"node1@jkmrto-XPS-15-9570", :"node2@jkmrto-XPS-15-9570"]
```

Amazing! We can see how after running the third node the logs of this node indicates us that they have been correctly connected. So in the end, all we have to care about to get connected the nodes is to indicates the list of nodes when starting the application.


## Checking the epmd

One interesting system of the Erlang ecosystem is the Erlang Port Mapper Daemon, also know as [epmd](http://erlang.org/doc/man/epmd.html). This service is in charge to map each erlang node to the port where it is listening for connections.

A epmd server is started as soon as one erlang node starts. If another erlang node is launched in the same node then the relation `{port, node_name}` will be added to the already running epmd.


We can check the erlang nodes running in our machine using the epmd service. The epmd executable is located in the same folder that `erl` binary.
```
> which erl
/home/jkmrto/.asdf/shims/erl
> cd /home/jkmrto/.asdf/shims
> epmd -names
epmd: up and running on port 4369 with data:
name node3 at port 39099
name node2 at port 44643
name node1 at port 45015
```


## Executing three instances dockerized

### Dockerizing the poc

Let's use this simple dockerfile:

``` Docker
# ./dockerfile
FROM elixir:1.9.4-slim

RUN mix local.hex --force
RUN mix local.rebar --force

RUN mkdir -p /opt/libcluster_poc
ADD ./config /opt/libcluster_poc/config
ADD ./lib /opt/libcluster_poc/lib
ADD ./mix.exs /opt/libcluster_poc


WORKDIR /opt/libcluster_poc  
RUN mix deps.get && mix compile

ENTRYPOINT [ "iex", "-S",  "mix" ]
```

Let's build and try it:

```
> docker build -t libcluster_poc .
```

And running it:

``` Elixir
> docker run -it  libcluster_poc

Erlang/OTP 22 [erts-10.5.4] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [hipe]

17:44:20.079 [warn]  [libcluster:example] unable to connect to :"node1@jkmrto-XPS-15-9570": not part of network
 
17:44:20.080 [warn]  [libcluster:example] unable to connect to :"node2@jkmrto-XPS-15-9570": not part of network

17:44:20.080 [warn]  [libcluster:example] unable to connect to :"node3@jkmrto-XPS-15-9570": not part of network
Interactive Elixir (1.9.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> 
```

### Using docker network

Once we have our docker defined, we need to start think about how to have some of them connected. Let's now use an internal docker net, avoiding using the `--net=host` trick.

Let's create the network, calling it `net_poc`:

```Elixir
docker network create net_poc 
```

### Custom docker run

Let's run our docker instances. We are going to need some options in order to get the expected behaviour.

These options are:

- `-it`: It is required to initialize an interactive session.

- `--net net_poc`: It is required to attach the docker to network previously created.

-  `--name`: This would be the name of the node used by the [docker DNS](https://docs.docker.com/v17.09/engine/userguide/networking/configure-dns/) to resolve the docker location inside our custom network. It is also the name given to the docker that allows to identify the container when executing `docker ps`.

- `--hostname`: This will be the hostname of the docker launched. It is used by erlang to fully qualified the erlang node. 

-  `--entrypoint=iex libcluster_poc --cookie cookie --sname node1 -S mix`: This syntax allows to define the docker image to be executed and a custom entrypoint, since it is needed to be different for each container in order to customize the name of the erlang node with the -sname flag.
    - `--entrypoint=iex`: Executable to be run when starting the docker.
    - `libcluster_poc`: Docker image to be executed. 
    - `--cookie cookie --sname node1 -S mix`: Entrypoint arguments.

```Elixir
# Node1
> docker run -it --net net_poc --hostname node1 --name node1 --entrypoint=iex libcluster_poc --cookie cookie --sname node1 -S mix
```

``` Elixir
# Node2
> docker run -it --net net_poc --hostname node2 --name node2 --entrypoint=iex libcluster_poc --cookie cookie --sname node2 -S mix
```


Once both of them are launched we can check that there is connectivity between both of them:

``` Elixir
# Node1
> Node.ping(:node2@node2)
pong
```

``` Elixir
# Node2
> Node.ping(:node1@node1)
pong
```

Great! We have are two docker fully connected. 

It is important to note that the nodes are not connected when starting them and they get connected when doing `Node.ping/1`. The next step is getting them connected at startup.

### Automatically connection with libcluster.

We just need to redefine the nodes at libcluster topologies to get nodes automatically connected at start.

```Elixir
#./lib/libcluster_poc.ex
...
topologies = [
      example: [
        strategy: Cluster.Strategy.Epmd,
        config: [
          hosts: [
            :node1@node1,
            :node2@node2
          ]
        ]
      ]
    ]
```

Let's rebuild the docker


```
> docker build -t libcluster_poc .
```

Let's run our dockers:

```Elixir
# Node1
> docker run -it --net net_poc --hostname node1 --name node1 --entrypoint=iex libcluster_poc --cookie cookie --sname node1 -S mix
[libcluster:example] unable to connect to :node2@node2
```

``` Elixir
# Node2
> $ docker run -it --net net_poc --hostname node2 --name node2 --entrypoint=iex libcluster_poc --cookie cookie --sname node2 -S mix
[libcluster:example] connected to :node1@node1
```

Wonderful! Now our two nodes are connected at start up thanks to libcluster. 

# On going

2. Autoconnect using libcluster.
 - Use docker dns to connect the containers

 - Add docker compose with custom network and libcluster.  

 3.Docker DNS

 - Example  with the first kind of connection without using docker dns

 - Example using docker DNS
 
 - What is docker dns
 
 - Using custom network 
 
 - Docker compose ?? (autoscale with docker compose up).
