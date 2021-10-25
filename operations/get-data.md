# Getting of a Node Data

To get a binary data associated with node, use `GetDataAsync`:

```csharp
var getDataResult = await client.GetDataAsync(new GetDataRequest("/path/to/node"));
byte[]? data = getDataResult.Data;
NodeStat? stat = getDataResult.Stat;
```

If such node exists and has any data, it will be accessible through `Data` property of the result. Like `ExistsResult`, `GetDataResult` also contains `NodeStat Stat` object, which is not-null when node exists.
