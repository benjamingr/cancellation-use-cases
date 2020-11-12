## Use Case: Server and Request Cancellation

Marcus is writing code using an HTTP server. Marcus needs to address cancellation coming from either the request itself (by the user aborting the request) or from the server itself needing to go down (a SIGINT handler).

In addition, Marcus uses an error instrumentation tool and needs to filter our cancellation from their tool:

```js
const express = require('express');
const rootController = new AbortController();

app = express();
app.use(cancellationMiddleware(rootController)); // create a linked req.controller

app.get('/', (req, res) => {
   doSomethingAsync(req.params.foo, { signal }).then((data) => {
       res.json(data);
   });
});

errorMiddleware(app, (err, res) => {
    if (!isCancellation(err)) {
        monitoring.track(err);
    }
    respondToError(err, res);
});

```