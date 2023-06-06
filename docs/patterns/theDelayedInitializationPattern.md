# The delayed initialization pattern

## Intent
Enforcing the
entity that configures a smart contract to pass a certain parameter, but
allowing this entity to set that parameter after an instance of the
smart contract has started.

## Consequences
-   The entity that configures the smart contract is forced to pass
    along certain parameters.
-   The entity that configures the smart contract is allowed to pass
    along the enforced parameters after the smart contract has been
    started.

## Context
When starting an instance of a smart contract, Agoric
provides 2 ways to pass parameters to the smart contract: the `terms`
argument and the `privateArgs` parameter [1]. The difference between
these 2 ways is that `terms` are public, while `privateArgs` are private
[1]. Both `terms` and `privateArgs` are optional parameters [1].
The delayed initialization pattern can enforce the configuring entity to
set certain parameters, by rejecting to give back any useful methods
until the parameters are set. It does so by returning a `creatorFacet`
with a single method called `initialize`. This `initialize` method
should take at least 1 argument. The `initialize` method will configure
an attribute / a set of attributes of the smart contract using the
passed along argument(s). The `initialize` method then returns the
`realCreatorFacet` which holds the methods that should have been
provided in the `creatorFacet`. Thus, in the delayed initialization
pattern, the `creatorFacet` acts as a gatekeeper that will only return
the methods that it holds if the configuring entity passes along the
parameters that should be configured. It is important to note that the
`privateArgs` can also be enforced to be set by adding checks to the
start of the smart contract. These checks can determine whether the
`privateArgs` are present and set as expected, and throw an error
otherwise. The true benefit of the delayed initialization pattern is
that it allows the user to first start an instance of the smart
contract, and then enforce the user to pass a certain parameter while
the smart contract is already running. This is in contrast with
enforcing `privateArgs` with checks: here, the user is forced to pass
the parameter at the moment that an instance of the smart contract is
started.

## Example
``` {.JavaScript}
const start = async zcf => {
  let requiredAttribute;
  const realCreatorFacet = Far('realCreatorFacet', {
    method: () => {
      //...
    }
    //...
  });
  const initialize = (valueForRequiredAttribute) => {
    requiredAttribute = valueForRequiredAttribute;
    return realCreatorFacet;
  }
  const creatorFacet = Far('creatorFacet', {
    initialize
  });
  return harden({ creatorFacet });
};
harden(start);
export { start };
```

The code above shows a smart contract that returns a
single `creatorFacet` with a single method `initialize`. The
`initialize` method expects one argument `valueForReauiredAttribute`. In
the `initialize` method, the `requiredAttribute` of the smart contract
is given the value of the `valueForRequiredAttribute` parameter. Then,
the actual creator facet `realCreatorFacet` is returned. This
`realCreatorFacet` holds the actual methods that are of interest to the
configuring entity.

## General rule
If the configuring entity should
set an attribute of the smart contract, but the configuring entity
should be allowed to set the attribute after the smart contract has
already started, then the delayed configuration pattern can be used.

## Known uses
-   The [oracle](https://docs.agoric.com/guides/zoe/contracts/oracle.html) smart contract.

## References
[1] Agoric, [Zoe service](https://docs.agoric.com/reference/zoe-api/zoe.html)