# Distributed Lock

First and foremost, be aware that given the non-realtime nature of .net and properties of distributed lock, such lock can not be considered as 'firm' lock and can only be used to perform optimizations when no strict guarantees are needed.

To create a `DistributedLock` one must provide an instance of `IZooKeeperClient`, `DistributedLockSettings` and an optional `ILog`.

```csharp
var settings = new DistributedLockSettings("path/to/lock/node");
var distributedLock = new DistributedLock(client, settings, log);
```

In this model, a single node represents a single locked resource.

#### Acquiring a Lock

`DistributedLock` provides two similar methods: `TryAcquireAsync` and `AcquireAsync`.

`TryAcquireAsync(TimeSpan timeout, CancellationToken cancellationToken = default)` would try to acquire lock within given timeout and return `null` if it fails to do so.

`AcquireAsync(CancellationToken cancellationToken = default)` would try to acquire lock indefinitely.

Both would throw `OperationCanceledException` if provided `CancellationToken` is canceled before lock is acquired.

#### Checking and Releasing a Lock: IDistributedLockToken

Acquired lock is represented by a disposable `IDistributedLockToken`. Its properties `IsAcquired` and `CancellationToken` indicate whether the lock is still being held; its values can change at any time. `Dispose` releases the lock and triggers the `CancellationToken`, although it is not guaranteed that lock will be released immediately. The exact moment of release depends upon the state of connection to ZooKeeper cluster.

#### Correct Usage of a Lock

Given the uncertain nature of DistributedLockToken, one could ask as to how to correctly use a distributed lock recipe. The correct usage pattern should look like this:

```csharp
var @lock = new DistributedLock(client, settings, log);

using (var token = await @lock.TryAcquireAsync(TimeSpan.FromSeconds(5)))
{
    if (token == null) return;

    while (true)
    {       
        if (!token.IsAcquired) break;

        if (DoPartOfWorkSync(token.CancellationToken))
            return;
    }
}
```

Taking of the lock creates a sequential ephemeral node in ZooKeeper. Such node will persist until it is deleted or client session is expired.

After we've checked that lock is still being held, we should normally have no less than a `client.SessionTimeout` time to perform our actions before the next check is needed. `DoPartOfWorkSync` should complete its work not much later than cancellation is requested.

However, if client loses connection and shortly reestablishes it, client should try to delete created node. Hence there is no guarantee that lock will persist for at least `client.SessionTimeout`.
