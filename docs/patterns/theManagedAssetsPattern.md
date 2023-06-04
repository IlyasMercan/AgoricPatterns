# The managed assets pattern

## Intent
Creating a level of
indirection between the creator of the smart contract and a potential
customer.

## Consequences
-   There are an arbitrary amount of traders that all trade with the
    creator.
-   The smart contract is able to run indefinitely.
-   The creator does not need to play an active role in the trade: the
    sole purpose of the trader is to provide input to the `internalSeat`
    by offering a `depositInvitation`, and collect output of the
    `internalSeat` by offering a `withdrawInvitation`.
-   The creator and the customer do not trade directly: both the
    customer and the creator will only trade with the `internalSeat`.
-   Due to the fact that the withdrawing is performed by offering a
    `withdrawInvitation`, gains are given to anyone that offers a
    `withdrawInvitation`. The creator must therefore be careful not to
    share a `withdrawInvitation` with any other entity, because any
    entity that offers this `withdrawInvitation` can get the gains from
    the `internalSeat`.

## Structure
![Seat structure diagram of the managed assets
pattern](./images/theManagedAssetsPattern.PNG)

Note that this seat structure diagram does not
specify how the customer got hold of the `customerInvitation`. This is
because the way that the customer obtained the `customerInvitation` is
irrelevant for the managed assets pattern. An example of how the
customer could have gotten the `customerInvitation` is via the
independent participation pattern.

## Participants
-   Creator: the entity that starts an instance of the smart contract.
-   Customer: the entity that wants to trade with the creator.

## Implementation
``` {.JavaScript}
const start = async zcf => {
  const { zcfSeat: internalSeat } = zcf.makeEmptySeatKit();
  const makeDepositOfferHandler = depositSeat => {
    //some reallocation to internalSeat
  }
  const makeWithdrawOfferHandler = withdrawSeat => {
    //some reallocation from internalSeat
  }
  const customerOfferHandler = customerSeat => {
    //some reallocation to internalSeat
  }
  const creatorFacet = Far('creatorFacet', {
    makeDepositInvitation: () => zcf.makeInvitation(makeDepositOfferHandler, 'deposit'),
    makeWithdrawInvitation: () => zcf.makeInvitation(makeWithdrawOfferHandler, 'withdraw')
  });
  return harden({ creatorFacet });
};
harden(start);
export { start };
```

The smart contract has an internal seat `internalSeat` which is used as
a level of indirection between the creator and the customer. A customer
will perform all of its trades with the `internalSeat`. The smart
contract returns a `creatorFacet`. This `creatorFacet` provides the
creator with methods to create `depositInvitations` and
`withdrawInvitations`. A `depositInvitation` can be used in an offer to
deposit inventory to the `internalSeat`. A `withdrawInvitation` can be
used in an offer to withdraw gains made from the trades from the
`internalSeat`. Note that in the code above the
`customerOfferHandler` is not linked to any invitation. The contract
designer should determine how invitations are made available to
customers.

## Known uses
-   The oracle smart contract.
-   The Over The Counter desk smart contract.

## Related patterns
-   The salesperson pattern: both these patterns are market places where
    1 creator performs trade with an arbitrary amount of customers. The
    difference is that the managed assets patterns provides an extra
    level of indirection between the customer and the creator, while the
    salesperson pattern does not provide this extra level of
    indirection.