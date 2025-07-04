# Kamino Lending Protocol Architecture

## Protocol Overview

The Kamino Lending protocol is a sophisticated decentralized lending platform on Solana that enables users to supply assets as collateral and borrow other assets against this collateral. The protocol implements a pool-based lending model with dynamic interest rates determined by supply and demand.

## Core Components

```mermaid
graph TD
    User([User]) --> |Interacts with| LM(Lending Market)
    LM --> |Contains| R1(Reserve 1)
    LM --> |Contains| R2(Reserve 2)
    LM --> |Contains| R3(Reserve 3...N)
    User --> |Creates| O(Obligation)
    O --> |Deposits collateral into| R1
    O --> |Borrows assets from| R2
    R1 --> |Price data| Oracle(Price Oracle)
    R2 --> |Price data| Oracle
    R3 --> |Price data| Oracle
```

### 1. Lending Market

The Lending Market is the central coordinator for the entire protocol:

- Manages all reserves within the market
- Enforces global protocol parameters
- Handles protocol-wide settings and emergency controls
- Manages elevation groups for special borrowing arrangements

### 2. Reserves

Each Reserve represents a pool for a specific token:

- Tracks liquidity (deposits and borrows)
- Manages collateral token issuance
- Controls token-specific risk parameters
- Handles interest rate calculations
- Determines fees for operations

### 3. Obligations

An Obligation represents a user's position in the protocol:

- Tracks collateral deposits across reserves
- Records borrowed amounts from reserves
- Calculates position health and risk metrics
- Enforces liquidation thresholds
- Manages user-specific risk settings

### 4. External Integrations

The protocol integrates with several external components:

- **Price Oracles**: Provide real-time price data for risk calculations
- **SPL Token Program**: Handles all token transfers and management
- **Farm Programs**: Optional yield farming integrations for rewards

## System Architecture

```mermaid
flowchart TD
    subgraph "User Interfaces"
        UI[Web UI]
        SDK[SDK/API]
    end
    
    subgraph "Kamino Lending Protocol"
        LM[Lending Market]
        R[Reserves]
        O[Obligations]
        
        LM --- R
        LM --- O
        R --- O
    end
    
    subgraph "External Services"
        Oracle[Price Oracles]
        TokenProgram[SPL Token Program]
        FarmPrograms[Farm Programs]
    end
    
    UI --> SDK
    SDK --> LM
    
    LM --> Oracle
    LM --> TokenProgram
    R --> TokenProgram
    R --> FarmPrograms
```

## Key Protocol Mechanisms

### Collateralization and Risk Management

The protocol uses a risk-based system for managing collateralization:

- Each asset has specific risk parameters (LTV, liquidation threshold)
- Positions must maintain sufficient collateralization ratio
- Undercollateralized positions can be liquidated
- Dynamic interest rates adjust based on utilization

### Interest Rate Model

```mermaid
graph LR
    subgraph "Interest Rate Curve"
        U0[0% Utilization] -->|Min Rate| U1[Optimal Utilization]
        U1 -->|Slope 1| U2[100% Utilization]
        
        style U0 fill:#f9f9f9,stroke:#ccc
        style U1 fill:#d9edf7,stroke:#31708f
        style U2 fill:#f2dede,stroke:#a94442
    end
```

The protocol uses a two-slope interest rate model:
- Below optimal utilization: gradually increasing rate from min to optimal
- Above optimal utilization: steeply increasing rate from optimal to max
- Interest accrues continuously to lender balances

### Liquidation Mechanism

When a position becomes undercollateralized:
- Liquidators can repay part of the borrower's debt
- In exchange, they receive the equivalent collateral value plus a bonus
- Liquidation thresholds are set per asset
- Close factors determine how much of a position can be liquidated at once

## Elevation Groups

The protocol supports specialized borrowing arrangements through elevation groups:
- Define specific debt and collateral reserve relationships
- Enforce additional constraints on certain asset combinations
- Allow for specialized risk parameters for specific asset pairs

## Security Features

The protocol implements multiple security mechanisms:
- Emergency mode to pause borrowing
- Configurable deposit and borrow limits per reserve
- Role-based access control for administrative functions
- Socialized loss handling for extreme cases
- Comprehensive error handling for every operation
