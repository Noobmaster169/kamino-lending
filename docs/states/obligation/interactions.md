# Obligation Interactions

This document explains how the Obligation component interacts with other parts of the Kamino Lending protocol.

## Key Interaction Patterns

```mermaid
graph TD
    O[Obligation] --> |"owned by"| User[User Wallet]
    O --> |"belongs to"| LM[Lending Market]
    O --> |"deposits into"| R1[Reserve 1]
    O --> |"borrows from"| R2[Reserve 2]
    O --> |"may use"| EG[Elevation Groups]
    R1 --> |"provides collateral value"| O
    R2 --> |"tracks debt"| O
    PO[Price Oracle] --> |"updates values"| O
```

## User Interactions

### Deposit Collateral Flow

When a user deposits collateral into their obligation:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Deposit Instruction
    participant O as Obligation
    participant R as Reserve
    participant CT as Collateral Token
    
    User->>Instruction: Request deposit
    Instruction->>R: Refresh reserve
    R->>R: Update prices and interest
    Instruction->>O: Add collateral deposit
    alt New deposit
        O->>O: Find or add collateral to deposits
    else Existing deposit
        O->>O: Update existing deposit amount
    end
    User->>CT: Transfer collateral tokens
    CT->>Instruction: Receive tokens
    Instruction->>O: Update obligation
    O->>O: Update market values
    O->>User: Complete operation
```

### Borrow Flow

When a user borrows against their obligation:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Borrow Instruction
    participant O as Obligation
    participant R as Reserve
    participant LM as Lending Market
    
    User->>Instruction: Request borrow
    Instruction->>LM: Check borrow permission
    Instruction->>R: Refresh reserve
    R->>R: Update prices and interest
    Instruction->>O: Refresh obligation
    O->>O: Update deposit and borrow values
    Instruction->>O: Calculate allowed borrow value
    alt Sufficient collateral
        O->>O: Add or update borrow record
        R->>R: Transfer liquidity to user
        R->>User: Receive borrowed tokens
        O->>O: Update borrow details
    else Insufficient collateral
        Instruction->>User: Return error
    end
```

### Repay Flow

When a user repays a loan:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Repay Instruction
    participant O as Obligation
    participant R as Reserve
    
    User->>Instruction: Request repay
    Instruction->>R: Refresh reserve
    R->>R: Update prices and interest
    Instruction->>O: Refresh obligation
    O->>O: Update borrowed amounts
    User->>R: Transfer repayment tokens
    Instruction->>O: Calculate repayment impact
    O->>O: Update or remove borrow record
    O->>User: Complete operation
```

### Withdraw Collateral Flow

When a user withdraws collateral:

```mermaid
sequenceDiagram
    participant User
    participant Instruction as Withdraw Instruction
    participant O as Obligation
    participant R as Reserve
    participant CT as Collateral Token
    
    User->>Instruction: Request withdrawal
    Instruction->>R: Refresh reserve
    R->>R: Update prices and interest
    Instruction->>O: Refresh obligation
    O->>O: Update deposit and borrow values
    Instruction->>O: Calculate if withdrawal is safe
    alt Safe withdrawal
        O->>O: Update or remove deposit record
        CT->>User: Transfer collateral tokens
    else Unsafe withdrawal
        Instruction->>User: Return error
    end
```

## Reserve Interactions

The Obligation interacts with Reserves in these ways:

### 1. Collateral Management

When a user deposits collateral:
- The Reserve provides collateral tokens
- The Obligation tracks these tokens as collateral
- The Reserve's collateral exchange rate affects valuation
- The Reserve's LTV and liquidation threshold parameters apply

```mermaid
sequenceDiagram
    participant O as Obligation
    participant R as Reserve
    
    O->>R: Request collateral deposit
    R->>O: Provide collateral token info
    O->>O: Store deposit with reserve reference
    O->>R: Request collateral valuation
    R->>R: Calculate exchange rate
    R->>O: Return valuation data
    O->>O: Update market values
```

### 2. Borrowing

When a user borrows from a reserve:
- The Obligation records the borrow details
- The Reserve provides the liquidity
- The Reserve's cumulative borrow rate is recorded for interest tracking
- The Obligation uses Reserve risk parameters to calculate borrowing capacity

```mermaid
sequenceDiagram
    participant O as Obligation
    participant R as Reserve
    
    O->>R: Request borrow
    R->>O: Provide current cumulative rate
    O->>O: Store borrow with rate snapshot
    R->>O: Provide borrowed liquidity info
    O->>O: Record borrowed amount
```

### 3. Interest Accrual

During obligation refresh:
- The Obligation compares current Reserve rates with recorded rates
- Interest is calculated based on rate changes
- Borrowed amounts are updated to include accrued interest

```mermaid
sequenceDiagram
    participant Any as Any Operation
    participant O as Obligation
    participant R as Reserve
    
    Any->>O: Refresh obligation
    O->>R: Get current cumulative borrow rate
    R->>O: Return current rate
    O->>O: Compare with stored rate
    O->>O: Calculate accrued interest
    O->>O: Update borrowed amounts
```

## Lending Market Interactions

The Obligation interacts with the Lending Market in these ways:

### 1. Initialization

When an Obligation is created:
- It must reference a valid Lending Market
- It inherits parameters from the Lending Market
- The Lending Market tracks the Obligation as part of its risk management

### 2. Risk Parameter Application

During operations:
- The Obligation uses Lending Market parameters like insolvency thresholds
- Global liquidation parameters from the Lending Market apply to the Obligation
- Elevation Group definitions from the Lending Market may apply

```mermaid
sequenceDiagram
    participant O as Obligation
    participant LM as Lending Market
    
    O->>LM: Request risk parameters
    LM->>O: Provide current risk settings
    O->>O: Apply parameters to calculations
    O->>O: Apply elevation group settings if relevant
```

### 3. Emergency Mode

In emergency situations:
- The Lending Market may enable emergency mode
- This affects what operations are allowed on the Obligation
- Borrowing may be disabled, but repayments still allowed

## Price Oracle Interactions

Obligations rely heavily on price oracles for valuation:

```mermaid
sequenceDiagram
    participant O as Obligation
    participant R as Reserve
    participant PO as Price Oracle
    
    O->>R: Request valuation
    R->>PO: Get current prices
    PO->>R: Return validated prices
    R->>O: Provide price data
    O->>O: Update market values
```

The Obligation:
- Doesn't interact with oracles directly but through Reserves
- Uses current prices to value all deposits and borrows
- Recalculates health metrics whenever prices change
- May become unhealthy due to price movements

## Elevation Group Interactions

If an Obligation is part of an Elevation Group:

```mermaid
graph TD
    O[Obligation] --"joins"--> EG[Elevation Group]
    EG --"allows"--> CR[Collateral Reserves]
    EG --"restricts to"--> DR[Debt Reserve]
    
    O --"can use as collateral"--> CR
    O --"can only borrow from"--> DR
    
    EG --"provides"--> SP[Special Parameters]
    SP --"modifies"--> TV[LTV and Liquidation Thresholds]
    TV --"applies to"--> O
```

Special interactions include:

1. **Group Membership**: The Obligation can only be part of one elevation group at a time
2. **Borrowing Restrictions**: Can only borrow from the designated debt reserve
3. **Collateral Limitations**: May be limited to specific collateral reserves in the group
4. **Special Parameters**: Uses the group's LTV and liquidation thresholds (typically higher)
5. **Isolation**: Cannot mix with assets outside the group in certain ways

## Liquidation Interactions

When an Obligation becomes unhealthy:

```mermaid
sequenceDiagram
    participant Liquidator as Liquidator
    participant Instruction as Liquidate Instruction
    participant O as Obligation
    participant BR as Borrow Reserve
    participant CR as Collateral Reserve
    
    Liquidator->>Instruction: Request liquidation
    Instruction->>O: Check health
    O->>O: Verify unhealthy status
    alt Not Liquidatable
        O->>Liquidator: Return error
    else Liquidatable
        Instruction->>BR: Receive repayment
        Liquidator->>BR: Transfer repayment tokens
        Instruction->>O: Calculate liquidation amount
        O->>O: Update borrow record
        Instruction->>CR: Calculate collateral to seize
        CR->>Liquidator: Transfer collateral tokens
        O->>O: Update collateral deposit
    end
```

The liquidation process:
1. Verifies the Obligation is actually unhealthy
2. Calculates how much debt can be repaid
3. Determines how much collateral the liquidator receives
4. Updates the Obligation's borrow and deposit records
5. May completely remove a borrow or deposit if fully liquidated

## Auto-Deleveraging Interactions

If the protocol has auto-deleveraging enabled:

```mermaid
sequenceDiagram
    participant Protocol as Protocol
    participant O as Obligation
    participant R as Reserve
    
    Protocol->>O: Check health periodically
    alt Healthy
        O->>Protocol: No action needed
    else Unhealthy for extended period
        Protocol->>O: Trigger auto-deleveraging
        O->>R: Calculate deleveraging amount
        R->>R: Use protocol liquidity to repay
        O->>O: Update borrow records
        O->>Protocol: Report deleveraging action
    end
```

This process:
1. Identifies unhealthy obligations that have remained unhealthy beyond the grace period
2. Uses protocol liquidity to repay part of the debt
3. Updates the obligation's borrow records
4. Charges a fee for the service
5. May restore the obligation to health

## Obligation as a User Position Manager

The Obligation serves as the central manager for a user's position:

1. **Position Tracking**: Maintains the complete record of a user's assets and liabilities
2. **Risk Assessment**: Continuously evaluates position health as prices change
3. **Borrowing Capacity**: Calculates how much the user can safely borrow
4. **Liquidation Management**: Determines when positions are at risk
5. **Record Keeping**: Tracks all collateral and debt across multiple reserves

Understanding these interactions is essential for comprehending how users interact with the protocol and how risk is managed at the individual position level.
