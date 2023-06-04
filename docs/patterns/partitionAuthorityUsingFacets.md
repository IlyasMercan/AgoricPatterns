# Partition authority using facets

## Intent
Separate what participants are allowed to perform what actions.

## Consequences
-   The creator of the smart contract gets access to methods hosted in the `creatorFacet`.
-   The users of the smart contract get access to methods hosted in the `publicFacet`.

## Context
An Agoric smart contracts returns a record which consists of (a subset of) the following 3 objects: a `creatorInvitation`, a `creatorFacet` and a `publicFacet` [1]. The `creatorInvitation` is an invitation that is given to the entity that started an instance of the smart contract [2]. Agoric describes a facet as *an object that exposes an API or particular view of some larger entity, which may be an object itself* [2]. The `creatorFacet` is a facet that is given to the entity that starts an instance of the smart contract, while the `publicFacet` is available to any entity that has a reference to the `instance` of the smart contract [1]. Thus: the `creatorFacet` should host methods that should only be available to the entity that started the instance of the smart contract, while the `publicFacet` should host methods that are available to potentially all entities.

## Example

``` {.JavaScript}
const start = async zcf => {
  const getConfidentialInformation = () => {
    return 'confidential information';
  } 
  const getPublicInformation = () => {
    return 'public information';
  }
  const publicFacet = Far('publicFacet', {
    getConfidentialInformation,
    getPublicInformation
  });
  return harden({ publicFacet });
};
harden(start);
export { start };
```

The code above shows a smart contract with 2 methods. The `getConfidentialInformation` method returns information that should only be known by the entity that started an instance of the smart contract. The `getPublicInformation` method returns information which can be known by all users. Both methods are made available via the `publicFacet`.

``` {.JavaScript}
//Alice starts an instance of the installation
const { publicFacet } = await zoe.startInstance(installation);

//Alice made the instance of the smart contract available via the board
 
//Bob uses the instance to get the publicFacet
  
//Bob uses the publicFacet to call the confidential method
t.deepEqual(
await E(publicFacet).getConfidentialInformation(),
'confidential information'
```

In the test code above, Alice starts an instance of the initial smart contract. Alice posts the `instance` of
the smart contract to the board, which Agoric describes as *a shared, on-chain location where users can post a value and make it accessible to others* [2] [3]. Bob retrieves the `instance` of the smart contract from the board and uses it to retrieve the `publicFacet`. Then,
Bob can use the `publicFacet` to access the confidential information via the `getConfidentialInformation` method. This is a problem, since only Alice should be able to access the `getConfidentialInformation` method.

``` {.JavaScript}
const start = async zcf => {
  const getConfidentialInformation = () => {
    return 'confidential information';
  }
  const getPublicInformation = () => {
    return 'public information';
  }
  const creatorFacet = Far('creatorFacet', {
    getConfidentialInformation
  });
  const publicFacet = Far('publicFacet', {
    getPublicInformation
  });
  return harden({ creatorFacet, publicFacet });
};
harden(start);
export { start };
```

The above code shows an improved version of the initial smart contract. In listing this smart contract, the `getConfidentialInformation` method is returned in the `creatorFacet` instead of in the `publicFacet`.

``` {.JavaScript}
//Alice starts an instance of the installation
const { creatorFacet, publicFacet } = await zoe.startInstance(installation);
  
//Alice made the instance of the smart contract available via the board
  
//Bob uses the instance to get the publicFacet
 
//Bob can no longer access the getConfidentialInformation method  
await t.throwsAsync(async () =>
  {
      await E(publicFacet).getConfidentialInformation()
  },
  {
    instanceOf: TypeError,
    message: 'target has no method "getConfidentialInformation", has ["getPublicInformation"]'
  }
);
```

As shown in the tests above, the change in the smart contract ensures that Bob can no longer access the `getConfidentialInformation` method, since it is no longer available to him.

## General rule
All methods that should be made publicly available should be hosted in the `publicFacet`, and all methods that should only be used by the entity that started an instance of the smart contract should be hosted in the `creatorFacet`. In short, use the facets as described by Agoric [1].

## Known uses
-   All studied smart contracts use this pattern.

## References
[1] Agoric, “Contract requirements,” 2022, (accessed December 17, 2022). [Online]. Available: https://docs.agoric.com/guides/zoe/contract-requirements.html

[2] Agoric, “Glossary,” 2022, (accessed December 10, 2022). [Online]. Available: https://docs.agoric.com/glossary

[3] Agoric, “Deploying smart contracts,” 2023, (accessed April 23, 2023). [Online]. Available: https://docs.agoric.com/guides/getting-started/deploying.html