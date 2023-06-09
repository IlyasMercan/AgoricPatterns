# The composite contract pattern

## Intent
Reusing a generic
smart contract by wrapping the generic smart contract within another
smart contract that configures the generic smart contract.

## Consequences
-   Separation of concerns: the sole responsibility of the wrapper smart
    contract is to configure the wrapped smart contract. The sole
    responsibility of the wrapped smart contract is to hold the generic
    core functionality.
-   Reusability: the wrapped smart contract that supports the core
    functionality must only be written once. It can then be reused by
    different kinds of wrapper smart contracts that configure this
    wrapped smart contract in different ways.

## Structure
<img src="https://raw.githubusercontent.com/IlyasMercan/AgoricPatterns/main/docs/patterns/images/theCompositeContractPattern.PNG" width="400">

## Participants
-   Creator: the entity that starts an instance of the smart contract.
-   Counterparty: the entity that wants to trade with the creator.

## Implementation

```js
const start = zcf => {
    const zoe = zcf.getZoeService();
    const startInstanceOfWrappedContract = ({
      installationWrappedContract,
      /*other parameters*/
    }) => {
      const issuerKeywordRecord = harden({/*issuer keywords*/});
      //...
      return E(zoe).startInstance(
        installationWrappedContract,
        issuerKeywordRecord,
        /*terms*/
      );
    };
    const creatorFacet = Far('creatorFacet', {
      startInstanceOfWrappedContract
    });
    return harden({ creatorFacet });
  };
  harden(start);
  export { start };
```

The composite pattern consists of 2 smart contracts: an outer smart
contract called the wrapper smart contract, and an inner smart contract
called the wrapped smart contract. The wrapped smart contract is a
generic smart contract that is to be configured. The wrapper smart
contract starts an instance of the wrapped smart contract and configures
this wrapped smart contract. The wrapper smart contract then returns the
objects returned by the wrapped smart contract. 

## Known uses
-   The [mint and sell NFTs](https://docs.agoric.com/guides/zoe/contracts/mint-and-sell-nfts.html) smart contract.
-   The [Over The Counter desk](https://docs.agoric.com/guides/zoe/contracts/otc-desk.html) smart contract.
-   The [loan](https://docs.agoric.com/guides/zoe/contracts/loan.html) smart contract.

## Related patterns
/