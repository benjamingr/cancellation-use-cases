## Questions regarding cancellation

 - How do users identify a cancellation error as cancellation and not an operational error in their instrumentation? 
 - Do different types of cancellation (e.g. HTTP request vs. SIGINT) have distinct identifiers (codes)?


## How different languages answer each of these questions

Please help by filling the below in

### C#

 - Identify: Users identify Cancellation using the [`OperationCanceledException`](https://docs.microsoft.com/en-us/dotnet/api/system.operationcanceledexception?view=net-5.0) class. Different APIs subclass it e.g. [HttpClient.GetAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient.getasync?view=net-5.0#System_Net_Http_HttpClient_GetAsync_System_Uri_System_Threading_CancellationToken_) uses a `TaskCanceledException`.
 - Do different types of cancellation: Not in the ecosystem:
   - Most userland libraries I checked do not subclass and return `TaskCanceledException` for example:
     - [MongoDB](https://github.com/mongodb/mongo-csharp-driver/blob/010e7ee46b085cdd3762894ece9e2d258b66ab0d/tests/MongoDB.Driver.Core.Tests/Core/Async/AsyncQueueTests.cs#L96)
     - [Redis](https://github.com/StackExchange/StackExchange.Redis/blob/b0f21d3616c795ad994dd4151a5a556e9d1558b3/tests/StackExchange.Redis.Tests/AggresssiveTests.cs#L50)
### Python

### Java

### Go

### Rust

### Haskell

### C++