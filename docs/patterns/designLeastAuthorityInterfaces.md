# Design least authority interfaces

## Intent
Provide participants with the minimal amount of authority to perform their
actions, and nothing more. Thus: avoid excessive authority.

## Consequences
-   Avoiding giving excessive rights to an entity. If an entity has excessive rights, then the entity might be able to bring the smart contract into an inconsistent state.

## Context
Agoric makes use of object capabilities, which means that references can only be obtained via creation, construction and
introduction [1]. A smart contract can return a `creatorInvitation`, a `creatorFacet` and a `publicFacet` [2]. Thus, Zoe introduces these objects to the entity that starts an instance of the smart contract (referred to as the creator). The creator then introduces the `publicFacet` to other entities that might be interested. Since the `publicFacet` becomes available to entities that cannot be trusted, it is important that the methods on the `publicFacet` only return the bare minimum needed by these untrusted entities, so that they can participate in the smart contract. A smart contract introducing an object to an untrusted entity should be seen as the smart contract giving the untrusted entity the permission to invoke any method on that object [3]. It is therefore important to adhere to the principle of least authority [3]: to ensure that an untrusted entity cannot bring a smart contract into an inconsistent state, the smart contract should only return the bare minimum functionality required by the untrusted entity to participate in the smart contract.

## Example

```{.JavaScript}
const start = zcf => {
  let internalSeat;
  const setInternalSeat = seat => {
    internalSeat = seat;
    return "seat has been set as the internal seat";
  };
  const getInternalSeat = () => {
    return internalSeat;
  }
  const creatorInvitation = zcf.makeInvitation(setInternalSeat, 'set seat');
  const publicFacet = Far('publicFacet', {
    getSeat: getInternalSeat
  });
  return harden({ creatorInvitation, publicFacet });
};
harden(start);
export { start };
```

The smart contract shown above is a trivial smart contract which allows 2 things: it allows the creator to position itself on the `internalSeat`, and it allows any other entity to get this `internalSeat`.

``` {#avoidAbundantIntroductionTest .JavaScript language="JavaScript" caption="Bob can bring the smart contract into an inconsistent state" label="avoidAbundantIntroductionTest"}
//Alice starts an instance of the installation
const { creatorInvitation, publicFacet } = await zoe.startInstance(installation, {
  Asset: alphaCoin.issuer,
  Price: betaCoin.issuer,
});
//Alice shares the public facet with anyone that is interested
//Alice wants to sell 250 AlphaCoins for 500 BetaCoins
//she creates a proposal and a matching payment
const aliceSellProposal = harden({
  give: { Asset: AmountMath.make(alphaCoin.brand, 250n) },
  want: { Price: AmountMath.make(betaCoin.brand, 500n) },
  exit: { onDemand: null },
});
const alicePayment = { 
  Asset: alphaCoinPurseAlice.withdraw(AmountMath.make(alphaCoin.brand, 250n)) 
};
//Alice offers the invitation, proposal and payment to zoe
const aliceSeat = await E(zoe).offer(
  creatorInvitation,
  aliceSellProposal,
  alicePayment,
);
//Alice's seat should still be available
t.deepEqual(await E(aliceSeat).hasExited(), false);
//Bob uses the publicFacet to get the seat
const seat = await E(publicFacet).getSeat();
//Bob is able to exit the seat; unexpectedly changing the state of Alice's seat
await E(seat).exit();
//Bob exited Alice's seat without Alice expecting this
t.deepEqual(await E(aliceSeat).hasExited(), true);
```

The above test shows that creator Alice uses her `creatorInvitation` to make an offer to Zoe. By performing this offer, Alice's seat will be set to the `internalSeat`. Bob comes along, and wants to see what Alice is offering. Bob uses the `getSeat` method of the `publicFacet` to get the `internalSeat` of the smart contract. This introduces Alice's seat object to Bob. Bob decides to exit the seat object, leaving the smart contract in an inconsistent state. This is because Alice does not expect her seat to be exited: Alice should be the only entity that is able to exit her seat.

``` {.JavaScript}
const start = zcf => {
  let internalSeat;
  const setInternalSeat = seat => {
    internalSeat = seat;
    return "seat has been set as the internal seat";
  };
  const proposal = seat => {
    return {
      want: seat.getProposal().want,
      give: seat.getProposal().give,
    };
  }
  const getInternalSeat = () => {
    return proposal(internalSeat);
  }
  const creatorInvitation = zcf.makeInvitation(setInternalSeat, 'set seat');
  const publicFacet = Far('publicFacet', {
    getSeat: getInternalSeat
  });
  return harden({ creatorInvitation, publicFacet });
};
harden(start);
export { start };
```

The above smart contract resolves the issue portrayed in the initial smart contract: instead of returning the actual `internalSeat`, only the 'give' and the 'want' part of the proposal of the `internalSeat` are returned.

``` {.JavaScript}
//Alice starts an instance of the installation
const { creatorInvitation, publicFacet } = await zoe.startInstance(installation, {
  Asset: alphaCoin.issuer,
  Price: betaCoin.issuer,
});
//Alice shares the public facet with anyone that is interested
//Alice wants to sell 250 AlphaCoins for 500 BetaCoins
//she creates a proposal and a matching payment
const aliceSellProposal = harden({
  give: { Asset: AmountMath.make(alphaCoin.brand, 250n) },
  want: { Price: AmountMath.make(betaCoin.brand, 500n) },
  exit: { onDemand: null },
});
const alicePayment = { 
  Asset: alphaCoinPurseAlice.withdraw(AmountMath.make(alphaCoin.brand, 250n)) 
};
//Alice offers the invitation, proposal and payment to zoe
const aliceSeat = await E(zoe).offer(
  creatorInvitation,
  aliceSellProposal,
  alicePayment,
);
//Alice's seat should still be available
t.deepEqual(await E(aliceSeat).hasExited(), false);
//Bob uses the publicFacet to get the seat
const seat = await E(publicFacet).getSeat();
//Bob doesn't get the whole seat, only the give and want part of the proposal
t.throws(() => seat.exit(), {
  message: 'seat.exit is not a function',
});
//Bob can't alter this proposal
t.throws(() => seat.give = 1000, {
  message: 'Cannot assign to read only property \'give\' of object \'[object Object]\''
});
//Alice should still be in her seat
t.deepEqual(await E(aliceSeat).hasExited(), false);
```

The above test shows again the scenario where Alice performs an offer to Zoe, and thus sets her seat as the `internalSeat`. Again, Bob performs a `getSeat` method call using the `publicFacet` that it got from Alice. This time however, Bob gets back an object which only consists of the 'give' and the 'want' part of the proposal of Alice's seat. Bob can thus no longer call the `exit` object on this function, and also can't change any of the properties of the object. Therefore, Bob cannot use the object to bring the smart contract into an invalid state.

## General rule
The methods of the smart contract should only return the bare minimum needed to participate in the smart contract .

## Known uses
-   All studied smart contracts use this pattern.

## References
[1] Agoric, [Glossary](https://docs.agoric.com/glossary)

[2] Agoric, [Contract requirements](https://docs.agoric.com/guides/zoe/contract-requirements.html)

[3] Agoric, [Hardened javascript](https://docs.agoric.com/guides/js-programming/hardened-js.html)