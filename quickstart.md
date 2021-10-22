# Quickstart

There are multiple 
* Install abstractions and client modules:
```
Install-Package Vostok.ZooKeeper.Client.Abstractions 
Install-Package Vostok.ZooKeeper.Client
```

* Install recipes, testing and local ensemble if needed:
```
Install-Package Vostok.ZooKeeper.Recipes
Install-Package Vostok.ZooKeeper.Testing
Install-Package Vostok.ZooKeeper.LocalEnsemble
```

* Create ZooKeeper client with the constructor

```
using var client = new ZooKeeperClient(zookeeperSettings, log);
```
