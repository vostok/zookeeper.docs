# Overview

Vostok.ZooKeeper provides a set of libraries for convenient interaction with open-source ZooKeeper system from within .NET applications.

There are multiple modules including the client itself, specific recipes and testing helpers:

* `Vostok.ZooKeeper.Client.Abstractions` contains all the necessary interfaces and models
* `Vostok.ZooKeeper.Client` contains implementation of the client that supports most of ZooKeeper's functionality
* `Vostok.ZooKeeper.Recipes` contains recipes that help to employ ZooKeeper for more advanced scenarios such as [distributed locks](https://zookeeper.apache.org/doc/current/recipes.html#sc\_recipes\_Locks)
* `Vostok.ZooKeeper.Testing` provides helpers for testing applications that are using ZooKeeper Client
* `Vostok.ZooKeeper.LocalEnsemble` is also usable in testing and provides an easy way to set up a local ZooKeeper cluster (ensemble)

### Client Features

Client implementation provides a certain level of abstraction from a low-level ZooKeeper protocol. In particular, it supports:

* Auto-reconnect on connection loss
* Watchers
* Observable connection changes
* Recursive node creation and deletion
