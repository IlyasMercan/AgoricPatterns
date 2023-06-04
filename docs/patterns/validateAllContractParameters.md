# Validate all contract parameters

## Intent
Verifying
whether the information provided by the user is as expected.

## Consequences
-   The smart contract will not be put into an inconsistent state due to
    unexpected user input.
-   The smart contract will not perform unexpected behavior due to
    unexpected user input.

## Context
Smart contracts can receive user input. An example is via
methods, where the user input is passed as an argument to the method. It
should not be assumed by the smart contract developer that all user
input will be as expected. It is possible that an attacker provides
malicious user input on purpose in order to bring the smart contract in
an inconsistent state, or to let the smart contract execute unexpected
behavior. To avoid this, checks should be done whenever user input is
received. Note that once again, this pattern is related to the \"checks
effects interactions\" pattern in Solidity [1]. This is because this
pattern, just as the \"checks effects interactions\" pattern, suggests
to first perform all checks on the input parameters updating the
contract state.

## Example
``` {.JavaScript}
const start = zcf => {
  let naturalNumbersList = [];
  const getNaturalNumbersList = () => {
    return naturalNumbersList;
  }
  const addToNaturalNumbersList = value => {
    naturalNumbersList.push(value);
  };
  const creatorFacet = Far('creatorFacet', {
    getNaturalNumbersList
  });
  const publicFacet = Far('publicFacet', {
    addToNaturalNumbersList
  });
  return harden({ creatorFacet, publicFacet });
};
harden(start);
export { start };
```

The code above shows a smart contract in which any
entity with the `publicFacet` can add numbers to the
`naturalNumbersList` using the `addToNaturalNumbersList` method. The
creator of the smart contract can retrieve the natural numbers list
using the `getNaturalNumbersList` method, provided by the
`creatorFacet`.

``` {.JavaScript}
//Alice starts an instance of the smart contract
const { creatorFacet, publicFacet } = await E(zoe).startInstance(installation);
//Alice shares the publicFacet
//Bob adds natural number 8 to the naturalNumberList
await E(publicFacet).addToNaturalNumbersList(8);
//Carol wrongfully adds number -11 to the naturalNumberList
await E(publicFacet).addToNaturalNumbersList(-11);
//The naturalNumberList is now in an invalid state, since -11 is not a natural number
t.deepEqual(
  await E(creatorFacet).getNaturalNumbersList(),
  [8, -11]
);
```

As shown in this test code, Carol is able to add a negative
number to the `naturalNumbersList`. This brings the smart contract into
an invalid state, since there should only be natural numbers in the
`naturalNumbersList`.

``` {.JavaScript}
const start = zcf => {
  let naturalNumbersList = [];
  const getNaturalNumbersList = () => {
    return naturalNumbersList;
  }
  const addToNaturalNumbersList = value => {
    Nat(value);
    naturalNumbersList.push(value);
  };
  const creatorFacet = Far('creatorFacet', {
    getNaturalNumbersList
  });
  const publicFacet = Far('publicFacet', {
    addToNaturalNumbersList
  });
  return harden({ creatorFacet, publicFacet });
};
harden(start);
export { start };
```

The flaw in the original smart contract is resolved by performing user input
validation, as seen in the code above. Here, the
`addToNaturalNumbersList` first verifies whether the input is indeed a
natural number. If this is the case, the code will execute as usual. If
this is not the case, an exception is thrown [2].

``` {.JavaScript language="JavaScript"}
//Alice starts an instance of the smart contract
const { creatorFacet, publicFacet } = await E(zoe).startInstance(installation);
//Alice shares the publicFacet
//Bob adds natural number 8 to the naturalNumberList
await E(publicFacet).addToNaturalNumbersList(8);
//Carol wrongfully adds number -11 to the naturalNumberList
await t.throwsAsync(async () =>
  {
    await E(publicFacet).addToNaturalNumbersList(-11);
  },
  {
    instanceOf: RangeError,
    message: '-11 is negative'
  }
);
//The naturalNumberList remains in a valid state
t.deepEqual(
  await E(creatorFacet).getNaturalNumbersList(),
  [8]
);
```

The above test shows that it has become
impossible to pass anything but a natural number to
`addToNaturalNumbersList` method. The contract can no longer be brought
into an inconsistent state via the `addToNaturalNumbersList` method.

## General rule
If the user provides any form of input, then it should
be verified whether this given input is indeed as expected. 

## Known uses
-   The atomic swap smart contract: validates proposal shape.
-   The covered call smart contract: validates proposal shape.
-   The Over The Counter desk smart contract: validates proposal shape.
-   The sell items smart contract: validates proposal shape, want part
    of proposal, issuer keywords, whether brand is fungible.
-   The simple exchange smart contract: validates the issuer keywords
-   The loan smart contract: validates issuer keywords, presence of
    terms, proposal shape
-   The funded call spread smart contract: validates whether brand is
    fungible, proposal shape
-   The priced call spread smart contract: validates whether brand is
    fungible, proposal shape.
-   The second price auction smart contract: validates proposal shape,
    issuer keywords
-   The escrow to vote smart contract: validates proposal shape, issuer
    keywords, whether brand is fungible.
-   The use object smart contract: validates proposal shape, issuer
    keywords.

## References
[1] F. Volland, “Checks effects interactions,” (accessed April 15, 2023). [Online]. Available: https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html

[2] Agoric, “Nat,” 2021, (accessed April 3, 2023). [Online]. Available: https://github.com/Agoric/nat/tree/10643089592f04be2bea4222202067835ceaba79