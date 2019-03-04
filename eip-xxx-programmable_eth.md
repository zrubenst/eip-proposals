---
eip: <to be assigned>
title: Programmable ETH Through Extension Functions
author: Zack Rubenstein (@zrubenst)
discussions-to: *TBD*
status: Draft
type: Standards Track
category: Core
created: 2019-03-04
---

## Simple Summary
Allow smart contracts to transfer Ether on a user's behalf.

## Abstract
Users will be able to allow/disallow smart contracts to transfer their Ether, and smart contracts will be able to transfer Ether from an address if and only if it is allowed to do so. This proposal, outlined in detail in the *specification* section below, aims to add a new function type: `extension`. Functions of this type have the ability to transfer Ether on behalf of any address so long as the owner of said address has approved the smart contract to be able to transfer it's Ether. Users allow and disallow extension functions in specific contracts by calling two new methods, `approve(address _spender)` and `unapprove(address _spender)`, which are called by setting the `to` field in a transaction to the user's own address and the `data` field is encoded the same way that a contract function call would normally be encoded. These functions are only available to the user themself.

## Motivation
The ERC20 Token Standard opened the Ethereum ecosystem to a wide range of applications. As systems and smart contracts have been and are being built around this token standard, arguably the most important "token" is being left behind: Ether. This proposal does not solely aim to make Ether ERC20-compliant, however it would add the basic functionality needed to allow for Ether to be ERC20-compliant, and opens the doors to a multitude of other usecases outside of tokens.

In one word, the motivation for this EIP is **friction**. In the case of decentralized exchanges for example, an ERC20 token `WETH` (Wrapped Ether) was developed that allows users to "wrap" Ether into the WETH token, and "unwrap" WETH back into Ether. So users coming into the ecosystem to trade assets on these WETH-based exchanges are forced to get their hands on Ether, and then wrap it into ERC20-compliant WETH so they can trade it. Even still, users need to keep some Ether unwrapped so that they can pay for any gas fees associated with the exchange and unwrapping their WETH back to Ether.

## Specification

#### Extension Functions
A new function type is to be added: `extension`. This function type acts as an internal function, and therefore cannot be called directly or by other contracts. Within the scope of extension functions is a new globally available variable `client`, which has all of the members of an `address` type as well as a new function, `transferFor(address _to, uint _amount)` which transfers the given amount of Ether from the client to the specified address. Example:

```
contract ExtensionExample {
    function transferOnBehalfOf(address _to, uint _amount) extension returns (bool) {
        client.transferFor(_to, _amount); // transfer Ether from client to specified address
        return true;
    }
    
    function grabAllFrom(address _from) public returns (bool) {
        this(_from).transferOnBehalfOf(msg.sender, _from.balance);
        return true;
    }
}
```

Notice how when the extension function is called from a non-extension function, a reference to `this` is made with the `_from` address passed in as an argument. This allows extension functions to be called, where the `client` will be the address supplied. This will revert if the client address has not approved the smart contract for extension functions.

#### User Approval/Unapproval

The second half of this proposal is enabling users to be able to approve smart contracts to execute extension functions in the context of the user, and revoke a smart contract's approval. This is achieved by adding two new methods, `approve(address _spender)` and `unapprove(address _spender)`, which are called by setting the `to` field in a transaction to the user's own address and the `data` field is encoded the same way that a contract function call would normally be encoded. These functions can only be called by the user themself.

#### Other (Minor) Changes

The `address` type should have two more members in addition to the `transferFor` function:

- `address[] extensions` returns an array of contract addresses which have been approved
- `approved(address _contract) returns (bool)` check if a contract has been approved

The JSON RPC should have additional methods:

- `eth_getExtensions` takes in same data as *eth_getBalance*, returns array of approved contract addresses

## Use Case --- ERC20-Compliant Ether

The perfect demonstration of what this proposal will enable is making Ether ERC20 compliant. Here is an abridged version of the smart contract for this, this code example focuses on the *changes* to the standard ERC20 Token contract.

```
interface ERC20I { /* Full interface for ERC20. Omitted */  }

contract ERC20Ether is ERC20I {
    // Assume full implementation, where the interface methods utlimatley 
    // call internal helpers, like `balanceOf` calls `_balanceOf` and 
    // `transfer` and `transferFrom` both call `_transfer`
    // NOTE: assume any code for ERC20 that is ommitted is implemented,
    // this example's intent is to show off the *key* features of this EIP
    
    function _balanceOf(address _user) internal view returns (uint) {
        return _user.balance; // return address's Ether balance
    }
    
    function _transfer(address _from, address _to, uint _amount) internal returns (bool) {
        require(_amount > 0);
        require(_from != _to);
        this(_from).transferOnBehalfOf(_to, _amount); // call extension function
        return true;
    }
    
    // -----------------------------------
    // Extension Function(s)
    
    function transferOnBehalfOf(address _to, uint _amount) extension returns (bool) {
        require(_amount > 0);
        require(_amount <= client.balance);
        client.transferFor(_to, _amount);
        return true;
    }
}
```

Realistically this contract should have other modifications such as checking if the address has approved the contract in the ERC20 `approve` and `allowance` functions.

`ERC20-Compliant Ether` should be an EIP in-and-of itself that relies on this proposal, but ultimatley there would be an agreed upon contract with a specific address that replaces `WETH` as the go-to Ether token representation, as it would for all intents and purposes **be** Ether (transfer and transferFrom move Ether, and the balanceOf simply returns the Ether balance of the given address).

## Rationale
*TODO: before submission*

## Backwards Compatibility
*TODO: before submission*

## Test Cases
*To be discussed*

## Implementation

*To be discussed*

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
