# The salesperson pattern

## Intent
Creation of a smart
contract where 1 instance of the smart contract will allow for an
arbitrary amount of trades, where each trade is an exchange between a
customer and the creator.

## Consequences
-   There are an arbitrary amount of traders that all trade with the
    creator.
-   The smart contract is able to run indefinitely.

## Structure
![Seat structure diagram of the salesperson
pattern](./images/theSalespersonPattern.PNG){#theSalespersonPatternSeatStructureDiagram
width="9cm"}

Again, the way that the buyer gets hold of the `customerInvitation` is a
design choice made by the smart contract designer: it is not included in
this pattern.

## Participants
-   Creator: the entity that starts an instance of the smart contract.
-   Customer: the entity that wants to trade with the creator.

## Implementation
``` {.JavaScript}
const start = zcf => {
  let internalSeat;
  const sellOfferHandler = creatorSeat => {
    internalSeat = creatorSeat;
    //...
  };
  const buyOfferHandler = customerSeat => {
    //some trade with between customerSeat and internalSeat
  };
  const creatorInvitation = zcf.makeInvitation(sellOfferHandler, 'sell');  
  return harden({ creatorInvitation });
};
harden(start);
export { start };
```

The smart contract has an internal seat `internalSeat`. The first thing
that the creator should do, is register itself as the seller. The offer
handler linked to the `creatorInvitation` will set the `internalSeat`
equal to the `creatorSeat`. Thus, by offering the `creatorInvitation`,
the creator can register itself as the salesperson. All customers will
trade with this `internalSeat`: the offer handler related to the
`customerInvitation` will perform some kind of reallocation between the
`customerSeat` and the `internalSeat`. Just as in the managed assets
pattern, the high-level overview of the implementation of the
salesperson pattern does not include how the invitation linked to the
`buyOfferHandler` is obtained. This is a decision that should be made by
the contract designer. 

## Known uses
-   The sell items smart contract.

## Related patterns
-   The managed assets pattern: as stated before, both the managed
    assets pattern and the salesperson pattern run indefinitely, and
    provide a way to let a creator trade with an arbitrary amount of
    customers.
-   The exchange pattern: as stated before, both patterns allow a
    contract to run indefinitely. The only difference is who the
    participants of the smart contract trade with: all customers trade
    with the creator in the salesperson pattern, while all traders trade
    with one another in the exchange pattern.