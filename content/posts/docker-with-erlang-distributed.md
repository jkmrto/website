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

## Roadmap to blog

1. Connect docker using just Elixir docker. 
 - Test connectivity with :net_admin.ping
2. Autoconnect using libcluster.
 - Explain what libcluster is, with link. 
 - Example with the first kind of connection withouut using docker dns
 - Example using docker DNS
  - What is docker dns
  - Using custom network 
 - Docker compose ?? (autoscale with docker compose up).




