# Kamino Lending Protocol Instructions

## Overview

Instructions in the Kamino Lending protocol are the executable operations that allow users and administrators to interact with the protocol. Each instruction performs a specific action that modifies the state of the protocol according to strictly defined rules.

## Instruction Categories

The instructions in the Kamino Lending protocol can be organized into several functional categories:

### Market Administration

These instructions manage the creation and configuration of lending markets:

- [Initialize Lending Market](./market-admin/init-lending-market.md): Creates a new lending market
- [Set Lending Market Owner](./market-admin/set-lending-market-owner.md): Transfers ownership of a lending market
- [Set Lending Market Operating Mode](./market-admin/set-lending-market-operating-mode.md): Configures emergency modes and feature flags
- [Update Lending Market](./market-admin/update-lending-market.md): Updates market parameters
- [Set Lending Market Elevation Group](./market-admin/set-lending-market-elevation-group.md): Configures elevation groups

### Reserve Management

These instructions manage the creation and configuration of reserves:

- [Initialize Reserve](./reserve-admin/init-reserve.md): Creates a new reserve for a specific token
- [Update Reserve Config](./reserve-admin/update-reserve-config.md): Modifies reserve risk parameters
- [Add Reserve Liquidity](./reserve-admin/add-reserve-liquidity.md): Adds initial liquidity to a reserve
- [Withdraw Fees](./reserve-admin/withdraw-fees.md): Withdraws protocol fees from a reserve

### User Deposit Operations

These instructions allow users to deposit assets and manage their collateral:

- [Deposit Reserve Liquidity](./user-deposit/deposit-reserve-liquidity.md): Deposits tokens into a reserve
- [Deposit Reserve Liquidity and Obligation Collateral](./user-deposit/deposit-reserve-liquidity-and-obligation-collateral.md): Deposits and adds as collateral in one step
- [Withdraw Obligation Collateral](./user-deposit/withdraw-obligation-collateral.md): Withdraws collateral from an obligation
- [Withdraw Reserve Liquidity](./user-deposit/withdraw-reserve-liquidity.md): Withdraws tokens from a reserve
- [Redeem Reserve Collateral](./user-deposit/redeem-reserve-collateral.md): Converts collateral tokens back to base tokens

### User Borrowing Operations

These instructions allow users to borrow assets and manage their debt:

- [Initialize Obligation](./user-borrow/init-obligation.md): Creates a new obligation for a user
- [Refresh Obligation](./user-borrow/refresh-obligation.md): Updates an obligation with current prices and interest
- [Borrow Obligation Liquidity](./user-borrow/borrow-obligation-liquidity.md): Borrows tokens against collateral
- [Repay Obligation Liquidity](./user-borrow/repay-obligation-liquidity.md): Repays borrowed tokens

### Liquidation Operations

These instructions handle liquidation of unhealthy positions:

- [Liquidate Obligation](./liquidation/liquidate-obligation.md): Liquidates an unhealthy obligation
- [Auto Deleverage](./liquidation/auto-deleverage.md): Automatically reduces leverage on risky positions

### Flash Loans

These instructions enable flash loans (atomic borrowing and repaying):

- [Flash Borrow Reserve Liquidity](./flash-loan/flash-borrow-reserve-liquidity.md): Borrows tokens for a flash loan
- [Flash Repay Reserve Liquidity](./flash-loan/flash-repay-reserve-liquidity.md): Repays a flash loan

### Farming and Rewards

These instructions manage farming programs integration:

- [Update Reserve Config with Farms](./farming/update-reserve-config-with-farms.md): Configures farming for a reserve
- [Register Farming and Supply](./farming/register-farming-and-supply.md): Registers a user for farming rewards
- [Withdraw Farming Rewards](./farming/withdraw-farming-rewards.md): Claims farming rewards

### Obligation Orders

These instructions enable conditional orders on obligations:

- [Create Obligation Order](./orders/create-obligation-order.md): Creates an automated order
- [Execute Obligation Order](./orders/execute-obligation-order.md): Executes a previously created order
- [Delete Obligation Order](./orders/delete-obligation-order.md): Cancels an existing order

## Instruction Structure

Each instruction in the protocol follows a similar pattern:

1. **Account Validation**: Verifies that all required accounts are provided and have proper permissions
2. **Parameter Validation**: Checks that all input parameters are valid
3. **State Verification**: Ensures the current protocol state allows the operation
4. **State Updates**: Modifies protocol state according to the instruction's logic
5. **Token Transfers**: Performs any required token transfers
6. **Event Emission**: Emits events for off-chain monitoring

## Common Account Types

Most instructions require some combination of these accounts:

- **Program Accounts**: Accounts owned by the lending program (LendingMarket, Reserve, Obligation)
- **User Accounts**: User wallets that sign transactions
- **Token Accounts**: SPL Token accounts for liquidity and collateral
- **System Accounts**: System program, rent sysvar, token program, etc.
- **Oracle Accounts**: Price feed accounts from oracles like Pyth or Switchboard

## Permissions Model

Instructions follow a strict permissions model:

- **Admin Operations**: Can only be performed by the lending market owner
- **User Operations**: Can be performed by any user on their own accounts
- **Global Constraints**: All operations must respect protocol-wide constraints
- **Reserve-Specific Rules**: Each reserve may have unique restrictions
- **Obligation Health**: Operations affecting obligations must maintain position health

## Error Handling

Instructions can fail for various reasons:

- **Invalid Parameters**: Input values outside allowed ranges
- **Insufficient Funds**: Not enough tokens for the requested operation
- **Health Violations**: Operations that would make positions unhealthy
- **Market Constraints**: Violations of market-wide restrictions
- **Emergency Mode**: Operations restricted during emergency conditions

Each instruction documentation explains the specific errors that can occur during that operation.

## Understanding the Documentation

The following pages provide detailed documentation for each instruction. Each page includes:

- **Purpose**: What the instruction does and when it's used
- **Required Accounts**: What accounts must be provided
- **Parameters**: What input values are required
- **Process**: Step-by-step explanation of the execution flow
- **Calculations**: Any important calculations performed
- **Constraints**: Rules and restrictions that apply
- **Error Cases**: Conditions that can cause the instruction to fail
- **Example Flow**: Visual representation of the operation

Use the navigation links above to explore specific instructions.
