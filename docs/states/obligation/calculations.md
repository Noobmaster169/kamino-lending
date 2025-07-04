# Obligation Calculations

This document provides detailed explanations of the key calculations performed by Obligation objects in the Kamino Lending protocol.

## Position Value Calculations

### Deposited Collateral Value

The total value of all collateral deposits is calculated by summing the market values of all deposits:

```
Deposited Value = Sum(Deposit₁ Market Value + Deposit₂ Market Value + ... + Depositₙ Market Value)
```

Each deposit's market value is calculated as:

```
Deposit Market Value = Deposit Amount * Collateral Exchange Rate * Token Price
```

Where:
- Deposit Amount: Number of collateral tokens deposited
- Collateral Exchange Rate: Rate at which collateral tokens convert to underlying tokens
- Token Price: Current market price of the underlying token

#### Example Calculation

With the following deposits:
1. 100 cSOL worth $25 each = $2,500
2. 1,000 cUSDC worth $1.05 each = $1,050

```
Deposited Value = $2,500 + $1,050 = $3,550
```

### Borrowed Value

The total value of all borrows is calculated by summing the market values of all borrows:

```
Borrowed Value = Sum(Borrow₁ Market Value + Borrow₂ Market Value + ... + Borrowₙ Market Value)
```

Each borrow's market value is calculated as:

```
Borrow Market Value = Borrowed Amount * Token Price
```

Where:
- Borrowed Amount: Number of tokens borrowed plus accrued interest
- Token Price: Current market price of the borrowed token

#### Example Calculation

With the following borrows:
1. 1.5 SOL borrowed at $100 each = $150
2. 1,000 USDC borrowed at $1 each = $1,000

```
Borrowed Value = $150 + $1,000 = $1,150
```

## Loan-to-Value Calculations

### Current Loan-to-Value (LTV)

The current LTV represents the ratio of borrowed value to deposited value:

```
Current LTV = Borrowed Value / Deposited Value
```

#### Example Calculation

With the values from above:
```
Current LTV = $1,150 / $3,550 = 0.324 = 32.4%
```

### Weighted Average Loan-to-Value Ratio

The allowed borrow value is based on a weighted average of deposit values and their respective LTV limits:

```
Weighted Avg LTV = Sum(Deposit₁ Value * LTV₁ + Deposit₂ Value * LTV₂ + ... + Depositₙ Value * LTVₙ) / Deposited Value
```

Where:
- Depositₙ Value: Market value of deposit n
- LTVₙ: Loan-to-Value ratio for the reserve of deposit n

#### Example Calculation

With the following:
1. $2,500 SOL deposit with 75% LTV
2. $1,050 USDC deposit with 85% LTV

```
Weighted Avg LTV = ($2,500 * 0.75 + $1,050 * 0.85) / $3,550
                  = ($1,875 + $892.50) / $3,550
                  = $2,767.50 / $3,550
                  = 0.779 = 77.9%
```

### Allowed Borrow Value

The maximum value a user can borrow based on their collateral:

```
Allowed Borrow Value = Sum(Deposit₁ Value * LTV₁ + Deposit₂ Value * LTV₂ + ... + Depositₙ Value * LTVₙ)
```

Using the example above:
```
Allowed Borrow Value = $1,875 + $892.50 = $2,767.50
```

## Liquidation Threshold Calculations

### Weighted Average Liquidation Threshold

Similar to the weighted average LTV, but using liquidation thresholds instead:

```
Weighted Avg Liquidation Threshold = Sum(Deposit₁ Value * LT₁ + Deposit₂ Value * LT₂ + ... + Depositₙ Value * LTₙ) / Deposited Value
```

Where:
- Depositₙ Value: Market value of deposit n
- LTₙ: Liquidation Threshold for the reserve of deposit n

#### Example Calculation

With the following:
1. $2,500 SOL deposit with 80% liquidation threshold
2. $1,050 USDC deposit with 90% liquidation threshold

```
Weighted Avg Liquidation Threshold = ($2,500 * 0.80 + $1,050 * 0.90) / $3,550
                                   = ($2,000 + $945) / $3,550
                                   = $2,945 / $3,550
                                   = 0.829 = 82.9%
```

### Unhealthy Borrow Value

The borrowed value at which a position becomes eligible for liquidation:

```
Unhealthy Borrow Value = Sum(Deposit₁ Value * LT₁ + Deposit₂ Value * LT₂ + ... + Depositₙ Value * LTₙ)
```

Using the example above:
```
Unhealthy Borrow Value = $2,000 + $945 = $2,945
```

## Position Health Calculations

### Health Factor

The health factor indicates how close a position is to liquidation:

```
Health Factor = Unhealthy Borrow Value / Borrowed Value
```

A health factor less than 1.0 means the position is eligible for liquidation.

#### Example Calculation

With the values from above:
```
Health Factor = $2,945 / $1,150 = 2.56
```
A health factor of 2.56 means the position is healthy and not at risk of liquidation.

### Net Value

The net value represents the equity value of the position:

```
Net Value = Deposited Value - Borrowed Value
```

#### Example Calculation

With the values from above:
```
Net Value = $3,550 - $1,150 = $2,400
```

## Interest Accrual Calculations

### Borrow Interest Update

When refreshing an obligation, interest is accrued on each borrow:

```
New Borrowed Amount = Original Borrowed Amount * (Current Cumulative Rate / Original Cumulative Rate)
```

Where:
- Original Borrowed Amount: The amount recorded in the obligation
- Current Cumulative Rate: The reserve's current cumulative borrow rate
- Original Cumulative Rate: The rate recorded in the obligation when the borrow was created or last updated

#### Example Calculation

If:
- Original Borrowed Amount = 100 USDC
- Original Cumulative Rate = 1.05
- Current Cumulative Rate = 1.06

```
New Borrowed Amount = 100 * (1.06 / 1.05) = 100 * 1.0095 = 100.95 USDC
```

## Elevation Group Calculations

When an obligation uses an elevation group, special calculations apply:

### Elevated Loan-to-Value

If the borrow reserve and collateral reserve are in the same elevation group:

```
Elevated LTV = Elevation Group LTV (higher than standard LTV)
```

### Elevated Liquidation Threshold

Similarly, for liquidation thresholds:

```
Elevated Liquidation Threshold = Elevation Group Liquidation Threshold (higher than standard threshold)
```

#### Example Calculation

With SOL and USDC in an elevation group:
- Standard SOL LTV: 75%
- Elevated SOL LTV (when borrowing USDC): 85%
- Standard SOL Liquidation Threshold: 80%
- Elevated SOL Liquidation Threshold: 90%

For borrowing USDC against SOL in the elevation group:
```
Allowed Borrow Value = $2,500 * 0.85 = $2,125 (vs $1,875 without elevation)
Unhealthy Borrow Value = $2,500 * 0.90 = $2,250 (vs $2,000 without elevation)
```

## Liquidation Calculations

### Maximum Liquidation Amount

When an obligation is eligible for liquidation, the maximum amount that can be liquidated is:

```
Max Liquidation Amount = Min(
    Borrowed Amount,
    Borrowed Amount * Liquidation Close Factor,
    Max Liquidatable Debt Market Value
)
```

Where:
- Liquidation Close Factor: Percentage of debt that can be closed (e.g., 50%)
- Max Liquidatable Debt Market Value: Protocol-wide maximum for a single liquidation

#### Example Calculation

If:
- Borrowed Amount = $1,000 USDC
- Liquidation Close Factor = 50%
- Max Liquidatable Debt Market Value = $10,000

```
Max Liquidation Amount = Min($1,000, $1,000 * 0.50, $10,000) = $500
```

### Liquidation Exchange

When liquidation occurs, the liquidator repays borrowed tokens and receives collateral tokens:

```
Collateral Received = (Repaid Amount * (1 + Liquidation Bonus)) / Collateral Price
```

Where:
- Repaid Amount: Amount of borrowed tokens repaid
- Liquidation Bonus: Bonus percentage as incentive for liquidators
- Collateral Price: Current price of the collateral token

#### Example Calculation

If:
- Repaid Amount = 500 USDC
- Liquidation Bonus = 10%
- SOL Price = $100

```
Collateral Received = (500 * 1.10) / 100 = 550 / 100 = 5.5 SOL
```
The liquidator receives 5.5 SOL worth $550 for repaying $500 of debt.

## Auto-Deleveraging Calculations

When auto-deleveraging is triggered, the protocol calculates:

```
Amount to Deleverage = Min(
    Amount to Restore Health,
    Borrowed Amount,
    Available Liquidity
)
```

Where:
- Amount to Restore Health: Amount needed to bring health factor to safe level
- Borrowed Amount: Total amount borrowed
- Available Liquidity: Amount available in the reserve for repayment

### Example Calculation

If:
- Borrowed Value = $1,500
- Unhealthy Borrow Value = $1,200 (position is unhealthy)
- Target Health Factor = 1.2
- Available Liquidity = $500

```
Amount to Restore Health = $1,500 - ($1,200 / 1.2) = $1,500 - $1,000 = $500
Amount to Deleverage = Min($500, $1,500, $500) = $500
```

## Scaling and Precision

Many obligation calculations use high-precision arithmetic to avoid rounding errors:

- Market values use `BigFractionBytes` for 128+ bits of precision
- Borrowed amounts use high precision to accurately track accrued interest
- Cumulative borrow rates maintain precision over time to ensure accurate interest calculations

For example, a borrowed amount might be stored as:
```
1,050,000,000,000,000,000 / 10^18 = 1.05 tokens
```

This high precision ensures that even small interest accruals are accurately tracked over time.

## Impact of Price Changes

All obligation calculations are highly sensitive to price changes:

1. **Price Increases in Collateral**:
   - Increases deposited value
   - Improves health factor
   - Increases borrowing capacity
   
2. **Price Decreases in Collateral**:
   - Decreases deposited value
   - Worsens health factor
   - May trigger liquidation

3. **Price Increases in Borrowed Assets**:
   - Increases borrowed value
   - Worsens health factor
   - May trigger liquidation
   
4. **Price Decreases in Borrowed Assets**:
   - Decreases borrowed value
   - Improves health factor
   - Increases borrowing capacity

This price sensitivity is why the protocol requires frequent refreshing of obligation data and oracle prices.
