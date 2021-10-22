# Quickstart

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
using Vostok.Logging.Abstractions;
using Vostok.ZooKeeper.Abstractions;
using Vostok.ZooKeeper.Client;

var log = new SynchronousConsoleLog();
var settings = new ZooKeeperClientSettings(connectionString: "10.217.10.42:2181,10.217.20.73:2181");
using var client = new ZooKeeperClient(zookeeperSettings, log);
```

Continue to explore Vostok.ZooKeeper by learning some basic concepts...

{% content-ref url="basics/" %}
[basics](basics/)
{% endcontent-ref %}
