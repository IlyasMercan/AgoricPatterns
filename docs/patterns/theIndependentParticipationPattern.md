# The independent participation pattern

## Intent
Giving the
counterparty the right to create an arbitrary amount of invitations to
participate in the smart contract.

## Consequences
-   Once the counterparty has the `publicFacet`, the counterparty is no
    longer dependent on the creator. The counterparty is able to create
    its own invitations, and can thus participate in the smart contract
    whenever it pleases.
-   One instance of the smart contract guarantees an arbitrary amount of
    trades.
-   The creator does not get to choose the counter parties that it will
    trade with.

## Structure
<img src="https://raw.githubusercontent.com/IlyasMercan/AgoricPatterns/main/docs/patterns/images/theIndependentParticipationPattern.PNG" width="400">

## Participants
-   Creator: the entity that starts an instance of the smart contract.
-   Counterparty: the entity that wants to trade with the creator.

## Implementation
```js
const start = zcf => {
  const offerHandler = seat => {
    //...
  };
  const publicFacet = Far('publicFacet', {
    makeInvitation: () => zcf.makeInvitation(offerHandler, 'description');
  });
  return harden({ publicFacet });
};
harden(start);
export { start };
```

The smart contract returns a `publicFacet` which contains a
`makeInvitation` method to create invitations. This way, when the
creator shares the `publicFacet` with the counterparty, the counterparty
can create an arbitrary amount of invitations using the `makeInvitation`
method.

## Known uses
-   The [automatic refund](https://docs.agoric.com/guides/zoe/contracts/automatic-refund.html) smart contract.
-   The [barter exchange](https://docs.agoric.com/guides/zoe/contracts/barter-exchange.html) smart contract.
-   The [mint and sell NFTs](https://docs.agoric.com/guides/zoe/contracts/mint-and-sell-nfts.html) smart contract.
-   The [oracle](https://docs.agoric.com/guides/zoe/contracts/oracle.html) smart contract.
-   The [sell items](https://docs.agoric.com/guides/zoe/contracts/sell-items.html) smart contract.
-   The [simple exchange](https://docs.agoric.com/guides/zoe/contracts/simple-exchange.html) smart contract.
-   The [use object](https://docs.agoric.com/guides/zoe/contracts/use-obj-example.html) smart contract.

## Related patterns
-   The dependent participation pattern: as explained in the dependent
    participation pattern, the independent participation pattern and the dependent participation pattern are almost identical. The difference is that in the independent participation pattern, the makeInvitation method is provided in the publicFacet and not the creatorFacet. This means that the counterparty does not depend on the creator to receive invitations.
-   The bilateral trade pattern: these patterns
    relate since they both determine a way for a counterparty to receive invitations. The difference is that in the independent participation pattern, an arbitrary amount of invitations can be created for 1 instance of the smart contract, while in the bilateral trade agreement, only 1 invitation is created for 1 instance of the smart contract. Another difference is that in the independent participation pattern, the counterparty does not depend on the creator to receive an invitation.