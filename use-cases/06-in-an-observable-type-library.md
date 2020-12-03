# Use In An Observable/Reactive Programming Library

AbortSignal is something we truly want to use in [RxJS](/reactivex/rxjs), and something that is currently supported in [IxJS](/reactivex/ixjs). That said, in its
current state, it's mildly tricky to use as compared to what we currently have with our `Subscription` type. The `Subscription` type currently used by RxJS has
been around for many years, and is effectively an `AbortController` glued to an `AbortSignal` with some ergonomics considerations.

What we'd use `AbortController` and `AbortSignal` for in RxJS is about two things:

1. Cancellation - The act of a consumer telling a producer it no longer wants values.
2. Teardown - The act of a producer setting up code to be executed in order to clean up when it has either:
  a. completed successfully
  b. stopped because of an error
  c. been told to "cancel" by the consumer.
  
Number 2 above is probably the most important piece, as it provides guarantees around memory management.

A basic (pseudo) observable's use of AbortSignal/Controller would look something like this (note this is overly simplified and removes a 
lot of other niceties from Observable for the sake of focusing on `AbortSignal` use):

```ts
type InitFunction<T> = (next: (value: T) => void, error: (err: any) => void, complete: () => void, signal: AbortSignal) => void;

class Observable<T> {
  constructor(private init: InitFunction<T>) {}

  subscribe(
    next: (value: T) => void,
    error: (err: any) => void,
    complete: () => void,
    signal?: AbortSignal,
  ): void {
    // Create our subscription signal. 
    const subscriptionAC = new AbortSignal();
    const subscriptionSignal = subscriptionAC.signal;
    
    if (signal) {
      // SHENANIGANS
      // Attach any provided external signal to our subscription signal.
      const abortSubscription = () => {
        subscriptionAC.abort();
      }
      subscriptionSignal.addEventListener('abort', () => {
        signal.removeEventListener('abort', abortSubscription);
      }, { once: true });
      signal.addEventListener('abort', abortSubscription, { once: true })
    }
    
    // used to avoid reentrant calls.
    let stopped = false;
    
    const wrappedNext = (value: T) => {
      if (!stopped && !signal.aborted) {
        next(value);
      }
    };
    
    const wrappedError = (err: any) => {
      if (!stopped && !signal.aborted) {
        stopped = true;
        error(err);
        // teardown
        subscriptionAC.abort();
      }
    }
    
    const wrappedComplete = () => {
      if (!stopped && !signal.aborted) {
        stopped = true;
        complete();
        // teardown
        subscriptionAC.abort();
      }
    }
    
    this.init(next, error, complete, subscriptionSignal);
  }
}
```

And usage examples:

```ts
const socketSource = new Observable((next, error, complete, signal) => {
  const socket = new WebSocket('ws://some-socket-url');
  socket.onmessage = next;
  socket.onerror = error;
  socket.onclose = e => {
    if (e.wasClean) {
      error(e);
    } else {
      complete();
    }
  };
  // Setting up teardown for the socket.
  signal.addEventListener('abort', () => {
    socket.close();
  }, { once: true });
});

// or

const range = (start: number, end: number) => new Observable((next, error, complete, signal) => {
  // Note that here we're checking signal.aborted to break the loop
  // Imagine this is called with `range(0, Infinity)` and aborted after n === 2.
  for (let n = start; n <= end && !signal.aborted; n++) {
    next(n);
  }
  complete();
});
```

### Things To Notice:

1. **What happens if we pass an already aborted `AbortSignal` to the subscription to `socketSource`?** We can never clean up that socket! 
2. **Repetative calls to `addEventListener('abort', FN, { once: true })`.** It's just really unergonomic. `{ once: true }` and `"abort"` are _required_ arguments. Without `{ once: true }`, AFAICT the handler would leak.
3. **Chaining signals is difficult.** The section with the comment `// SHENANIGANS` above. We need to:
  - Add an event listener to the parent signal that aborts the child controller.
  - Keep a handle to the listener so that if the child signal is aborted, we can remove it from the parent
  - Add an event listener to the child signal that removes afformentioned listener from the parent
  - Make sure `{ once: true }` is on everything.

These things are handled in RxJS cleanly in a few ways:

1. RxJS avoids this with `Subscription` currently because if a `Subscription` is already "`closed`" (aborted), and you `add()` handler to it, it will immediately execute the handler. This prevents people from creating memory leaks with a non-obvious footgun.
2. RxJS `Subscription` has a method called `add()` that accepts a 
3. The `add()` method in RxJS's `Subscription` will automatically wire up the parent/child relationship such that if a child subscription is unsubscribed, it will remove itself from the parent.
