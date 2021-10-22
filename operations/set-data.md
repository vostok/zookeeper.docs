# Setting of a Node Data

It is possible to associate some binary data with a node using `SetDataAsync`.
Its request model accepts an array of bytes of size no more than 1024*1023. If that size is exceeded, `ArgumentException` will be thrown upon request.

Every setting of node data increments a version of that node which can be found in `NodeStat.Version`. Deletion of a node resets its version.

In `SetDataRequest` one can specify an expected version of node. If an actual version differs from expected, request will fail with `VersionsMismatch` and node will remain unchanged.

If node does not exist, request will fail with `NodeNotFound` status.

```
client.SetData(new ("/node/with/data", new byte[1]) { Version = 2 })
```