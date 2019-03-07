---
eip: <to be assigned>
title: ERC20-Compliant Ether
author: Zack Rubenstein (@zrubenst)
discussions-to: *TBD*
status: Draft
type: Standards Track
category: ERC
created: 2019-03-07
requires: XXX (Programmable Ether)
---

## Simple Summary
Make Ether comply with the [ERC20](https://eips.ethereum.org/EIPS/eip-20) token standard.

## Motivation
With the potential implementation of [EIPXXX (Programmable Ether)](https://eips.ethereum.org/EIPS/eip-20), it becomes possible to make Ether ERC20 compliant.

## Specification

EIPXXX would add a new member function `transferTo` to variables of type `address`. This single addition enables the following smart contract to become inter-operable with Ether.

```
/**
 * @title SafeMath
 * @dev see https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/math/SafeMath.sol
 */
library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) return 0;
    uint256 c = a * b;
    require(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b > 0);
    uint256 c = a / b;
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b <= a);
    uint256 c = a - b;
    return c;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    require(c >= a);
    return c;
  }

  function mod(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b != 0);
    return a % b;
  }
}

/**
 * @title ERC20 interface
 * @dev see https://eips.ethereum.org/EIPS/eip-20
 */
interface IERC20 {
  function transfer(address to, uint256 value) external returns (bool);
  function approve(address spender, uint256 value) external returns (bool);
  function transferFrom(address from, address to, uint256 value) external returns (bool);
  function totalSupply() external view returns (uint256);
  function balanceOf(address owner) external view returns (uint256);
  function allowance(address owner, address spender) external view returns (uint256);

  event Transfer(address indexed from, address indexed to, uint256 value);
  event Approval(address indexed owner, address indexed spender, uint256 value);
}


/**
 * @title ERC20-Compliant Ether
 * @dev see https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/token/ERC20/ERC20.sol
 *
 * An ERC20 Token that is mapped one-to-one with Ether. balanceOf returns the Ether balance, 
 * and transfer/transferFrom transfer Ether, not a secondary token (like WETH).
 * No deposits or withdrawals of Ether, because for all intents and purposes, this is Ether.
 */
contract EtherERC20 is IERC20 {
  using SafeMath for uint256;

  mapping (address => uint256) private _balances;
  mapping (address => mapping (address => uint256)) private _allowed;

  // ------------------------------------
  // ERC20 Interface Implementation

  function totalSupply() public view returns (uint256) {
    return 0;
  }

  function balanceOf(address owner) public view returns (uint256) {
    return owner.balance;
  }

  function allowance(address owner, address spender) public view returns (uint256) {
    return _allowed[owner][spender];
  }

  function transfer(address to, uint256 value) public returns (bool) {
    _transfer(msg.sender, to, value);
    return true;
  }

  function approve(address spender, uint256 value) public returns (bool) {
    _approve(msg.sender, spender, value);
    return true;
  }

  function transferFrom(address from, address to, uint256 value) public returns (bool) {
    _approve(from, msg.sender, _allowed[from][msg.sender].sub(value));
    _transfer(from, to, value);
    return true;
  }

  // ------------------------------------
  // Core Functions

  function _transfer(address from, address to, uint256 value) internal {
    from.transferTo(to, value); // will revert if the from address has not approved this contract
    emit Transfer(from, to, value);
  }

  function _approve(address owner, address spender, uint256 value) internal {
    _allowed[owner][spender] = value;
    emit Approval(owner, spender, value);
  }
}
```

An ERC20 Token that is mapped one-to-one with Ether. balanceOf returns the Ether balance, and transfer/transferFrom transfer Ether, not a secondary token (like WETH). No deposits or withdrawals of Ether, because for all intents and purposes, this is Ether.

## Implementation

If [EIPXXX]() is accepted, and the smart contract above is validated/approved, it will be deployed. It's address would be pinned here. Technologies that rely on ERC20 tokens that wish to support Ether would point to the deployed contract's address and treat it as any old ERC20 token, because it *is* an ERC20 token.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
