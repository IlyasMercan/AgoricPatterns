# The bilateral trade agreement pattern

## Intent
Creation of
a smart contract where 1 instance of the smart contract will allow for
exactly 1 trade with another entity, where the entity that created the
smart contract determines what other entity it will trade with.

## Consequences

-   One instance of the smart contract guarantees exactly 1 trade.

-   There are exactly 2 trading entities.

-   The creator determines what is to be traded.

-   The counterparty is dependent on the creator to get an invitation.

-   The counterparty either adheres to the conditions imposed by the
    creator and trades, or does not trade at all.

## Structure
<img src="https://raw.githubusercontent.com/IlyasMercan/AgoricPatterns/main/docs/patterns/images/theBilateralTradeAgreementPattern.PNG" width="400">

## Participants
-   Creator: the entity that starts an instance of the smart contract.
-   Counterparty: the entity who the creator will trade with. The
    counterparty is determined by the creator.

## Implementation

```js
const start = zcf => {
  const firstOfferOfferHandler = creatorSeat => {
    assertProposalShape(creatorSeat, /*some shape*/);
    const counterOfferOfferHandler = counterPartySeat => {
      //some reallocation
      //exit all seats
      //return success message
    };
    const counterPartyInvitation = zcf.makeInvitation(
      counterOfferOfferHandler,
      'counterOffer',
      /*some shape*/
    );
    return counterPartyInvitation;
  };
  const creatorInvitation = zcf.makeInvitation(
    firstOfferOfferHandler,
    'firstOffer',
  );
  return { creatorInvitation };
};
harden(start);
export { start };
```

The smart contract returns a single `creatorInvitation`. The offer
handler linked to the `creatorInvitation` will return a
`counterPartyInvitation` which is an invitation for the counterparty.
Thus: when the creator uses this `creatorInvitation` in an offer and
gets a `creatorSeat` in return, the creator will be able to call the
`getOfferResult` method on the `creatorSeat` in order to obtain the
`counterPartyInvitation`. The creator can then start looking for a
counterparty who is interested to participate in the trade. Once this
counterparty has been found, the creator can send the
`counterPartyInvitation` to this entity. In the offer handler of the
`counterPartyInvitation`, a reallocation happens, each seat is exited
and a string message is returned. Thus, when the counterparty uses the
`counterPartyInvitation` in an offer, the trade will be completed. Note
that optional assertions about the shape of each proposal can be made.

## Known uses
-   The [atomic swap](https://docs.agoric.com/guides/zoe/contracts/atomic-swap.html) smart contract.
-   The [covered call](https://docs.agoric.com/guides/zoe/contracts/covered-call.html) smart contract.
-   The [loan](https://docs.agoric.com/guides/zoe/contracts/loan.html) smart contract.
-   The [funded call spread](https://docs.agoric.com/guides/zoe/contracts/fundedCallSpread.html) smart contract.

## Related patterns
-   The dependent participation pattern and the independent
    participation pattern: the bilateral trade agreement pattern could
    be seen as a third way to create an invitation. In contrast to both
    these patterns, the bilateral trade agreement pattern enforces the
    creation of exactly 1 invitation for the counterparty.