## Use Case: Service in a large code base

Frank is writing a JavaScript service that talks to a database and needs to support cancellation inside a large code base.

They need other services to consume the service.

If a request is stale, the service may need to cancel requests on its own - Frank is not sure how to accomplish this since
they have an abort signal which you can't `abort` on. They are also not sure what the error handling best practices are.

```js

class DataHandler {
    constructor(config, db) {
        this.config = config;
        this.db = db;
    }

    async handleData(data, { signal } = {}) {
        const result = await this.db.handle(this.process(data), { signal });
        // ...
        return result;
    }
}
```
