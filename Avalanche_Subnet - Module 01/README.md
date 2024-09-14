# AVAX PROOF: Advanced Avalanche Course (Metacrafters) 

</br>
<img src="https://cdn.prod.website-files.com/62418210ede7e7f14869de35/6245c9b2c388101db3d950f5_metacrafterslogo-gold.webp">

Avalanche Subnets offer a cutting-edge solution for building and managing decentralized applications on the Avalanche network, marking a significant advancement in blockchain technology.

## Project 

Welcome to this exciting challenge! In this tutorial, you'll learn how to set up your very own EVM subnet on Avalanche, which is an excellent starting point for building your DeFi Kingdom clone. We'll guide you through creating and deploying a custom subnet, and then deploying two smart contracts onto it: an ERC20 token contract and a Vault contract. These contracts will serve as the foundational building blocks for your DeFi Kingdom clone, allowing you to explore the exciting world of decentralized finance and take your first steps towards building your very own DeFi empire. Let's get started!

## Project Instructions 

**Set up your EVM subnet:** You can use our guide and the Avalanche documentation to create a custom EVM subnet on the Avalanche network.

**Define your native currency:** You can set up your own native currency, which can be used as the in-game currency for your DeFi Kingdom clone.

**Connect to Metamask:** Connect you EVM Subnet to metamask, this can be done by following the steps laid out in our guide.

**Deploy basic building blocks:** You can use Solidity and Remix to deploy the basic building blocks of your game, such as smart contracts for battling, exploring, and trading. These contracts will define the game rules, such as liquidity pools, tokens, and more.

## Contract - ERC20 Explanation 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "Polygon";
    string public symbol = "MATIC";
    uint8 public decimals = 18;

		event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(
        address sender,
        address recipient,
        uint amount
    ) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

This contract is a simplified implementation of the ERC20 standard for fungible tokens on the Ethereum blockchain. Let's break down its functionality:

### Key Components:

1. **State Variables**:
   - `totalSupply`: Tracks the total supply of the token.
   - `balanceOf`: A mapping that stores the token balance of each address.
   - `allowance`: A mapping that tracks the amount an owner allows a spender to transfer on their behalf.
   - `name`: The name of the token, here set as "Solidity by Example".
   - `symbol`: The symbol of the token, in this case, "SOLBYEX".
   - `decimals`: The number of decimal places the token supports, set to 18 (which is common for most ERC20 tokens).

2. **Events**:
   - `Transfer`: Emitted when tokens are transferred between addresses.
   - `Approval`: Emitted when an owner approves a spender to transfer tokens on their behalf.

### Functions:

1. **`transfer(address recipient, uint amount)`**: 
   - This function allows a user to transfer tokens from their own account (`msg.sender`) to another (`recipient`).
   - It decreases the sender's balance and increases the recipient's balance.
   - It emits a `Transfer` event after the transfer is successful.

2. **`approve(address spender, uint amount)`**: 
   - This function lets an account owner (`msg.sender`) approve another account (`spender`) to spend a certain amount of tokens on their behalf.
   - The approved amount is stored in the `allowance` mapping.
   - It emits an `Approval` event after approval is successful.

3. **`transferFrom(address sender, address recipient, uint amount)`**: 
   - This function allows a spender to transfer tokens on behalf of the token owner (`sender`), provided they have been approved to do so using the `approve` function.
   - The allowance is decreased accordingly, and the sender’s balance is reduced while the recipient’s balance is increased.
   - A `Transfer` event is emitted once the transfer is complete.

4. **`mint(uint amount)`**:
   - This function allows the contract owner to mint new tokens, increasing both their balance and the `totalSupply`.
   - A `Transfer` event is emitted with `address(0)` as the sender to signify new tokens entering circulation.

5. **`burn(uint amount)`**:
   - This function allows a user to burn (destroy) their tokens, reducing their balance and the `totalSupply`.
   - A `Transfer` event is emitted with `address(0)` as the recipient to signify tokens being removed from circulation.

### How It Works:
- The contract allows for basic ERC20 operations: transferring tokens, approving spending, and minting/burning tokens.
- The `balanceOf` and `allowance` mappings ensure users' token balances and approved spend amounts are tracked correctly.
- Minting and burning allow for dynamic changes to the `totalSupply`, which is common in many token systems.

This contract covers the core functionality of an ERC20 token but can be further extended with additional features, such as security mechanisms like access control.

## Contract - Vault Explanation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function totalSupply() external view returns (uint);

    function balanceOf(address account) external view returns (uint);

    function transfer(address recipient, uint amount) external returns (bool);

    function allowance(address owner, address spender) external view returns (uint);

    function approve(address spender, uint amount) external returns (bool);

    function transferFrom(
        address sender,
        address recipient,
        uint amount
    ) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);
}

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
```

This contract is a vault that allows users to deposit and withdraw ERC20 tokens while keeping track of their shares in the vault. It's a simplified version of how vaults work in decentralized finance (DeFi), where tokens are pooled and users receive shares that represent their stake in the vault. Here's a breakdown of the contract:

### Key Components:

1. **`IERC20` Interface**:
   - This defines the basic ERC20 functions that the vault will interact with:
     - `totalSupply()`, `balanceOf()`, `transfer()`, `allowance()`, `approve()`, and `transferFrom()`.
   - It also defines two events:
     - `Transfer`: Emitted when tokens are transferred between addresses.
     - `Approval`: Emitted when an owner approves a spender to transfer tokens on their behalf.

2. **`Vault` Contract**:
   - The contract interacts with any ERC20-compliant token through the `IERC20` interface. 
   - **`IERC20 public immutable token`**: This holds the ERC20 token that the vault manages, and it is immutable, meaning it is set once during contract creation and cannot be changed.
   - **`totalSupply`**: Tracks the total supply of shares in the vault.
   - **`balanceOf`**: Tracks the share balance of each address.

### Constructor:
- **`constructor(address _token)`**: This sets the address of the ERC20 token the vault will handle, ensuring that the vault is tied to a specific token when deployed.

### Private Functions:

1. **`_mint(address _to, uint _shares)`**:
   - Mints new shares when a user deposits tokens into the vault.
   - Increases the `totalSupply` of shares and the balance of the depositor.

2. **`_burn(address _from, uint _shares)`**:
   - Burns shares when a user withdraws tokens.
   - Reduces the `totalSupply` of shares and the balance of the withdrawer.

### Public Functions:

1. **`deposit(uint _amount)`**:
   - Allows a user to deposit ERC20 tokens into the vault.
   - The user receives a proportional amount of shares based on their deposit.
   
   #### Share Calculation:
   - If this is the first deposit (i.e., `totalSupply == 0`), the number of shares equals the amount of tokens deposited.
   - Otherwise, shares are calculated based on the existing vault token balance and total supply. The formula:
     - `shares = (_amount * totalSupply) / token.balanceOf(address(this))`
     - This ensures that the shares minted are proportional to the amount of tokens the user is adding to the vault relative to the existing balance.

   - The `_mint` function is called to allocate shares to the user, and the tokens are transferred from the user to the vault using `transferFrom`.

2. **`withdraw(uint _shares)`**:
   - Allows a user to withdraw tokens from the vault by burning their shares.
   
   #### Token Withdrawal Calculation:
   - The amount of tokens to be withdrawn is proportional to the user's share in the vault:
     - `amount = (_shares * token.balanceOf(address(this))) / totalSupply`
     - This ensures the user gets back a proportional amount of tokens relative to their share in the total supply.

   - The `_burn` function is called to destroy the user’s shares, and the corresponding tokens are transferred from the vault back to the user.

### How It Works:
- **Deposits**: Users deposit ERC20 tokens into the vault, and in return, they receive shares representing their portion of the vault's assets. The share amount is calculated based on how much they deposit compared to the current token balance in the vault.
- **Withdrawals**: Users can redeem their shares for tokens, which are withdrawn proportionally to the number of shares they own. When withdrawing, the user’s shares are burned, reducing their stake in the vault.

## Course Overview 

<img width="610" src="https://github.com/user-attachments/assets/f1fee5d5-afb4-47bd-892c-3469c24ce1c5">


In this course, you'll explore the latest Avalanche features, pushing the boundaries of what's possible. 

You'll begin by creating a custom subnet from scratch using Go or Rust. Then, deploy an EVM-based subnet using a pre-built version from Ava Labs. Beyond creating and deploying a custom subnet, you'll also deploy two smart contracts on it: an ERC20 token contract and a Vault contract.

After mastering subnets, you'll move on to HyperSDK, a powerful framework that lets you easily launch your own optimized subnet (Hyperchain) on Avalanche.


