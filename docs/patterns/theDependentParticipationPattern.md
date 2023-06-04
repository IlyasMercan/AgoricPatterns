# The dependent participation pattern

## Intent
Giving the
creator the right to create an arbitrary amount of invitations, which it
can share with potential counter parties.

## Consequences
-   The counterparty can only obtain an invitation if the creator
    explicitly gives an invitation to the counterparty. The counterparty
    is therefore dependent on the creator to get invitations.
-   One instance of the smart contract guarantees an arbitrary amount of
    trades.
-   The creator gets to choose the counter parties that it will trade
    with.

## Structure
<img src="https://raw.githubusercontent.com/IlyasMercan/AgoricPatterns/main/docs/patterns/images/theDependentParticipationPattern.PNG" width="400">

## Participants
-   Creator: the entity that starts an instance of the smart contract.
-   Counterparty: the entity that wants to trade with the creator.

## Implementation

``` {.JavaScript}
const start = zcf => {
  const offerHandler = seat => {
    //...
  };
  const creatorFacet = Far('creatorFacet', {
    makeInvitation: () => zcf.makeInvitation(offerHandler, 'description')
  });
  return harden({ creatorFacet });
};
harden(start);
export { start };
```

The smart contract returns a `creatorFacet` with a method
`makeInvitation`. This allows the creator to create an arbitrary amount
of invitations.

## Known uses
-   The mint payments smart contract.
-   The Over The Counter desk smart contract.
-   The priced call spread smart contract.
-   The second-price auction smart contract.
-   The escrow to vote smart contract.

Note that, although the sell items smart contract does return a
`creatorFacet` with a `makeBuyerInvitation` method, it is not included
in the list of known uses. This is because both the `publicFacet` and
the `creatorFacet` offer this `makeBuyerInvitation` method: there is
thus no guarantee that the creator gets to choose the counter parties
that it will trade with.

## Related patterns
-   The independent participation pattern: this pattern is nearly
    identical to the dependent participation pattern. The difference is
    that in the independent participation pattern, the `makeInvitation` method is provided in the `publicFacet`
    and not the `creatorFacet`. This means that the counterparty does
    not depend on the creator to receive invitations.
-   The bilateral trade pattern: in this pattern, the counterparty is
    also dependent on the creator. The difference is that in the
    bilateral trade pattern, one instance of the smart contract
    guarantees exactly one trade, while in the dependent participation
    pattern once instance of the smart contract guarantees an arbitrary
    amount of trades.