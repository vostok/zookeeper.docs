# Observing Events

ZooKeeper allows you to subscribe to a certain events happening to a node:
- Creation of node
- Deletion of node
- Changes of node data
- Adding and deleting of node's children

To do this, one can specify a `NodeWatcher` along with any `GetRequest`: `ExistsRequest`, `GetChildrenRequest` and `GetDataRequest`.
A watcher will be attached to a given node only if request results in success. Watcher will be triggered when a change occurs to this node *only once*.
While there is no guarantee that notification will be delivered to the client regardless of the state of connection, delivered events can never be out of order. 

```
INodeWatcher watcher = new AdHocNodeWatcher((NodeChangedEventType eventType, string changedNodePath) =>
{
    Console.WriteLine($"Something has changed with node {changedNodePath}...");
    Console.WriteLine(eventType switch
    {
        NodeChangedEventType.Created => "It was created!",
        NodeChangedEventType.Deleted => "It was deleted...",
        NodeChangedEventType.DataChanged => "Binary data stored in it has changed",
        NodeChangedEventType.ChildrenChanged => "There were children of this node added or deleted"
    });
});

client.Exists(new ExistsRequest("/path/to/watched/node") { Watcher = watcher });
```


## Deduplication of watchers

By default, an event will be delivered to a watcher only once, no matter how many times the same watcher instance was passed to client.
For example, if we attach the same watcher two times... 
```
client.Exists(new ExistsRequest("/path/to/watched/node") { Watcher = watcher });
client.Exists(new ExistsRequest("/path/to/watched/node") { Watcher = watcher });

client.Create("/path/to/watched/node", CreateMode.Ephemeral);
// 'It was created!' is printed once
```
...it will be triggered exactly once.

To override this behaviour, use `IgnoreWatchersCache = true`:
```
client.Exists(new ExistsRequest("/path/to/watched/node") { Watcher = watcher, IgnoreWatchersCache = true });
client.Exists(new ExistsRequest("/path/to/watched/node") { Watcher = watcher, IgnoreWatchersCache = true });

client.Create("/path/to/watched/node", CreateMode.Ephemeral);
// 'It was created!'
// 'It was created!' 
```

Notice that the total count of deduplicated watchers is limited by a `WatchersCacheCapacity` in `ZooKeeperClientSettings`.

## Reattaching a Watcher

Any watcher is notified only of a single event. So, once it gets called, it may be necessary to attach a new watcher. 
```
INodeWatcher watcher = null;
INodeWatcher GetWatcher() => watcher ??= new AdHocNodeWatcher((t, p) =>
{
    ...
    Task.Run(() => AttachWatcher());
});

void AttachWatcher() => client.Exists(new ExistsRequest("/path/to/watched/node") { Watcher = GetWatcher() });

AttachWatcher();
```

**One should never** call methods of client inside a watcher body as it would inevitably lead to a deadlock. In the example above, we're executing the new request in the another thread to avoid this.