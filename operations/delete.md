# Deleting a Node

To delete a node, call `DeleteAsync`. By default, any children of a given node will be deleted too. Override this behaviour by setting `DeleteRequest.DeleteChildrenIfNeeded`.

Specify `DeleteRequest.Version` to only delete a node with expected version.

```csharp
var result = await client.DeleteAsync(new DeleteRequest("/node/path") {DeleteChildrenIfNeeded = false, Version = 73});
if (result.Status == ZooKeeperStatus.NodeHasChildren)
    Console.WriteLine("Not a leaf.");
else if (result.Status == ZooKeeperStatus.VersionsMismatch)
    Console.WriteLine("Node had an unexpected version and therefore was not deleted.");
```

Note that deletion of a node that has children with its children is not atomic. Nodes will be deleted recursively, starting with leaves, until it is possible to delete an original node. This means that an inconsistent state may be read before deletion is completed or an operation may be left halfway done in case of a connection-related issues.
