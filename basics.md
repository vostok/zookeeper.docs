# Basics

## Limitations

This implementation does not support (as yet) batch requests, transactions and dynamic cluster reconfigurations of ZK 3.6+.


## A Short Introduction to ZooKeeper

ZooKeeper cluster consists of a tree-like structure of nodes with `/` as its root. There are two types of nodes: persistent and ephemeral.
Once created, persistent node will exist until explicitly deleted. On the other hand, lifetime of an ephemeral node is limited to that of a client-server session within which this node was created.

Only persistent nodes can have children. Any node can also contain (a small amount of) arbitrary bytes as its data.

ZooKeeper client holds a session with servers in cluster that is described by _session id_ and _session timeout_.
Client constantly queries servers to keep the session alive and receive notifications.
When backend does not hear anything from a client within the whole session timeout, it considers such session expired.


## Creating a Client

Client's constructor requires an object of type `ZooKeeperClientSettings`. The only non-optional setting there is a ZooKeeper's cluster topology which can be in form of either a connection string or a list of replicas.
Connection string must consist of pairs host:port separated only by `,`.

```
Vostok.Logging.Abstractions.ILog log = ...;
var settings = new ZooKeeperClientSettings(connectionString: "10.217.10.42:2181,10.217.20.73:2181")
{
    ...
};
using var client = new ZooKeeperClient(settings, log);
```

Note that client implements `IDisposable` interface and therefore should be disposed once not needed anymore to correctly release all the resources.

ZooKeeperClient accepts two types of topologies: static (either `string` or `IList<Uri>`) and dynamic (`Func<string>` or `Func<IList<Uri>>`).
It is expected that static topology does not change over time. So, if the client is created with, say, replicas A, B, and C, then there is no way to inform it on a new replica D.

Contrarily to a static topology, dynamic one does allow this to happen. Though be aware of its implementation-driven limitations: old and new topologies are compared only by reference. 
So, client that observes a new dynamic topology will drop existing connection and therefore lose all session's ephemeral nodes regardless of topology composition.

Now let's take a look at additional settings.

`TimeSpan Timeout` specifies connection and session timeout. Note that the real session timeout is decided by ZooKeeper's server and may differ from one set in parameters. See the next section for details.
`bool CanBeReadOnly` if set to `true`, allows client to operate in read-only mode during partitions that isolate the server node it's connected to from established quorum.
`int WatchersCacheCapacity` limits unique watchers that can be used with ZooKeeperClient instance. After reaching this limit, same watcher can be triggered multiple times, if it was added multiple times on same node. See the section on watchers for details.


## Connecting to a ZooKeeper Cluster

State of the current connection (do not confuse it with session!) is described by `ConnectionState` property.
`ConnectionState` can be one of these: `Connected`, `ConnectedReadonly`, `Disconnected`, `Expired`, `Died`, and `AuthFailed`.

One can subscribe to the changes of `ConnectionState` using `IObservable OnConnectionStateChanged`. When subscribed to, this sequence immediately produces event with current `ConnectionState`. It _may_ produce an `IObserver<T>.OnCompleted` notification when client gets disposed, and _never_ produces `IObserver<T>.OnError` notifications.

Initially, client is created in `Disconnected` state. It will try to establish a working connection upon the first request.
To establish a connection immediately, use `ConnectAsync()`/`Connect()` method.

Once successfully connected, `ConnectionState` will become either `Connected` or `ConnectedReadonly`.
Then, if for some reason client loses connection, `ConnectionState` once again becomes `Disconnected`.
`ConnectionState` becomes `Dead` only when client instance is disposed and therefore cannot be used anymore.
Client is always connected to a single server node. If that node falls out of quorum, client would try to connect to another node unless `CanBeReadOnly` was set to `true`. In the latter case, state will become `ConnectedReadonly`.

`ConnectionState.Expired` is emitted by server-side when client tries to reconnect to a timed-out session and is not strictly guaranteed to appear every time when session is lost. 
Given that, a correct way to ensure that ephemeral nodes still exist would be to subscribe onto `ConnectionState` changes and manually probe nodes with `ExistsAsync` every time that `ConnectionState` event is emitted.

`SessionId` property returns an `int` id of an established session when client is in connected state and `0` otherwise.
`SessionTimeout` returns negotiated session timeout which may differ from `Timeout` set in `ZooKeeperClientSettings` upon client creation: ZooKeeper cluster may have configured `minSessionTimeout` and `maxSessionTimeout` that limit the possible range of timeout values.


## IZooKeeperClient Overview

`IZooKeeperClient` exposes a bunch of methods for manipulations with ZooKeeper nodes along with some session-related properties.
With it, you can create and delete nodes, retrieve and change contents of nodes, probe nodes existence and get all the children of given node.
All methods of client are thread-safe. A single ZooKeeperClient instance can (and should!) be shared between multiple consumers.

Let's take a look at CreateAsync method as an example.
It accepts a request model object as its argument that has two required and some optional arguments:

```
var createRequest = new CreateRequest("/path/to/node", CreateMode.Ephemeral)
{
    Data = new byte[42],
    CreateParentsIfNeeded = true,
};
```

Any operation returns a complex result:
```
CreateResult createResult = await client.CreateAsync(createRequest);
if (createResult.IsSuccessful)
{
    Console.WriteLine($"Path from createRequest: {createResult.Path}");
    Console.WriteLine($"Actual node's path: {createResult.NewPath}");
    
    if (createResult.Path != createResult.NewPath)
        Console.WriteLine($"Seems like node was created with {CreateMode.PersistentSequential} or {CreateMode.EphemeralSequential}.");
}
else
{
    Console.WriteLine($"Request has failed with ZooKeeperStatus: {createResult.Status}.");
    Console.WriteLine($"Was there an exception? {createResult.Exception != null}");
}
```

Any result type is derived from the `ZooKeeperResult` that has a bunch of additional methods that help to analyze an outcome of operation.
For example, let's throw an exception if error is a non-retryable network failure:
```
if (!createResult.IsSuccessful && !createResult.IsRetriableNetworkError())
    createResult.EnsureSuccess();  // Throws `ZooKeeperException` if operation was not successful
```

If you do not need additional parameters for your request, a set of concise extension methods would come in handy:
```
CreateResult createResult = await client.CreateAsync("/path/to/node", CreateMode.Ephemeral, new byte[42]);
```

For any asynchronous method there's also a synchronous extension that simply does `.GetAwaiter().GetResult()`:
```
CreateResult createResult = client.Create(createRequest);
```

For detailed information on other methods please refer to a corresponding section.








