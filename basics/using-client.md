# Using Client

`IZooKeeperClient` exposes a bunch of methods for manipulations with ZooKeeper nodes along with some session-related properties. With it, you can create and delete nodes, retrieve and change contents of nodes, probe nodes existence and get all the children of given node. All methods of client are thread-safe. A single ZooKeeperClient instance can (and should!) be shared between multiple consumers.

Let's take a look at CreateAsync method as an example. It accepts a request model object as its argument that has two required and some optional arguments:

```csharp
var createRequest = new CreateRequest("/path/to/node", CreateMode.Ephemeral)
{
    Data = new byte[42],
    CreateParentsIfNeeded = true,
};
```

#### Inspecting a Result

Any operation returns a complex result:

```csharp
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

Any result type is derived from the `ZooKeeperResult` that has a bunch of additional methods that help to analyze an outcome of operation. For example, let's throw an exception if error is a non-retryable network failure:

```csharp
if (!createResult.IsSuccessful && !createResult.IsRetriableNetworkError())
    createResult.EnsureSuccess();  // Throws `ZooKeeperException` if operation was not successful
```

#### Extension Methods

If you do not need additional parameters for your request, a set of concise extension methods would come in handy:

```csharp
CreateResult createResult = await client.CreateAsync("/path/to/node", CreateMode.Ephemeral, new byte[42]);
```

For any asynchronous method there's also a synchronous extension that simply does `.GetAwaiter().GetResult()`:

```csharp
CreateResult createResult = client.Create(createRequest);
```

For detailed information on other methods please refer to a corresponding [section](../operations/).
