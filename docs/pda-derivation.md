# Kamino Lending Program Derived Addresses (PDAs)

This document provides a comprehensive reference for all Program Derived Addresses (PDAs) used in the Kamino Lending protocol. For each account type, we provide the seed structure and examples of how to derive these addresses in TypeScript using the Solana web3.js library.

## PDA Overview

Program Derived Addresses are deterministic addresses derived from a set of seeds and a program ID. They allow programs to create and manage accounts without needing to sign transactions with their private key.

## Constants and Seeds

The Kamino Lending protocol defines the following seed constants:

```rust
pub const LENDING_MARKET_AUTH: &[u8] = b"lma";
pub const RESERVE_LIQ_SUPPLY: &[u8] = b"reserve_liq_supply";
pub const FEE_RECEIVER: &[u8] = b"fee_receiver";
pub const RESERVE_COLL_MINT: &[u8] = b"reserve_coll_mint";
pub const RESERVE_COLL_SUPPLY: &[u8] = b"reserve_coll_supply";
pub const BASE_SEED_REFERRER_TOKEN_STATE: &[u8] = b"referrer_acc";
pub const BASE_SEED_USER_METADATA: &[u8] = b"user_meta";
pub const BASE_SEED_REFERRER_STATE: &[u8] = b"ref_state";
pub const BASE_SEED_SHORT_URL: &[u8] = b"short_url";
pub const GLOBAL_CONFIG_STATE: &[u8] = b"global_config";
```

## PDA Reference Table

| Account Type | Seeds | Description | TypeScript Function |
|-------------|-------|-------------|-------------------|
| Lending Market Authority | `["lma", lending_market]` | Authority PDA for the lending market | `getLendingMarketAuthority(lendingMarket: PublicKey): PublicKey` |
| Global Config | `["global_config"]` | Global configuration state | `getGlobalConfig(): PublicKey` |
| Liquidity Supply Vault | `["reserve_liq_supply", market, mint]` | Reserve's liquidity pool | `getLiquiditySupplyVault(market: PublicKey, mint: PublicKey): PublicKey` |
| Collateral Token Mint | `["reserve_coll_mint", market, mint]` | Reserve's collateral token mint | `getCollateralTokenMint(market: PublicKey, mint: PublicKey): PublicKey` |
| Collateral Supply Vault | `["reserve_coll_supply", market, mint]` | Reserve's collateral token supply | `getCollateralSupplyVault(market: PublicKey, mint: PublicKey): PublicKey` |
| Fee Receiver | `["fee_receiver", market, mint]` | Account receiving reserve fees | `getFeeReceiver(market: PublicKey, mint: PublicKey): PublicKey` |
| Referrer Token State | `["referrer_acc", referrer, reserve]` | Referrer's state for a specific token | `getReferrerTokenState(referrer: PublicKey, reserve: PublicKey): PublicKey` |
| User Metadata | `["user_meta", owner]` | User metadata information | `getUserMetadata(owner: PublicKey): PublicKey` |
| Obligation | `[tag, id, owner, lending_market, seed1, seed2]` | User's lending position | `getObligation(tag: number, id: number, owner: PublicKey, lendingMarket: PublicKey, seed1: PublicKey, seed2: PublicKey): PublicKey` |

## TypeScript Implementation

Here are TypeScript functions to derive the PDAs for the Kamino Lending protocol:

```typescript
import { PublicKey } from '@solana/web3.js';

// The Kamino Lending program ID (replace with actual ID)
const PROGRAM_ID = new PublicKey('KLDP8X1Hi3ybJfZeQD1UjigxHyFbCCGFYahzEJJXqcbY');

/**
 * Derives the lending market authority address for a lending market
 */
export function getLendingMarketAuthority(lendingMarket: PublicKey): PublicKey {
  const [authority] = PublicKey.findProgramAddressSync(
    [Buffer.from('lma'), lendingMarket.toBuffer()],
    PROGRAM_ID
  );
  return authority;
}

/**
 * Derives the global config address
 */
export function getGlobalConfig(): PublicKey {
  const [config] = PublicKey.findProgramAddressSync(
    [Buffer.from('global_config')],
    PROGRAM_ID
  );
  return config;
}

/**
 * Derives the liquidity supply vault address for a reserve
 */
export function getLiquiditySupplyVault(market: PublicKey, mint: PublicKey): PublicKey {
  const [vault] = PublicKey.findProgramAddressSync(
    [Buffer.from('reserve_liq_supply'), market.toBuffer(), mint.toBuffer()],
    PROGRAM_ID
  );
  return vault;
}

/**
 * Derives the collateral token mint address for a reserve
 */
export function getCollateralTokenMint(market: PublicKey, mint: PublicKey): PublicKey {
  const [tokenMint] = PublicKey.findProgramAddressSync(
    [Buffer.from('reserve_coll_mint'), market.toBuffer(), mint.toBuffer()],
    PROGRAM_ID
  );
  return tokenMint;
}

/**
 * Derives the collateral supply vault address for a reserve
 */
export function getCollateralSupplyVault(market: PublicKey, mint: PublicKey): PublicKey {
  const [vault] = PublicKey.findProgramAddressSync(
    [Buffer.from('reserve_coll_supply'), market.toBuffer(), mint.toBuffer()],
    PROGRAM_ID
  );
  return vault;
}

/**
 * Derives the fee receiver address for a reserve
 */
export function getFeeReceiver(market: PublicKey, mint: PublicKey): PublicKey {
  const [receiver] = PublicKey.findProgramAddressSync(
    [Buffer.from('fee_receiver'), market.toBuffer(), mint.toBuffer()],
    PROGRAM_ID
  );
  return receiver;
}

/**
 * Derives the referrer token state address
 */
export function getReferrerTokenState(referrer: PublicKey, reserve: PublicKey): PublicKey {
  const [state] = PublicKey.findProgramAddressSync(
    [Buffer.from('referrer_acc'), referrer.toBuffer(), reserve.toBuffer()],
    PROGRAM_ID
  );
  return state;
}

/**
 * Derives the user metadata address
 */
export function getUserMetadata(owner: PublicKey): PublicKey {
  const [metadata] = PublicKey.findProgramAddressSync(
    [Buffer.from('user_meta'), owner.toBuffer()],
    PROGRAM_ID
  );
  return metadata;
}

/**
 * Derives an obligation address
 * 
 * @param tag - Obligation type tag (e.g., 0 for standard, 1 for margin)
 * @param id - Obligation identifier 
 * @param owner - Owner of the obligation
 * @param lendingMarket - Parent lending market
 * @param seed1 - Additional seed account (varies by obligation type)
 * @param seed2 - Additional seed account (varies by obligation type)
 */
export function getObligation(
  tag: number, 
  id: number, 
  owner: PublicKey, 
  lendingMarket: PublicKey, 
  seed1: PublicKey, 
  seed2: PublicKey
): PublicKey {
  const [obligation] = PublicKey.findProgramAddressSync(
    [
      Buffer.from([tag]), 
      Buffer.from([id]), 
      owner.toBuffer(), 
      lendingMarket.toBuffer(), 
      seed1.toBuffer(), 
      seed2.toBuffer()
    ],
    PROGRAM_ID
  );
  return obligation;
}
```

## Common Use Cases

### Creating a Standard Obligation

To create a standard obligation PDA:

```typescript
const ownerWallet = new PublicKey('...'); // User's wallet
const lendingMarket = new PublicKey('...'); // Lending market address
const seedAccount1 = ownerWallet; // Often the same as the owner for standard obligations
const seedAccount2 = ownerWallet; // Often the same as the owner for standard obligations

// For standard obligations, tag is typically 0
const obligationAddress = getObligation(0, 0, ownerWallet, lendingMarket, seedAccount1, seedAccount2);
```

### Creating a Margin Obligation

For margin obligations, which are specialized for leveraged trading:

```typescript
const ownerWallet = new PublicKey('...'); // User's wallet
const lendingMarket = new PublicKey('...'); // Lending market address
const seedAccount1 = new PublicKey('...'); // Often a different account for margin obligations
const seedAccount2 = new PublicKey('...'); // Often a different account for margin obligations

// For margin obligations, tag is typically 1
const marginObligationAddress = getObligation(1, 0, ownerWallet, lendingMarket, seedAccount1, seedAccount2);
```

### Getting Reserve-Related PDAs

To get all PDAs related to a reserve:

```typescript
const lendingMarket = new PublicKey('...'); // Lending market address
const tokenMint = new PublicKey('...'); // Token mint address (e.g., USDC)

const liquiditySupplyVault = getLiquiditySupplyVault(lendingMarket, tokenMint);
const collateralTokenMint = getCollateralTokenMint(lendingMarket, tokenMint);
const collateralSupplyVault = getCollateralSupplyVault(lendingMarket, tokenMint);
const feeReceiver = getFeeReceiver(lendingMarket, tokenMint);
```

## Notes and Special Considerations

1. **Obligation Seeds**: The obligation PDA derivation uses multiple seeds including tag, ID, and additional seed accounts. The tag differentiates between different types of obligations (standard vs margin), and the ID allows users to have multiple obligations of the same type.

2. **Seed Accounts**: The `seed1_account` and `seed2_account` parameters in the obligation PDA allow for flexibility in how obligations are organized. For standard obligations, these are often the same as the owner's public key, but they can be different for specialized obligations.

3. **Program ID**: The examples use a placeholder program ID. In production, you should use the actual Kamino Lending program ID.

4. **Buffer Handling**: When converting public keys to buffers in TypeScript, it's important to ensure proper byte ordering using the `.toBuffer()` method provided by the PublicKey class.

5. **Tag and ID**: The `tag` and `id` parameters in obligation PDAs are single-byte values (`[tag]` and `[id]` in the seed structure), limiting them to values 0-255.
