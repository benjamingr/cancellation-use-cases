## Use Case: High Throughput TCP Server

Chén is building a fast TCP server that needs to deal with and vet very large amounts of data from clients.

The server can go down and requests may be aborted. If requests are aborted the destruction/cancellation is dealt with by the stream. If the server goes down ongoing requests get disconnected:

```js
// Accepts incoming connections and immediatley pipe them to an output stream
net.createServer((socket) => {
    socket.pipe(outputStream)
});
```

Chén uses more modern JavaScript features like promises and async iterators but not in the "hot" parts of the code.

Chén is not sure if they should even use controllers or signals since this use case is very performance sensitive. Chén runs node with the `--abort-on-uncaught-exceptions` flag and has no `uncaughtException` handler at the process level.

They do forward uncaught exceptions to an error monitoring tool.