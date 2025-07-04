# Reserve Interactions

This document explains how the Reserve component interacts with other parts of the Kamino Lending protocol.

## Key Interaction Patterns

```mermaid
graph TD
    R[Reserve] --> |"refers to"| LM[Lending Market]
    R --> |"issues"| CT[Collateral Token]
    R --> |"tracks"| LP[Liquidity Pool]
    R --> |"queries"| PO[Price Oracle]
    R --> |"services"| O[Obligation]
    R --> |"may connect to"| FR[Farming Programs]
```

## User Interactions

### Deposit Flow

When a user deposits liquidity into a reserve:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Deposit Instruction
    participant Reserve
    participant LM as Lending Market
    participant LP as Liquidity Pool
    participant CM as Collateral Mint
    
    User->>Instruction: Request deposit
    Instruction->>LM: Check emergency_mode
    alt emergency_mode enabled
        LM->>Instruction: Reject operation
        Instruction->>User: Return error
    else emergency_mode disabled
        Instruction->>Reserve: Process deposit
        Reserve->>Reserve: Update last_update
        Reserve->>Reserve: Calculate collateral amount
        Reserve->>LP: Transfer liquidity tokens
        Reserve->>CM: Mint collateral tokens
        CM->>User: Receive collateral tokens
        Reserve->>Reserve: Update liquidity and collateral
        Reserve->>User: Complete operation
    end
```

### Withdraw Flow

When a user withdraws from a reserve:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Withdraw Instruction
    participant Reserve
    participant LM as Lending Market
    participant LP as Liquidity Pool
    participant CM as Collateral Mint
    
    User->>Instruction: Request withdrawal
    Instruction->>Reserve: Validate withdrawal
    Reserve->>Reserve: Update last_update
    Reserve->>Reserve: Calculate liquidity amount
    alt Insufficient liquidity
        Reserve->>User: Return error
    else Sufficient liquidity
        Reserve->>CM: Burn collateral tokens
        User->>CM: Transfer collateral tokens
        Reserve->>LP: Transfer liquidity tokens
        LP->>User: Receive liquidity tokens
        Reserve->>Reserve: Update liquidity and collateral
        Reserve->>User: Complete operation
    end
```

### Borrow Flow

When a user borrows from a reserve:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Borrow Instruction
    participant Reserve
    participant LM as Lending Market
    participant O as Obligation
    participant LP as Liquidity Pool
    
    User->>Instruction: Request borrow
    Instruction->>LM: Check emergency_mode and borrow_disabled
    alt emergency_mode enabled or borrow_disabled
        LM->>Instruction: Reject operation
        Instruction->>User: Return error
    else allowed to borrow
        Instruction->>Reserve: Process borrow
        Reserve->>Reserve: Update last_update and accrue interest
        Reserve->>Reserve: Calculate fees
        Reserve->>O: Update obligation borrowed amount
        Reserve->>LP: Transfer liquidity tokens to user
        Reserve->>Reserve: Update borrowed and available amounts
        Reserve->>User: Complete operation
    end
```

### Repay Flow

When a user repays a loan:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Repay Instruction
    participant Reserve
    participant O as Obligation
    participant LP as Liquidity Pool
    
    User->>Instruction: Request repay
    Instruction->>Reserve: Process repay
    Reserve->>Reserve: Update last_update and accrue interest
    Reserve->>Reserve: Calculate repayment amount
    User->>LP: Transfer liquidity tokens
    Reserve->>O: Update obligation borrowed amount
    Reserve->>Reserve: Update borrowed and available amounts
    Reserve->>User: Complete operation
```

## Lending Market Interactions

The Reserve interacts with the Lending Market in these ways:

### 1. Initialization

When a new Reserve is created:
- The Lending Market owner must authorize the creation
- The Reserve stores a reference to its parent Lending Market
- The Reserve inherits certain global parameters from the Lending Market

### 2. Parameter Verification

During operations, the Reserve:
- Checks the Lending Market's emergency mode status
- Validates that borrowing is not disabled globally
- Applies the Lending Market's referral fee settings
- May apply Elevation Group settings from the Lending Market

### 3. Risk Parameter Updates

When risk parameters are updated:
- The Lending Market owner typically initiates the change
- The Reserve applies the new parameters to its configuration
- Existing and new loans are affected by the parameter changes

## Obligation Interactions

Reserves interact with Obligations in several important ways:

### 1. Collateral Management

When an Obligation uses a Reserve's collateral:
- The Obligation tracks the collateral deposit amount
- The Reserve provides the collateral exchange rate
- The Reserve's collateral valuation affects the Obligation's borrow limit

```mermaid
sequenceDiagram
    participant O as Obligation
    participant R as Reserve
    participant PO as Price Oracle
    
    O->>R: Request collateral valuation
    R->>R: Calculate collateral exchange rate
    R->>PO: Get token price
    PO->>R: Return current price
    R->>R: Calculate value = amount * exchange_rate * price
    R->>O: Return collateral value
```

### 2. Loan Servicing

When an Obligation borrows from a Reserve:
- The Reserve provides the liquidity tokens
- The Obligation records the borrowed amount
- The Reserve tracks the total borrowed amount
- Interest accrues on the borrowed amount

### 3. Health Calculation

During Obligation health checks:
- The Reserve provides current token prices
- The Reserve supplies liquidation thresholds
- The Reserve's loan-to-value parameters affect liquidation risk

## Oracle Interactions

Reserves rely heavily on price oracles:

```mermaid
sequenceDiagram
    participant R as Reserve
    participant PO as Price Oracle
    participant SO as Switchboard Oracle
    participant PYO as Pyth Oracle
    
    R->>PO: Request price refresh
    PO->>SO: Try getting price
    alt Switchboard price available and fresh
        SO->>PO: Return price
    else Switchboard unavailable
        PO->>PYO: Try getting price
        PYO->>PO: Return price
    end
    PO->>R: Return validated price
    R->>R: Update calculations with new price
```

The Reserve:
- Regularly refreshes price data for accurate valuations
- Validates price freshness to avoid stale data
- May use multiple oracle sources for redundancy
- Uses price data for all value calculations

## Farm Program Interactions

If farming programs are connected:

```mermaid
sequenceDiagram
    participant R as Reserve
    participant FC as Farm Collateral
    participant FD as Farm Debt
    participant LP as Liquidity Provider
    participant BO as Borrower
    
    LP->>R: Deposit liquidity
    R->>FC: Stake collateral tokens
    FC->>LP: Accrue farming rewards
    
    BO->>R: Borrow liquidity
    R->>FD: Register borrower position
    FD->>BO: Accrue debt-side rewards
```

The Reserve may:
1. Connect with a collateral farm to provide yield to depositors
2. Connect with a debt farm to provide incentives to borrowers
3. Track farming positions alongside lending positions
4. Route rewards to appropriate users

## Interest Rate Updates

Interest rates are dynamically updated based on reserve utilization:

```mermaid
sequenceDiagram
    participant Any as Any Operation
    participant R as Reserve
    
    Any->>R: Trigger operation
    R->>R: Check time since last update
    R->>R: Calculate current utilization
    R->>R: Determine new borrow rate
    R->>R: Apply rate to outstanding debt
    R->>R: Update cumulative borrow rate
    R->>R: Record update timestamp
```

This happens:
- On every operation that affects the reserve
- When the reserve is explicitly refreshed
- Before any borrow or liquidation calculation

## Fee Collection and Distribution

The Reserve manages several types of fees:

```mermaid
graph TD
    R[Reserve] --> |"collects"| BF[Borrow Fees]
    R --> |"collects"| FL[Flash Loan Fees]
    R --> |"collects"| RF[Redemption Fees]
    
    BF --> |"distributes to"| P[Protocol Treasury]
    BF --> |"distributes to"| REF[Referrer]
    
    FL --> |"distributes to"| P
    FL --> |"distributes to"| REF
    
    RF --> |"distributes to"| P
```

When fees are collected:
1. The fee amount is calculated based on the operation
2. A portion goes to the protocol treasury
3. A portion may go to referrers
4. Fees accumulate until explicitly claimed

## Reserve as a State Controller

The Reserve serves as a state controller for several important protocol aspects:

1. **Liquidity Management**: Controls the flow of tokens in and out of the protocol
2. **Exchange Rate Tracking**: Maintains the relationship between collateral and underlying tokens
3. **Interest Accrual**: Ensures borrowed amounts grow appropriately over time
4. **Risk Parameter Application**: Applies risk settings to all operations
5. **Price Integration**: Bridges between market prices and protocol valuations

Understanding these interactions is essential for comprehending how the Reserve integrates with the broader protocol ecosystem.
