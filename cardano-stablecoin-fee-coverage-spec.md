# Cardano Stablecoin Fee & minADA Coverage Specification

**Version:** 1.0  
**Date:** 2026-02-08  
**Author:** Arlo (NeuralSpark)  
**Status:** Draft

## Executive Summary

This specification defines how Begin Wallet can offer a "true stablecoin UX" on Cardano, where users can transact using only stablecoins (USDM, DJED, iUSD) without holding ADA directly. Begin acts as a liquidity layer, covering both transaction fees and minADA requirements in exchange for stablecoin payments.

## Problem Statement

### The minADA Challenge

Cardano requires every UTXO to contain a minimum amount of ADA based on its size. This creates friction for stablecoin users:

| UTXO Contents | Approximate minADA Required |
|---------------|----------------------------|
| ADA only | 1.0 ADA |
| 1 token (short name) | 1.4 ADA |
| 1 token (32-char name) | 1.56 ADA |
| Multiple tokens | 1.5-2.5+ ADA |

**Source:** [Cardano Ledger Documentation](https://cardano-ledger.readthedocs.io/en/latest/explanations/min-utxo-mary.html)

### The Splitting Problem

When a user splits a token bundle, new UTXOs are created, each requiring minADA:

```
BEFORE (1 UTXO):
└── 10 USDM + 1.5 ADA (minADA)

SEND 1 USDM (creates 2 UTXOs):
├── Recipient: 1 USDM + 1.5 ADA (minADA)
└── Change: 9 USDM + 1.5 ADA (minADA) ← WHERE DOES THIS COME FROM?
```

**Without additional ADA, users cannot split their token holdings.**

## Solution: Begin as Liquidity Layer

Begin covers both transaction fees AND minADA requirements in exchange for stablecoin payments.

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER'S WALLET                            │
│  Assets: 10 USDM + 1.5 ADA (minADA from when received)          │
│  Wants to: Send 1 USDM to Bob                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BEGIN FEE COVERAGE SERVICE                   │
│                                                                 │
│  1. User creates partial tx: Send 1 USDM to Bob                 │
│  2. User adds: Send 1.80 USDM to Begin Pool                     │
│  3. User signs their inputs                                     │
│  4. Begin validates & calculates coverage needed                │
│  5. Begin injects ADA from pool:                                │
│     - 0.20 ADA (tx fee)                                         │
│     - 1.50 ADA (minADA for change UTXO)                         │
│  6. Begin signs as fee payer & broadcasts                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     RESULT ON-CHAIN                             │
│                                                                 │
│  Bob receives: 1 USDM + 1.5 ADA                                 │
│  User keeps: 9 USDM + 1.5 ADA (Begin-provided) - 1.80 USDM      │
│            = 8.20 USDM + 1.5 ADA                                │
│  Begin: +1.80 USDM, -1.70 ADA                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Integration with Nucast gasless-tx-ts

**Library:** [@nucastio/gasless](https://github.com/Nucastio/gasless-tx-ts)  
**License:** MIT  
**Catalyst Funded:** Yes (Fund 10, ₳120,000)

The Nucast library provides the foundation for transaction sponsorship:

```typescript
import { Gasless } from "@nucastio/gasless";

// Pool configuration (Begin's backend)
const beginPool = new Gasless({
  mode: "pool",
  wallet: {
    network: 1, // mainnet
    key: {
      type: "mnemonic",
      words: process.env.BEGIN_POOL_MNEMONIC.split(" "),
    },
  },
  conditions: {
    tokenRequirements: [
      { 
        unit: USDM_POLICY_ID + USDM_ASSET_NAME,
        quantity: 1800000, // 1.80 USDM (6 decimals)
        comparison: "gte" 
      }
    ],
  },
  apiKey: process.env.BLOCKFROST_API_KEY,
});

// Start pool server
await beginPool.listen(5050);
```

**Extension Required:** The current Nucast library covers transaction fees but does not inject minADA for new outputs. We need to extend `sponsorTx` to:

1. Analyze transaction outputs
2. Calculate minADA needed for each output
3. Inject ADA from pool into outputs missing minADA
4. Adjust user's stablecoin payment requirement accordingly

## Pricing Model

### Cost Components

| Component | Cost (ADA) | Cost (USD @ $0.80/ADA) |
|-----------|-----------|------------------------|
| Transaction fee | ~0.17-0.25 | ~$0.14-0.20 |
| minADA per new UTXO | ~1.5 | ~$1.20 |
| **Simple send (no split)** | ~0.20 | ~$0.16 |
| **Send with 1 split** | ~1.70 | ~$1.36 |

### Pricing Strategy

```typescript
interface FeeCoverageQuote {
  txFeeAda: number;        // Estimated tx fee
  minAdaNeeded: number;    // Total minADA for new outputs
  totalAdaCost: number;    // txFeeAda + minAdaNeeded
  exchangeRate: number;    // ADA/USDM rate (from oracle + buffer)
  stablecoinRequired: number; // User pays this in USDM
  marginPercent: number;   // Begin's margin (10-20%)
}

function calculateQuote(tx: Transaction): FeeCoverageQuote {
  const txFee = estimateTxFee(tx);
  const newOutputs = countNewOutputsNeedingMinAda(tx);
  const minAdaNeeded = newOutputs * CURRENT_MIN_ADA;
  
  const totalAda = txFee + minAdaNeeded;
  const rate = getOracleRate("ADA/USD") * 1.15; // 15% buffer
  const stablecoinRequired = totalAda * rate;
  
  return {
    txFeeAda: txFee,
    minAdaNeeded,
    totalAdaCost: totalAda,
    exchangeRate: rate,
    stablecoinRequired,
    marginPercent: 15,
  };
}
```

### User-Facing Pricing (Suggested)

| Transaction Type | User Pays (USDM) | Begin Earns |
|-----------------|------------------|-------------|
| Simple send (same tokens, no split) | 0.25 | ~$0.08 |
| Send with 1 split | 1.80 | ~$0.35 |
| Send with 2 splits | 3.30 | ~$0.65 |
| UTXO consolidation | 0.25 | ~$0.08 |

## The Receiving Benefit

When users receive tokens, those tokens come **with minADA from the sender**:

```
Alice sends 5 USDM to User
└── User receives: 5 USDM + 1.5 ADA (Alice's minADA)
```

**Over time, users accumulate "free" ADA** they never purchased. This creates opportunities:

### UTXO Consolidation Flow

```
User has 5 UTXOs from receiving tokens:
├── 2 USDM + 1.5 ADA
├── 3 USDM + 1.5 ADA  
├── 5 USDM + 1.5 ADA
├── 1 USDM + 1.5 ADA
└── 4 USDM + 1.5 ADA
Total: 15 USDM + 7.5 ADA locked

CONSOLIDATE into 1 UTXO:
└── 15 USDM + 1.5 ADA (only needs 1x minADA now)

FREED: 6.0 ADA!
User can convert to ~7.50 USDM via Begin
```

### Consolidation UX

```typescript
// Prompt when user has excess locked ADA
function checkConsolidationOpportunity(wallet: Wallet): ConsolidationOffer | null {
  const utxos = wallet.getTokenUtxos();
  if (utxos.length <= 1) return null;
  
  const totalLockedAda = utxos.reduce((sum, u) => sum + u.adaAmount, 0);
  const minRequired = MIN_ADA_VALUE; // For consolidated UTXO
  const recoverable = totalLockedAda - minRequired;
  
  if (recoverable < 1.0) return null; // Not worth it
  
  return {
    currentUtxos: utxos.length,
    lockedAda: totalLockedAda,
    recoverableAda: recoverable,
    recoverableUsdm: recoverable * getOracleRate("ADA/USD"),
    txFee: 0.25, // USDM
  };
}
```

## Implementation Phases

### Phase 1: Basic Babel Fees (2-3 weeks)
- Integrate Nucast gasless-tx-ts library
- Implement pool server with USDM acceptance
- Cover transaction fees only (no minADA injection)
- Require users have sufficient ADA for splits

### Phase 2: Full minADA Coverage (3-4 weeks)  
- Extend Nucast library to inject minADA
- Implement dynamic pricing with oracle
- Add split detection and pricing
- Deploy ADA liquidity pool

### Phase 3: Consolidation & Optimization (2-3 weeks)
- Add UTXO consolidation prompts
- Implement ADA recovery flow
- Add batch transaction support
- Optimize pool management

## Risk Mitigation

### Price Volatility
- **Risk:** ADA price changes between quote and execution
- **Mitigation:** 15% buffer on exchange rate, 5-minute quote expiry

### Pool Depletion
- **Risk:** Pool runs out of ADA during high volume
- **Mitigation:** 
  - Auto-swap accumulated USDM → ADA (hourly)
  - Alert when pool < 1000 ADA
  - Rate limiting during low liquidity

### Abuse Prevention
- **Risk:** Users exploit the service
- **Mitigation:**
  - Whitelist supported stablecoins only
  - Rate limit per wallet address
  - Transaction simulation before signing
  - Maximum coverage per transaction

## Supported Stablecoins

| Token | Policy ID | Priority |
|-------|-----------|----------|
| USDM (Mehen) | `c48cbb3d5e57...` | Primary |
| DJED | `8db269c3ec63...` | Secondary |
| iUSD (Indigo) | `f66d78b4a3cb...` | Secondary |

## Revenue Projections

Assuming 10,000 monthly active users with average 5 txs/month:

| Scenario | Txs/Month | Avg Fee | Monthly Revenue |
|----------|-----------|---------|-----------------|
| Conservative (20% use babel fees) | 10,000 | $0.20 | $2,000 |
| Moderate (40% use babel fees) | 20,000 | $0.25 | $5,000 |
| Optimistic (60% + splits) | 30,000 | $0.35 | $10,500 |

## References

1. **Nucast gasless-tx-ts:** https://github.com/Nucastio/gasless-tx-ts
2. **Cardano minADA Documentation:** https://cardano-ledger.readthedocs.io/en/latest/explanations/min-utxo-mary.html
3. **IOHK Babel Fees Blog:** https://iohk.io/blog/posts/2021/02/25/babel-fees/
4. **Octane (Solana comparison):** https://github.com/anza-xyz/octane

## Appendix: minADA Calculation Formula

From Cardano Ledger specification:

```
minAda(u) = max(minUTxOValue, (minUTxOValue / adaOnlyUTxOSize) * (utxoEntrySizeWithoutVal + size(B)))

Where:
- minUTxOValue = 1,000,000 lovelace (1 ADA)
- adaOnlyUTxOSize = 27 bytes
- utxoEntrySizeWithoutVal = 27 bytes
- size(B) = 6 + roundupBytesToWords(numAssets * 12 + sumAssetNameLengths + numPolicyIds * 28)
```

**Practical values:**
- ADA-only UTXO: 1.0 ADA
- Single token (32-char name): ~1.56 ADA
- Multiple tokens: 1.5-2.5+ ADA depending on bundle size
