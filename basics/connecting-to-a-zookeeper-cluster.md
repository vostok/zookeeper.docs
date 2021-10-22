# Connecting to a ZooKeeper Cluster

State of the current connection (do not confuse it with session!) is described by `ConnectionState` property. `ConnectionState` can be one of these: `Connected`, `ConnectedReadonly`, `Disconnected`, `Expired`, `Died`, and `AuthFailed`.

One can subscribe to the changes of `ConnectionState` using `IObservable OnConnectionStateChanged`. When subscribed to, this sequence immediately produces event with current `ConnectionState`. It _may_ produce an `IObserver<T>.OnCompleted` notification when client gets disposed, and _never_ produces `IObserver<T>.OnError` notifications.

Initially, client is created in `Disconnected` state. It will try to establish a working connection upon the first request. To establish a connection immediately, use `ConnectAsync()`/`Connect()` method.

Once successfully connected, `ConnectionState` will become either `Connected` or `ConnectedReadonly`. Then, if for some reason client loses connection, `ConnectionState` once again becomes `Disconnected`. `ConnectionState` becomes `Dead` only when client instance is disposed and therefore cannot be used anymore. Client is always connected to a single server node. If that node falls out of quorum, client would try to connect to another node unless `CanBeReadOnly` was set to `true`. In the latter case, state will become `ConnectedReadonly`.

`ConnectionState.Expired` is emitted by server-side when client tries to reconnect to a timed-out session and is not strictly guaranteed to appear every time when session is lost. Given that, a correct way to ensure that ephemeral nodes still exist would be to subscribe onto `ConnectionState` changes and manually probe nodes with `ExistsAsync` every time that `ConnectionState` event is emitted.

`SessionId` property returns an `int` id of an established session when client is in connected state and `0` otherwise. `SessionTimeout` returns negotiated session timeout which may differ from `Timeout` set in `ZooKeeperClientSettings` upon client creation: ZooKeeper cluster may have configured `minSessionTimeout` and `maxSessionTimeout` that limit the possible range of timeout values.
