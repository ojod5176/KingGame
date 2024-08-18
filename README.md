# ERC20 Token Smart Contract

## Overview

This smart contract implements a basic version of an ERC20 token, which includes essential functionalities like transferring tokens, approving allowances, minting new tokens, and burning tokens. The token is named "Solidity by Example" with the symbol "SOLBYEX" and follows the standard ERC20 structure.

## Features

- **Transfer Tokens**: Allows token holders to transfer tokens to another address.
- **Approve Allowance**: Token holders can approve another address to spend tokens on their behalf.
- **Transfer From**: Allows the approved address to transfer tokens from the holder's account to another account.
- **Mint Tokens**: Allows the creation of new tokens, increasing the total supply.
- **Burn Tokens**: Allows the destruction of tokens, decreasing the total supply.

## Contract Details

### Variables

- `uint public totalSupply`: Tracks the total supply of the token.
- `mapping(address => uint) public balanceOf`: Maps each address to its token balance.
- `mapping(address => mapping(address => uint)) public allowance`: Maps each address to another address's allowed spending amount.
- `string public name = "Solidity by Example"`: The name of the token.
- `string public symbol = "SOLBYEX"`: The symbol of the token.
- `uint8 public decimals = 18`: The number of decimals the token uses (common for ERC20 tokens).

### Events

- `event Transfer(address indexed from, address indexed to, uint value)`: Emitted when a transfer of tokens occurs.
- `event Approval(address indexed owner, address indexed spender, uint value)`: Emitted when an approval is granted for an allowance.

### Functions

#### `transfer(address recipient, uint amount) external returns (bool)`

- Transfers a specified `amount` of tokens from the caller's address to the `recipient` address.
- Emits a `Transfer` event.
- Returns `true` if the transfer is successful.

#### `approve(address spender, uint amount) external returns (bool)`

- Approves `spender` to spend a specified `amount` of tokens on behalf of the caller.
- Emits an `Approval` event.
- Returns `true` if the approval is successful.

#### `transferFrom(address sender, address recipient, uint amount) external returns (bool)`

- Transfers a specified `amount` of tokens from `sender` to `recipient` using an allowance.
- Decreases the allowance of the `msg.sender` by the `amount` transferred.
- Emits a `Transfer` event.
- Returns `true` if the transfer is successful.

#### `mint(uint amount) external`

- Mints a specified `amount` of tokens to the caller's address.
- Increases the `totalSupply` and the caller's balance.
- Emits a `Transfer` event with `from` set to the zero address.

#### `burn(uint amount) external`

- Burns a specified `amount` of tokens from the caller's address.
- Decreases the `totalSupply` and the caller's balance.
- Emits a `Transfer` event with `to` set to the zero address.

## Usage

1. **Deploying the Contract**: Deploy the contract on the Ethereum network. The token name is "Solidity by Example" and the symbol is "SOLBYEX".
2. **Transferring Tokens**: Call the `transfer` function to send tokens to another address.
3. **Approving Allowances**: Use the `approve` function to allow another address to spend tokens on your behalf.
4. **Transferring Using Allowance**: Use the `transferFrom` function to transfer tokens from one address to another using the approved allowance.
5. **Minting Tokens**: Call the `mint` function to create new tokens and increase the total supply.
6. **Burning Tokens**: Call the `burn` function to destroy tokens and decrease the total supply.

## License

This project is licensed under the MIT License.





# Vault Smart Contract

## Overview

The `Vault` smart contract is designed to interact with an ERC20 token, allowing users to deposit tokens into the vault and receive shares representing their stake. Users can also withdraw their tokens by burning these shares. The contract maintains a mapping of each user's balance of shares and tracks the total supply of shares in the vault.

## Features

- **Token Deposit**: Users can deposit an ERC20 token into the vault. In return, they receive shares that represent their proportional ownership of the vault.
- **Token Withdrawal**: Users can withdraw their tokens from the vault by burning their shares. The amount of tokens withdrawn is proportional to the number of shares burned.
- **Minting & Burning**: The contract mints shares when tokens are deposited and burns shares when tokens are withdrawn.

## Contract Details

### Variables

- `IERC20 public immutable token`: The ERC20 token that the vault interacts with.
- `uint public totalSupply`: The total supply of shares in the vault.
- `mapping(address => uint) public balanceOf`: A mapping that stores the balance of shares for each address.

### Constructor

``` js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "./IERC20.sol";
import "./ERC20Token.sol";

contract Vault {
    IERC20 public immutable token;

    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function _mint(address _to, uint _shares) private {
        totalSupply += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint _shares) private {
        totalSupply -= _shares;
        balanceOf[_from] -= _shares;
    }

    function deposit(uint _amount) external {
        /*
        a = amount
        B = balance of token before deposit
        T = total supply
        s = shares to mint

        (T + s) / T = (a + B) / B 

        s = aT / B
        */
        uint shares;
        if (totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply) / token.balanceOf(address(this));
        }

        _mint(msg.sender, shares);
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        /*
        a = amount
        B = balance of token before withdraw
        T = total supply
        s = shares to burn

        (T - s) / T = (B - a) / B 

        a = sB / T
        */
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        _burn(msg.sender, _shares);
        token.transfer(msg.sender, amount);
    }

}


