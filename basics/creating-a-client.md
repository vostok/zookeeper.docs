# Creating a Client

Client's constructor requires an object of type `ZooKeeperClientSettings`. The only non-optional setting there is a ZooKeeper's cluster topology which can be in form of either a connection string or a list of replicas. Connection string must consist of pairs host:port separated only by `,`.

```
Vostok.Logging.Abstractions.ILog log = ...;
var settings = new ZooKeeperClientSettings(connectionString: "10.217.10.42:2181,10.217.20.73:2181")
{
    ...
};
using var client = new ZooKeeperClient(settings, log);
```

Note that client implements `IDisposable` interface and therefore should be disposed once not needed anymore to correctly release all the resources.

ZooKeeperClient accepts two types of topologies: static (either `string` or `IList<Uri>`) and dynamic (`Func<string>` or `Func<IList<Uri>>`). It is expected that static topology does not change over time. So, if the client is created with, say, replicas A, B, and C, then there is no way to inform it on a new replica D.

Contrarily to a static topology, dynamic one does allow this to happen. Though be aware of its implementation-driven limitations: old and new topologies are compared only by reference. So, client that observes a new dynamic topology will drop existing connection and therefore lose all session's ephemeral nodes regardless of topology composition.

Now let's take a look at additional settings.

|                             |                                                                                                                                                                                                                                                                                           |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TimeSpan Timeout`          | Specifies connection and session timeout. Note that the real session timeout is decided by ZooKeeper's server and may differ from one set in parameters. [See the section on client-server connection for details. ](connecting-to-a-zookeeper-cluster.md)                                |
| `bool CanBeReadOnly`        | If set to `true`, allows client to operate in read-only mode during partitions that isolate the server node it's connected to from established quorum.                                                                                                                                    |
| `int WatchersCacheCapacity` | Limits unique watchers that can be used with ZooKeeperClient instance. After reaching this limit, same watcher can be triggered multiple times, if it was added multiple times on same node.[ See the section on watchers for details.](../observing-events.md#deduplication-of-watchers) |

&#x20;&#x20;
