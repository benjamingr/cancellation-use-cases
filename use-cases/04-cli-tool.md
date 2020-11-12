## Use Case: CLI Tool

María is writing a CLI tool in Node that is used in Frontend tooling.

The CLI tool is running a relatively expensive process on a code base (bundling/linting/transpiling etc) that may get interrupted because:
 - The user might exit the process with SIGINT
 - The code might need to stop running early because it ran into an error

```js

const processor = require('./processor')

const controller = new AbortController();
// heh
processor.process(process.argv, { signal }).catch((e) => {
    printFriendlyErrorToUser(e);
});
```