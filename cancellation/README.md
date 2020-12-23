## Linked Abort Controller Explainer

TODO

## Short overview of the issue

TODO

## Use Cases the issue manifests in

There are several relatively common use cases where linked cancellation is beneficial and present:

 - [Server and Request Cancellation](https://github.com/benjamingr/cancellation-use-cases/blob/main/use-cases/01-server-process-request.md)
 - [Authoring infrastructure libraries using AbortSignal/AbortController](https://github.com/benjamingr/cancellation-use-cases/blob/main/use-cases/06-in-an-observable-type-library.md)
 - [Adding a service in a large app](https://github.com/benjamingr/cancellation-use-cases/blob/main/use-cases/05-service-in-a-large.md) already utilizing AbortController.
 
## What other ecosystems do about it

TODO

## What people do today? What are the drawbacks?

TODO

## Alternative ideas of how to solve the issue

TODO

### `new AbortController(...signals)`

TODO

### `new AbortSignal(...signals)`

TODO

### `AbortSignal.race(signal, otherSignal)`

TODO

### `signal.follow(otherSignal)`

TODO

## What about adding capabilities that make this possible in userland?

TODO

## Process, how would this work? 

TODO
