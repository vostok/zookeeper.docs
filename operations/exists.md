# Probing Node Existence

Existence of node can be checked with `ExistsRequest`:

```csharp
var existsResult = await client.ExistsAsync(new ExistsRequest("/path/to/probed/node"));
```

During Exists request a watcher can be set onto a probed node.

The result of this operation consists of `bool Exists` and `NodeStat Stat` with the latter one being not-null only when node exists.
