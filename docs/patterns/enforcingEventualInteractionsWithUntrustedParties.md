# Enforcing eventual interactions with untrusted parties

## Intent
Preventing reentrancy attacks.

## Consequences
-   All method calls that are performed with the eventual send will be executed after the immediate calls [1].

## Context
As previously described, Agoric makes use of vats. A vat is
*a unit of synchrony*, implemented as an event loop with a queue of messages, a stack of frames and a heap of objects [2]. The event loop of the vat follows the same concurrency model as a web browser [2]: objects are stored in the heap, synchronous calls are added as a frame to the top of the call stack and asynchronous calls are added as a message to the end of the queue [3]. The frames in the stack are processed from top to bottom. When the stack is empty, the messages in the queue are processed from left to right [1]. On an immediate call, a frame is created and pushed to the top of the stack. Agoric introduces the notion of an eventual send. On an eventual send, a message is created and added to the end of the queue. The call to the method involved in the eventual send is performed after the stack has been cleared, and thus after all immediate calls have been performed. The call to this method is thus sent eventually. Hence the name
"eventual send".

Eventual send is implemented in Agoric using the `E` function [4]. Thus, if an entity `e` wants to call a method `m` on an object `o`, which might cause a reentrancy attack, then entity `e` should call `E(o).m`. This ensures entity `e` that the call to method `m` on object `o` will be done after all immediate calls have been performed. Since all immediate calls have already been completed, there is no way for the reentrancy attack to succeed: the untrusted method call `o.m` can no longer call into the original contract and bring it into an inconsistent state, since the original contract has already completely executed by the time that the untrusted method `o.m` will be called [5].

## Example
Note: the following code example and the corresponding explanation is based on an example provided by Brian Warner of Agoric [5].

```js
let balance;

function buyPainting() {
    if(balance > 100) {
        customer.deliver(painting);
    }
    this.balance -= 100;
}
```

The `buyPainting` function in the code above states the following: if the balance is greater than 100, then the painting is delivered to the customer. After that, 100 is subtracted from the balance. The issue is that the `customer` object and its implementation of the `deliver` method can't be trusted. It might be that the `deliver` method of the `customer` object accepts the painting and throw an exception, avoiding that 100 is charged. It could also be possible that the implementation of the deliver method calls the `buyPainting` function again, bringing the balance is an inconsistent state.

```js
let balance;

function buyPainting() {
    if(balance > 100) {
        E(customer).deliver(painting);
    }
    this.balance -= 100;
}
```

The `buyPainting` function in the code above has the following flow of execution: if the balance is greater than 100, the painting is eventually delivered to the customer. Then, after 100 is subtracted from the balance (and thus, the stack is empty), the painting will be delivered to the customer.

It no longer matters how the `customer` object has implemented the
`deliver` method. If the `customer` object has implemented the `deliver` method to throw an exception, then the exception will be thrown after the balance has been subtracted. If the `customer` object implemented the `deliver` method to call the `buyPainting` function, the `buyPainting` function will be called after the balance has been subtracted. Because of the usage of `E`, the delivery of the painting will always occur when the stack is empty, and thus after the balance has been subtracted.

```js
let balance;

async function buyPainting() {
    if(balance > 100) {
        await E(customer).deliver(painting);
    }
    this.balance -= 100;
}
```

The `buyPainting` function in code above has the same flow of execution as the `buyPainting` function in the initial piece of code, because the result of the eventual send is awaited. This example goes to show that using `E` does not automatically mitigate reentrancy hazards.

Note that by using the E function, the smart contract developer leaves all interactions for last, after all the immediate calls on the stack have been handled. This pattern is thus related to the \"checks effects interactions\" pattern in Solidity, which also suggests that the smart contract developer should leave all interactions for last [6].

## General rule
Given an object `o` with corresponding method `m`, the general rules are as follows.
- If object `o` is a remote presence, then `E(o).m` should be called instead of `o.m`.

- If object `o` is a promise of a remote presence, then `E(o).m` should be called instead of `o.m`.

- If object `o` cannot be trusted or method `m` can potentially cause a reentrancy hazard, then `E(o).m` should be called instead of `o.m`.

Note that the `E` function can also be used on local objects [7].

## Known uses
- The `E` function is always needed to start an instance of a smart contract.

## References
[1] M. S. Miller, D. Tribble, and J. Shapiro, “Concurrency among strangers: Programming in e as plan coordination,” Thesis, 2005.

[2] Agoric, [Javascript framework for secure distributed computing](https://docs.agoric.com/guides/js-programming/)

[3] M. W. Docs, [The event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)

[4] Agoric, [Glossary](https://docs.agoric.com/glossary)

[5] Agoric, [How agoric solves reentrancy hazards and other platform features](https://www.youtube.com/watch?v=38oTyVv_D9I)

[6] F. Volland, [Checks effects interactions](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html)

[7] Agoric, [Eventual send with e()](https://docs.agoric.com/guides/js-programming/eventual-send.html)