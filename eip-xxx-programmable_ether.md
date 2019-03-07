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
Currently, Ether can only be transferred by a user by them signing a transaction with the value set to an amount to transfer (in Wei), or by a contract sending it's Ether from it's own balance to another. Enabling smart contracts to move Ether on behalf of another address will allow for a multitude of new oppurtunuties for developers to improve their dapp's user experience, and discover new applications of Ethereum.

A good example are decentralized exchanges. These exchanges rely on assets conforming to the [ERC20](https://eips.ethereum.org/EIPS/eip-20) token standard. This is because this standard enables users to allow smart contracts to move tokens on behalf of the token holder. ERC20 tokens have become an integral part of the ecosystem, and Ether has no way of being compliant. The possible applications for this proposal extend far beyond decentralized exchanges. Every technology that makes use of ERC20 compliant assets stands to benefit. Looking beyond ERC20 tokens, many business models currently in the ecosystem could be improved, and many new applications of Ethereum may become possible.

## Specification

Add a new member function `transferTo` to the solidity `address` type which transfers Ether from that address to a given address. Used like so

```
0x123.transferTo(0xabc, 1 ether) // transfers 1 ether from 0x123 to 0xabc
```

Calling `transferTo` from an address which has not approved the smart contract to do so will result in a `revert`.

In addition, a function `approved(address spender) returns (bool)` would be added so that contracts can know if an address has approved a smart contract or not, example:

```
if (0x123.approved(this)) 0x123.transferTo(this, 1 ether) 
```

### Approving Contracts

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

## Rationale

The rationale behind the addition of the `transferTo` and `approved` members on the `address` type is that there is already a `transfer` function and `balance` member, for transferring Ether from the caller to the address and getting the address's Ether balance respectively. `transferTo` reverts if the address does not have approval, much like its counterpart `transfer` reverts if the address has an insufficient balance (which transferTo does aswell)

Users setting the approval for a contract by calling a method on themselves as if they were a smart contract seemed to be the best way to go about having users set the approval. Another option would be for users to call the approve method on the *contract* they want to approve/unapprove, maybe by adding those as constant functions that every contract has. This of course would bring backwards campatibility issues, but that may not matter as these old contracts wouldn't be using the new transferTo function. Regardless, users need a way to allow/disallow contracts and this way seems the cleanest.

## Backwards Compatibility

This proposal only aims to add functionality to the ecosytem. Contracts created prior to this addition would not be affected.

## Test Cases
*Test cases for an implementation are mandatory for EIPs that are affecting consensus changes. Other EIPs can choose to include links to test cases if applicable.*

## Implementation
*To be discussed*

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
