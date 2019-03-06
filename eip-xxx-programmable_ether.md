---
eip: <to be assigned>
title: Programmable Ether
author: Zack Rubenstein (@zrubenst)
discussions-to: *TBD*
status: Draft
type: Standards Track
category: Core
created: 2019-03-04
---

## Simple Summary
Allow smart contracts to transfer Ether on behalf of users

## Abstract
Users will be able to allow/disallow smart contracts to transfer their Ether, and smart contracts would be able to transfer Ether from an address if and only if it is allowed to do so. 

## Motivation
Currently, Ether can only be transferred by a user by them signing a transaction with the value set to an amount to transfer (in Wei), or by a contract sending it's Ether from it's own balance to another. Enabling smart contracts to move Ether on behalf of another address will allow for a multitude of new oppurtunuties for developers to improve their dapps user experience, and discover new applications of Ethereum.

A good example are decentralized exchanges. Early implementations relied exclusively on smart contracts, where ultimatley users paid transaction fees for placing and cancelling trades. More recent dexes have started to have users sign messages to place/cancel trades, removing the need to have users pay any transaction fees. This improvement was made possible through the adoption of [ERC20](https://eips.ethereum.org/EIPS/eip-20), which allow smart contracts to move tokens on behalf of the token holder (token holders must approve that specific contract to do so prior). ERC20 tokens have become an integral part of the ecosystem, and Ether has no way of being compliant. By allowing smart contracts to be approved to move Ether, dex users will be able to use the platform and trade Ether directly.

The possible applications for this proposal extend far beyond decentralized exchanges. Every technology that makes use of ERC20 compliant assets stands to benefit, many business models currently in the ecosystem could be improved, and many new applications of Ethereum may become possible.

## Specification

Add a new member function `transferTo` to the solidity `address` type which transfers Ether from that address to a given address. Used like so

```
address fromAddress = 0x123;
address toAddress = 0xabc;
fromAddress.transferTo(toAddress, 1 ether) // transfers 1 ether
```

Calling `transferTo` from an address which has not approved the smart contract to do so will result in a `revert`. A companion member function `sendTo` would be included with the same parameters, except this one returns `false` instead of reverting.

Contracts can be approved to transfer Ether by the user making a transaction with the `to` field set to their own address, and calling `approve(address _spender)` on themselves as if it were a smart contract method call. Example:

```
{
    to: 0x123,
    from: 0x123,
    data: 0xeff0d125f90000000000000000000000001111111111111111111111111111111111111111,
}
```

This is the raw transaction that, if signed and committed by the owner of `0x123`, would approve the contract deployed at `0x1111111111111111111111111111111111111111` to transfer `0x123`'s Ether using the `transferTo` function.

Another function `unapprove(address _spender)` would also be added, used in the same way as approve. This would disallow the specified smart contract from calling `transferTo`.

### Minor Changes

In addition to the new member functions on `address` types, a function `approved(address spender)` would be added so that contracts can know if an address has approved a smart contract or not, example:

```
if (0x123.approved(this)) 0x123.transferTo(this, 1 ether) 
```

## Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases
Test cases for an implementation are mandatory for EIPs that are affecting consensus changes. Other EIPs can choose to include links to test cases if applicable.

## Implementation
The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
