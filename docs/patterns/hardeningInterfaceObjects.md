# Hardening interface objects

## Intent
Entities can only interact with the hardened object by using the methods that the hardened object provides [1].

## Consequences
-   The properties of the hardened object become read-only [1].
-   An entity cannot bring the smart contract into an inconsistent state by directly altering the properties of the smart contract [1].

## Context
Agoric describes a hardened object is an object of which the properties can't be changed [2]. To harden an object, the `harden` function is used. Object `o` can be hardened using the `harden` function by calling `harden(o)`. The `harden` function recursively calls JavaScript's `Object.freeze` function on the properties of the object that is hardened [3]. Agoric suggests that the `harden` function should be called on all objects that will be transferred across a trust boundary [1] [4]. The strictest way an entity could define a trust boundary, is by assuming itself as trusted, and all other entities as untrusted.

## Example
``` {.JavaScript}
//Alice creates an object which holds her contact informatoin
const aliceContactObject = {
    name:"Alice",
    walletAddress:"12345-wallet-address-Alice-12345"
};

//Alice sends this object to Bob: the object passes the trust boundary

//Bob replaces Alice's address with his own address
aliceContactObject.walletAddress = "54321-wallet-address-Bob-54321";

//The walletAddress of aliceContactObject is no longer Alice's wallet address
t.deepEqual(aliceContactObject.walletAddress,"54321-wallet-address-Bob-54321");
```

The code above shows an example of what happens when `harden` is not called on an object that passes the trust boundary. Alice creates an object holding her contact information. She passes this object to Bob without using the `harden` function first. Bob is now able to change the `walletAddressAlice` property of the `aliceContactObject` object to his own wallet address. This means that, at this point in time, there is an object circulating which Alice expects to hold her wallet address, while it actually holds Bob's address.

``` {.JavaScript}
//Alice creates a hardened object which holds her contact informatoin
const aliceContactObject = harden({
    name:"Alice",
    walletAddress:"12345-wallet-address-Alice-12345"
});

//Alice sends this object to Bob: the object passes the trust boundary

//If Bob tries to replace Alice's address with his own address, an error is thrown
const error = t.throws(() => {
	aliceContactObject.walletAddress = "54321-wallet-address-Bob-54321";
}, {instanceOf: TypeError});

//Bob is not able to change Alice's wallet address without getting an error
t.deepEqual(
    error?.message,
    "Cannot assign to read only property 'walletAddress' of object '[object Object]'"
);
```

The code above shows the usage of `harden` can avoid the problem in the initial piece of code. In this example, Alice first uses the `harden` function on the `aliceContactObject` object before sending it to Bob. Bob then attempts to change Alice's `walletAddress` field in the `aliceContactObject` object to his own wallet address. This, however, throws a `typeError` with a message stating that the `walletAddress` property is a read-only property. It is thus impossible for Bob to change the `walletAddress` property without triggering a `typeError` from Agoric. 

## General rule
If entity `e` creates an object `o` and entity `e` passes object `o` across the trust boundary, then entity `e` should pass `harden(o)` instead of `o` [1].

## Known uses
-   Every smart contract uses the `harden` function on the `start`
    method before exporting the `start` method.

## References
[1] Endo, [Endo and hardened javascript (ses) programming guide](https://github.com/endojs/endo/blob/HEAD/packages/ses/docs/guide.md)

[2] Agoric, [Glossary](https://docs.agoric.com/glossary)

[3] M. S. Miller and K. Sills, [Why not to use object.freeze for immutable js objects?](https://ocapjs.org/t/why-not-to-use-object-freeze-for-immutable-js-objects/91)

[4] Agoric, [Hardened javascript](https://docs.agoric.com/guides/js-programming/hardened-js.html)