# The secure shutdown pattern

## Intent
Ensuring that no
tokens are lost when a smart contract with an internal seat is shut
down.

## Consequences
-   No tokens are lost when a smart contract is shut down.

## Context
Smart contracts can have a seat as an attribute, as seen in
the managed assets pattern. When a contract shuts down, the funds
allocated to this internal seat are lost. This is a known problem in Agoric [1] [2].
Thus, a smart contract developer should ensure that all funds are
withdrawn from the internal seat before the contract is shut down.

## Example
```js
const start = async zcf => {
  const { zcfSeat: internalSeat } = zcf.makeEmptySeatKit();
  const shutdown = seat => {
    zcf.shutdown('contract expired');
  };
  const deposit = seat => {
    internalSeat.incrementBy(
      seat.decrementBy(harden(seat.getCurrentAllocation())),
    );
    zcf.reallocate(internalSeat, seat);
    seat.exit();
    return 'added to internalSeat';
  }
  const getCurrentAllocation = () => {
    return internalSeat.getCurrentAllocation();
  }
  const creatorFacet = Far('creatorFacet', {
    makeShutdownInvitation: () => zcf.makeInvitation(shutdown, 'shutdown'),
    makeDepositInvitation: () => zcf.makeInvitation(deposit, 'deposit'),
    getAllocation: getCurrentAllocation
  });
  return harden({ creatorFacet });
};
harden(start);
export { start };
```

This piece of code shows an example of a contract where the
secure shutdown pattern has not been applied. In this smart contract,
the creator can deposit payments to the `internalSeat` via the
`makeDepositInvitation` method, the creator can check the current
balance of the `internalSeat` via the `getCurrentAllocation` method, and
the creator can shut down the smart contract via the
`makeShutdownInvitation` method. However, the `shutdown` method (which
is the offer handler related to the shut down invitation) shuts down the
smart contract without reallocating the funds from the `internalSeat` to
the seat of the entity that issued the shut down invitation.

```js
//Alice starts an instance of the installation
const { creatorFacet } = await zoe.startInstance(installation, {
  Asset: alphaCoin.issuer,
  Price: betaCoin.issuer,
});
//Alice stores assets on the internal seat
const depositInvitationAlice = await E(creatorFacet).makeDepositInvitation();
//Alice states that she wants to add 250 AlphaCoins to the internalSeat
const depositProposalAlice = harden({
  give: { Asset: AmountMath.make(alphaCoin.brand, 250n) },
  exit: { onDemand : null}
});
//Alice gets these coins out of her purse, and collects them in a payment
const depositPaymentsAlice = {
  Asset: alphaCoinPurseAlice.withdraw(AmountMath.make(alphaCoin.brand, 250n)),
};
const aliceSeatDeposit =
    await E(zoe).offer(depositInvitationAlice, depositProposalAlice, depositPaymentsAlice)
//The internal seat should now have 250 AlphaCoins
t.deepEqual(
  (await E(creatorFacet).getAllocation()).Asset.value,
  250n
);
//Alice shuts down the smart contract
const shutDownInvitation = await E(creatorFacet).makeShutdownInvitation();
const aliceShutdownSeat = await E(zoe).offer(shutDownInvitation);
//We ensure that Alice did not get any money back
//Alice can't get payout from deposit seat: this payment is used up
await t.throwsAsync(async () => {
  await E(aliceSeatDeposit).getPayout('Asset').then(payment => {
    alphaCoinPurseAlice.deposit(payment)
  })
});
//There are no payouts in the aliceShutdownSeat
t.deepEqual(await E(aliceShutdownSeat).getPayouts(), {});
t.deepEqual(alphaCoinPurseAlice.getCurrentAmount(), AmountMath.make(alphaCoin.brand, 750n));
//Thus: where did the 250 AlphaCoins go?
```

As shown in this test, Alice first uses the smart contract to
deposit 250 AlphaCoins to the `internalseat`. She then shuts down the
smart contract without first withdrawing the 250 AlphaCoins from the
`internalseat`. By doing this, the 250 AlphaCoins are lost.

```js
const start = async zcf => {
  const { zcfSeat: internalSeat } = zcf.makeEmptySeatKit();
  const shutdown = seat => {
    seat.incrementBy(
      internalSeat.decrementBy(harden(internalSeat.getCurrentAllocation())),
    );
    zcf.reallocate(seat, internalSeat);
    zcf.shutdown('contract expired');
  };
  const deposit = seat => {
    internalSeat.incrementBy(
      seat.decrementBy(harden(seat.getCurrentAllocation())),
    );
    zcf.reallocate(internalSeat, seat);
    seat.exit();
    return 'added to internalSeat';
  }
  const getCurrentAllocation = () => {
    return internalSeat.getCurrentAllocation();
  }
  const creatorFacet = Far('creatorFacet', {
    makeShutdownInvitation: () => zcf.makeInvitation(shutdown, 'shutdown'),
    makeDepositInvitation: () => zcf.makeInvitation(deposit, 'deposit'),
    getAllocation: getCurrentAllocation
  });
  return harden({ creatorFacet });
};
harden(start);
export { start };
```

This code above shows a modified version of the initial smart contract. In this smart contract, the `shutdown` method first
reallocates all funds from the `internalSeat` to the seat that issued
the shutdown invitation.

```js
//Alice starts an instance of the installation
const { creatorFacet } = await zoe.startInstance(installation, {
  Asset: alphaCoin.issuer,
  Price: betaCoin.issuer,
});
//Alice stores assets on the internal seat
const depositInvitationAlice = await E(creatorFacet).makeDepositInvitation();
//Alice states that she wants to add 250 AlphaCoins to the internalSeat
const depositProposalAlice = harden({
  give: { Asset: AmountMath.make(alphaCoin.brand, 250n) },
  exit: { onDemand : null}
});
//Alice gets these coins out of her purse, and collects them in a payment
const depositPaymentsAlice = {
  Asset: alphaCoinPurseAlice.withdraw(AmountMath.make(alphaCoin.brand, 250n)),
};
const aliceSeatDeposit =
    await E(zoe).offer(depositInvitationAlice, depositProposalAlice, depositPaymentsAlice)
//The internal seat should now have 250 AlphaCoins
t.deepEqual(
  (await E(creatorFacet).getAllocation()).Asset.value,
  250n
);
//Alice shuts down the smart contract
const shutDownInvitation = await E(creatorFacet).makeShutdownInvitation();
const aliceShutdownSeat = await E(zoe).offer(shutDownInvitation);
//There is a payout in the aliceShutdownSeat
await E(aliceShutdownSeat).getPayout('Asset').then(payment => { 
  alphaCoinPurseAlice.deposit(payment)
});
t.deepEqual(alphaCoinPurseAlice.getCurrentAmount(), AmountMath.make(alphaCoin.brand, 1000n));
```

The test above shows that Alice gets her 250
AlphaCoins that she deposited to the `internalSeat` back when she offers
the shutdown invitation.

## General rule
If a smart contract uses an
internal seat then all funds allocated to this internal seat should be
withdrawn before shutting down the smart contract.

## Known uses
-   The [oracle](https://docs.agoric.com/guides/zoe/contracts/oracle.html) smart contract.

## References
[1] anilhelvaci, [What if a contract dies when it is holding assets?](https://github.com/Agoric/agoric-sdk/discussions/5469)

[2] N. Lai, [A contract must not lose assets if it dies](https://github.com/Agoric/agoric-sdk/issues/5486)