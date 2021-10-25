# Getting Children of a Node

To get a list of _immediate_ children of a node, use `GetChildrenAsync`.

Given the nodes

```csharp
var result = await client.GetChildrenAync(new GetChildrenRequest("/prefix"));
```

Given the nodes

```
/prefix/a
/prefix/a/1
/prefix/a/2
/prefix/b/1
/prefix/c
```

The `result.ChildrenNames` would contain

```
a
b
c
```

Result model also contains `NodeStat Stat`.
