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

- Identify: Cancellation can be identified by providing [`context.Context`](https://golang.org/pkg/context/) to
a library (this is kind of standard, however, some libraries doesn't support context as argument and instead will get a channel that may be closed to signal cancellation). When cancelling a context, user can check if the `ctx.Err()` equals `context.Cancelled` or `context.DeadlineExceeded`.
- By default, SIGINT (or any other terminating signal) won't cancel running contexts, instead the user should 
catch a signal in a dedicated goroutine and cancel the root context (user is responsible to create root context and pass child contexts). Usually there is no reason to what caused the cancel

### Rust
 - Identify: Cancellation cannot be easily identified and is not exceptional. In Rust, cancellation is facilitated by [`drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html)ping the future - users do not cancel explicitly and RAII is used to collect resources. There is currently _no way_ to do asynchronous cleanup in rust ("async drop") and cancellation has to be fully synchronous. There are no exceptions and cancellation is not an error - it's like deallocating the async function.
   - This is a problem if you need to do asynchronous cleanup when cancelling. This is not a property of RAII but a property of Rust (C++ code for example which also faciliates RAII uses [cancellation_token](https://github.com/lewissbaker/cppcoro#Cancellation))
 
### Haskell

### C++

 - Identify: Like Rust, C++ async code uses RAII primarily for resource management and cancellation. C++ is also considering a [`co_using`](https://github.com/lewissbaker/papers/issues/4) concept for async RAII. C++ uses [cancelation_token](https://github.com/lewissbaker/cppcoro#Cancellation)s for explicit marking of cancelation (rather than just destructing like rust). The exception thrown is [`operation_cancelled`](https://github.com/lewissbaker/cppcoro#Cancellation). These constructs are not widely adopted in C++ land. 
 - Different popular APIs don't actually support coroutines/async in C++ and libraries I've checked did not use `cancelation_token` or `operation_cancelled` in them.

### Kotlin

 - Identify: Users identify Cancellation using the ['CancellationException'](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)
  - This is part of the coroutine itself (coroutines and not actions inside them are cancelled, cancellation is cooperative by checking `isActive`). 
  - Do different types of cancellation: There is the general cancellation and the inherit exception of ['TimeoutCancellationException'](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-timeout-cancellation-exception/index.html)
