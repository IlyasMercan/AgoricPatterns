# Verifying untrusted invitations

## Intent
Providing a way
for an entity to establish trust in the invitation that it receives.

## Consequences
-   The entity that received the invitation is sure that the invitation
    can be trusted. Thus: the invitation is indeed the invitation to the
    smart contract that the entity expected to participate in.

## Context
For financial transactions to be possible, entities must be
able to send one another invitations to participate in smart contracts.
The issue is that an entity can't trust the invitations it receives from
other entities. To resolve this, Zoe provides an `invitationIssuer`
object. The `invitationIssuer` object can be used by an entity to claim
an invitation, given an untrusted invitation [1].

## Example
Note
that the following example is based on the default way by which Agoric
handles untrusted invitations as, for example, illustrated in the atomic
swap smart contract test code provided by Agoric [2].

``` {#invitationIssuer .JavaScript language="JavaScript" caption="Bob uses the invitationIssuer to claim the untrusted counterInvitation" label="invitationIssuer"}
//Bob does not trust the invitation
//Bob claims the invitation using Zoe's invitationIssuer
const invitationIssuer = await E(zoe).getInvitationIssuer();
const invitationBob = await E(invitationIssuer).claim(counterInvitation);
```

An example of this is shown in the code above: Bob gets an invitation
`counterInvitation` from Alice, who he doesn't trust. Bob first gets the
`invitationIssuer` from Zoe, and then claims his invitation by passing
the `invitationIssuer` the untrusted `counterInvitation`. 

## General rule
An untrusted invitation can be turned into a trusted invitation
by claiming it, using the `invitationIssuer` provided by `zoe`.

## Known uses
-   The claiming of untrusted invitations is needed in the test code / dapp for each smart contract that makes use of invitations.

## References
[1] Agoric, [Glossary](https://docs.agoric.com/glossary)

[2] Agoric, [test-atomicswap.js](https://github.com/Agoric/agoric-sdk/blob/f29591519809dbadf19db0a26f38704d87429b89/packages/zoe/test/unitTests/contracts/test-atomicSwap.js)