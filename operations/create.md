# Creating a Node

Use `CreateAsync/Create` methods to create a node. 
If such node already exists, request will fail with status `NodeAlreadyExists`. There is no way to change this behaviour if one would want to ensure that node has specific data.

A node can be of one of the following types: 
- Persistent 
- Ephemeral
- PersistentSequential
- EphemeralSequential

## Persistent vs Ephemeral Nodes

Persistent nodes exist until deleted by a deliberate request, whilst ephemeral are sustained as long as client session is alive. In more detail the difference between persistent and ephemeral nodes was discussed in overview.

## Sequential Nodes

When sequential node is being created, a monotonically increasing number is being added to its path. So, the real path of created node differs from one specified in the request and is stored in `NewPath` property of the result: 
```
var createResult = await client.CreateAsync("/parent/seq-child", CreateMode.EphemeralSequential);
createResult.EnsureSuccess();

Console.WriteLine(createResult.NewPath);

var createResult2 = await client.CreateAsync("/parent/seq-child", CreateMode.EphemeralSequential);
createResult2.EnsureSuccess();

Console.WriteLine(createResult2.NewPath);

// Prints:
// /parent/seq-child0000000000
// /parent/seq-child0000000001
```

Creation of a sequential node is handled by ZooKeeper itself and is guaranteed to be atomic. 
The counting is being done on the parent node and the number increases every time a node of any type (not just sequential) is created.

## Setting Data to a Node

To set an arbitrary bytes as node's data, specify them in request: 
```
var createResult = await client.CreateAsync(new CreateRequest("/path/to/node")
{
    Data = new byte[1024*1023]
);
```

Maximum allowed data size is 1024*1023 bytes per node. `ArgumentException` will be thrown if this size is exceeded.

## Creating Parent Nodes

By default, client would automatically the whole subtree if some parent nodes do not exist.
To override this behaviour, set `CreateParentsIfNeeded` to `false`:

```
client.Create(new CreateRequest("/parent-does-not-exist/child", CreateMode.Ephemeral) { CreateParentsIfNeeded = false }).EnsureSuccess();
// Vostok.ZooKeeper.Client.Abstractions.Model.ZooKeeperException: ZooKeeper operation has failed with status 'NodeNotFound' for path '/parent-does-not-exist/child'.
```

Since only a persistent node can have children, all automatically created parent nodes would be persistent. 
