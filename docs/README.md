# Kamino Lending Protocol Documentation

## Overview

Kamino Lending is a decentralized lending protocol built on the Solana blockchain. This protocol enables users to:

- Deposit assets as collateral
- Borrow other assets against their collateral
- Earn interest on deposited assets
- Participate in liquidation processes

This documentation is designed to provide a comprehensive understanding of the protocol without requiring code reading.

## Documentation Structure

### Architecture
- [Protocol Overview](./architecture/overview.md) - High-level architecture and key components
- [Protocol Flows](./architecture/protocol-flow.md) - Main user interaction flows

### Core Protocol States
- [Lending Market](./states/lending-market/overview.md) - The central marketplace that connects all reserves
- [Reserve](./states/reserve/overview.md) - Individual asset pools for depositing and borrowing
- [Obligation](./states/obligation/overview.md) - User loan positions with collateral and debt

### Protocol Instructions
Instructions are organized by their functional area:

#### Market Management
- Initialization and configuration of the lending market

#### Reserve Operations
- Deposit and withdrawal operations
- Interest accrual
- Reserve configuration

#### Obligation Operations
- Creating and managing loan positions
- Borrowing and repayment operations

#### Risk Management
- Price refreshes
- Liquidation operations
- Emergency actions

## Key Concepts

### Lending Market
The central entity that coordinates all activity, manages reserves, and enforces protocol parameters.

### Reserves
Individual pools for each supported token, with unique parameters for risk management.

### Obligations
User positions that track collateral deposits and outstanding loans.

### Collateralization
All loans must be over-collateralized, with liquidation occurring if collateral value falls too low.

### Interest Rates
Dynamic rates that adjust based on utilization to balance supply and demand.

## Visual Learning

Throughout the documentation, you'll find detailed diagrams explaining:
- Data structures and state relationships
- Transaction flows and processes
- Risk calculation formulas
- State transitions

Start exploring the documentation by visiting the [Protocol Overview](./architecture/overview.md).
