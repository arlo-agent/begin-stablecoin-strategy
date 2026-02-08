# Gasless / Fee-Sponsored Stablecoin Transactions for Begin Wallet

**Research Date:** February 8, 2026  
**Purpose:** Enable users to send stablecoins without holding native chain assets (ADA/SOL)

---

## Executive Summary

Gasless transactions are becoming a key UX differentiator for crypto wallets. Both Cardano and Solana have viable approaches to sponsor transaction fees on behalf of users, allowing stablecoin transfers without requiring users to hold native assets.

### Key Findings

| Aspect | Cardano | Solana |
|--------|---------|--------|
| **Primary Solution** | Nucast Gasless Tx Library | Octane / Kora Relayer |
| **Mechanism** | Meta-transaction + relayer | Fee payer + partial signing |
| **Maturity** | Emerging (Catalyst-funded, 2025) | Mature (Anza-maintained) |
| **Transaction Cost** | ~0.17 ADA (~$0.13 USD) | ~0.000005 SOL (~$0.001 USD) |
| **Implementation Effort** | Medium-High | Low-Medium |
| **Native Protocol Support** | Requires relayer infrastructure | Native fee payer field |

**Recommendation for Begin Wallet:** Start with Solana implementation (lower cost, mature tooling), then implement Cardano using Nucast or custom relayer.

---

## Part 1: Cardano Approach

### 1.1 Nucast Gasless Transaction Library

**Status:** Funded by Project Catalyst Fund 12 (₳120K), Development complete as of 2025

**Architecture Overview:**
```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  User Wallet    │────▶│  Relayer Script  │────▶│  Cardano Network│
│  (Signs tx)     │     │  (Adds fee UTxO) │     │  (Processes tx) │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                               │
                        ┌──────▼──────┐
                        │  Fee Pool   │
                        │  (ADA)      │
                        └─────────────┘
```

**How It Works:**

1. **User Transaction Creation:** User signs a transaction containing their intended instructions (e.g., send stablecoin) but without paying the ADA fee
2. **Relayer Scripts:** Backend relayer receives the signed transaction
3. **Fee Injection:** Relayer adds a UTXO input containing ADA to cover the transaction fee
4. **Transaction Submission:** Relayer co-signs and submits the transaction to the blockchain
5. **Fee Pool Management:** Platform maintains an ADA pool to fund transaction fees

**Key Features:**
- Open-source MIT License
- No dependencies on external services
- Well-documented API for dApp integration
- Designed for both dApps and DAOs

**Technical Considerations:**
- **Cardano UTXO Model:** Unlike account-based chains, Cardano uses UTXOs. The relayer must manage a pool of fee-paying UTXOs
- **Multi-Signature:** Transaction requires signatures from both user (for the actual transfer) and relayer (for fee payment)
- **Collateral:** For Plutus script transactions, collateral must also be sponsored

### 1.2 Alternative Cardano Approaches

#### A. Direct Multi-Signature Transaction Building
Build transactions with multiple signers where:
- User signs the stablecoin transfer instruction
- Wallet backend signs and provides the ADA fee input

**Pros:** Full control, no external dependency  
**Cons:** Must manage UTXO set carefully, more complex implementation

#### B. Transaction Chaining
Chain transactions where a preceding transaction provisions the fee:
- First tx: Backend sends minimal ADA to user
- Second tx: User sends stablecoin (now has ADA for fee)

**Pros:** Simple conceptually  
**Cons:** Two transactions, double the on-chain cost, worse UX (delay)

#### C. WalletX Advertising-Sponsored Model
An alternative Catalyst project exploring ad-sponsored transactions:
- Ads displayed in wallet subsidize transaction fees
- User sees optional ads, fees are covered

**Pros:** Revenue-generating  
**Cons:** UX tradeoff, may not align with Begin's brand

### 1.3 Cardano Transaction Fee Economics

**Current Fee Structure:**
- Formula: `fee = a * tx_size_bytes + b`
- Parameter `a` (per byte): ~44 lovelace
- Parameter `b` (base fee): ~155,381 lovelace
- **Typical simple tx:** ~0.17 ADA (~170,000 lovelace)
- **Stablecoin tx (with native asset):** ~0.18-0.25 ADA

**At current ADA price (~$0.75):**
- Simple stablecoin transfer: **~$0.13 - $0.19 USD**

---

## Part 2: Solana Approach

### 2.1 Octane - Official Gasless Relayer

**Repository:** github.com/anza-xyz/octane (maintained by Anza, formerly Solana Labs)

**Architecture:**
```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  User Wallet    │────▶│  Octane Server   │────▶│  Solana Network │
│  (Partial sign) │     │  (Fee payer)     │     │  (Processes tx) │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                       │
        │                       ▼
        │               ┌──────────────┐
        └──────────────▶│ Token Payment│
          (Pays in SPL) │ to Octane    │
                        └──────────────┘
```

**How It Works:**

1. **User Creates Transaction:** User builds a transaction with:
   - Their intended instructions (e.g., USDC transfer)
   - A fee payment instruction (sending SPL tokens to Octane operator)
   - Fee payer set to Octane's address (not their own)

2. **User Partially Signs:** User signs to authorize their instructions

3. **Octane Validates:** Octane verifies:
   - Token payment amount covers the SOL fee + margin
   - Transaction is safe (no malicious instructions)
   - Rate limits not exceeded

4. **Octane Co-Signs:** If valid, Octane signs as the fee payer

5. **Transaction Broadcast:** Octane submits the fully-signed transaction

**Key Solana Properties Enabling This:**
- **Multiple Signers:** Transactions can have separate fee payer and instruction signers
- **Atomic Execution:** All instructions succeed or fail together (fee payment included)
- **Account Validation:** Instructions declare which accounts they touch, enabling pre-validation

**Octane Deployment Options:**
- Self-hosted on Vercel (serverless)
- Self-hosted on any Node.js platform
- Use a public Octane endpoint (third-party)

### 2.2 Kora - Alternative Relayer

**Focus:** More focused on stablecoin use cases, similar architecture to Octane

**Key Differences:**
- Rust core library for validation
- TypeScript SDK for easy integration
- JSON-RPC server with endpoints:
  - `estimate_transaction_fee` - Get fee quote in SPL token
  - `get_supported_tokens` - List accepted payment tokens
  - `sign_and_send_transaction` - Submit gasless transaction

**Configuration-Driven:** Operators define allowed tokens, programs, and pricing in `kora.toml`

### 2.3 Circle Gas Station

**Enterprise Solution:** Circle offers Gas Station as part of their Programmable Wallets product

**How It Works:**
- Create policies defining when to sponsor fees
- Policies can limit by wallet, daily amount, or transaction type
- Gas Station maintains fee payer wallets automatically
- SDK handles the signing flow

**Best For:** Apps already using Circle's wallet infrastructure

### 2.4 Direct Fee Payer Implementation

For maximum control, Begin can implement fee sponsorship directly:

```javascript
// Simplified flow
const transaction = new Transaction();

// User's instruction
transaction.add(
  createTransferInstruction(userATA, recipientATA, user.publicKey, amount)
);

// Set wallet backend as fee payer
transaction.feePayer = walletBackendKeypair.publicKey;

// Get recent blockhash
transaction.recentBlockhash = (await connection.getLatestBlockhash()).blockhash;

// User signs their part (client-side)
transaction.partialSign(userKeypair);

// Backend signs as fee payer (server-side)
transaction.partialSign(walletBackendKeypair);

// Submit
await connection.sendRawTransaction(transaction.serialize());
```

### 2.5 Solana Transaction Fee Economics

**Fee Structure:**
- **Base Fee:** 5,000 lamports per signature (~0.000005 SOL)
- **Priority Fee:** Optional, varies by network congestion
- **Account Creation (rent):** ~0.002 SOL for new token accounts

**At current SOL price (~$190):**
- Simple transfer (existing accounts): **~$0.001 USD**
- Transfer + new account creation: **~$0.40 USD**

**Critical Note:** Token Account Rent
- First-time recipients need a token account created
- This costs ~0.002 SOL in rent (refundable if account closed)
- This is the bigger cost to sponsor, not the transaction fee itself

---

## Part 3: Comparison & Recommendations

### 3.1 Comparison Table

| Feature | Cardano (Nucast) | Solana (Octane) |
|---------|------------------|-----------------|
| **Transaction Cost** | ~$0.15 | ~$0.001 |
| **Account Setup Cost** | N/A (no separate accounts) | ~$0.40 (token account rent) |
| **Implementation Maturity** | New (2025) | Mature (2022+) |
| **Open Source** | Yes (MIT) | Yes (MIT) |
| **Hosted Options** | Self-host only | Vercel, self-host, third-party |
| **Token Payment** | Not native (relayer absorbs) | Native (user pays in SPL) |
| **Multi-Sig Required** | Yes | Yes (partial signing) |
| **Smart Contract Risk** | Low (no SC for basic tx) | Low |

### 3.2 Security Considerations

**For Both Chains:**

1. **Rate Limiting:** Prevent spam attacks exhausting the fee pool
   - Per-user limits (by wallet address)
   - Global rate limits
   - Progressive backoff

2. **Transaction Validation:** Prevent malicious transactions
   - Whitelist allowed programs/scripts
   - Validate transaction structure before signing
   - Simulate transactions before submitting

3. **Fee Pool Management:**
   - Don't hold more than necessary (limit exposure)
   - Auto-replenish from swap fees
   - Monitor for unusual spending patterns

4. **Abuse Prevention:**
   - Link gasless transactions to verified users
   - Limit free transactions per account/period
   - Require in-app activity (swaps) to earn free transactions

**Cardano-Specific:**
- Manage UTXO fragmentation (consolidate periodically)
- Handle collateral for script transactions

**Solana-Specific:**
- Never return fee payer signature to client before broadcast
- Validate the fee payer isn't touched by user's instructions
- Watch for token price slippage (SPL payment value)

### 3.3 Recommended Implementation Path for Begin Wallet

#### Phase 1: Solana (2-4 weeks)
1. **Implement direct fee payer pattern** (no Octane dependency)
2. **Backend signs as fee payer** for stablecoin transfers
3. **Fund with swap fee revenue** - maintain ~$50-100 in SOL pool
4. **Track usage** per user for abuse prevention

#### Phase 2: Cardano (4-6 weeks)
1. **Evaluate Nucast library** (if mature and documented)
2. **Alternatively, build custom relayer:**
   - Maintain ADA UTXO pool
   - Accept partially-signed transactions from users
   - Co-sign with fee-paying UTXO
   - Submit to network
3. **Fund with swap fee revenue** - maintain ~100-200 ADA pool

#### Phase 3: Enhancements
1. **Add SPL token payment option** (user pays fee in stablecoin)
2. **Implement tiered access** (free for verified, paid for others)
3. **Add analytics** to track cost per user

---

## Part 4: Cost Analysis & Revenue Model

### 4.1 Transaction Cost Summary

| Chain | Cost per Tx | Free Txs per $1 |
|-------|-------------|-----------------|
| Solana | $0.001 | 1,000 |
| Cardano | $0.15 | 6-7 |

### 4.2 Revenue from Swap Fees

**Assumptions:**
- Average swap: $100
- Swap fee: 0.5% = $0.50 per swap

**Cardano Subsidization:**
- 1 swap ($0.50 revenue) = ~3 free stablecoin transactions
- Break-even: 1 swap : 3 sends

**Solana Subsidization:**
- 1 swap ($0.50 revenue) = ~500 free stablecoin transactions
- Break-even: 1 swap : 500 sends

### 4.3 Free Transaction Budget Recommendations

**For User Acquisition (promotional):**
- Solana: Offer 10 free transactions per new user (~$0.01 cost)
- Cardano: Offer 2 free transactions per new user (~$0.30 cost)

**Ongoing Free Tier:**
- Solana: 5 free tx/month per active user
- Cardano: 1 free tx/month per active user (or require swap activity)

### 4.4 Projected Monthly Costs

**Scenario: 10,000 MAU, 50% use gasless**

| Chain | Free Tx/User | Total Tx | Monthly Cost |
|-------|--------------|----------|--------------|
| Solana | 5 | 25,000 | $25 |
| Cardano | 2 | 10,000 | $1,500 |

**Takeaway:** Solana gasless is nearly free to offer. Cardano requires careful budgeting but is still viable if subsidized by swap revenue.

---

## Part 5: Technical Implementation Notes

### 5.1 Solana Implementation Skeleton

```typescript
// Backend service - Gasless Transaction Handler

interface GaslessRequest {
  serializedTransaction: string; // Base64 encoded, user-signed
  userPubkey: string;
}

async function processGaslessTransaction(req: GaslessRequest): Promise<string> {
  // 1. Decode transaction
  const tx = Transaction.from(Buffer.from(req.serializedTransaction, 'base64'));
  
  // 2. Validate
  if (!isValidGaslessTransaction(tx, req.userPubkey)) {
    throw new Error('Invalid transaction');
  }
  
  // 3. Check rate limits
  if (!checkRateLimit(req.userPubkey)) {
    throw new Error('Rate limit exceeded');
  }
  
  // 4. Set fee payer and sign
  tx.feePayer = feePayerKeypair.publicKey;
  tx.partialSign(feePayerKeypair);
  
  // 5. Submit
  const signature = await connection.sendRawTransaction(tx.serialize());
  
  // 6. Track usage
  recordTransaction(req.userPubkey);
  
  return signature;
}

function isValidGaslessTransaction(tx: Transaction, expectedSigner: string): boolean {
  // Check transaction structure
  // Verify user signature
  // Ensure no malicious instructions
  // Validate fee payer isn't being drained
  return true; // Implement actual validation
}
```

### 5.2 Cardano Implementation Considerations

```typescript
// Conceptual flow - Cardano Gasless

interface CardanoGaslessRequest {
  partialTxHex: string; // CBOR-encoded partial transaction
  userWitnesses: string; // User's signatures
}

async function processCardanoGasless(req: CardanoGaslessRequest): Promise<string> {
  // 1. Decode partial transaction
  const tx = decodeTransaction(req.partialTxHex);
  
  // 2. Get fee-paying UTXO from pool
  const feeUtxo = await selectFeeUtxo(estimateFee(tx));
  
  // 3. Add fee input to transaction
  tx.addInput(feeUtxo);
  
  // 4. Recalculate and set change output
  tx.setChange(feePayerAddress, feeUtxo.value - fee);
  
  // 5. Sign with fee payer key
  tx.addWitness(feePayerKey.sign(tx.hash()));
  
  // 6. Add user witnesses
  tx.addWitnesses(decodeWitnesses(req.userWitnesses));
  
  // 7. Submit
  return await submitTransaction(tx);
}
```

---

## Part 6: References & Resources

### Cardano
- **Nucast Project Catalyst Proposal:** https://projectcatalyst.io/funds/12/cardano-open-developers/gasless-tx-library-empowering-cardano-dapps-and-daos-to-sponsor-gas-fees-to-increase-user-base-and-adoption-open-source
- **Nucast Website:** https://www.nucast.io/
- **Cardano Fee Docs:** https://docs.cardano.org/about-cardano/explore-more/fee-structure
- **Cardano Transaction Guide:** https://developers.cardano.org/docs/get-started/infrastructure/cardano-cli/basic-operations/simple-transactions/

### Solana
- **Octane GitHub:** https://github.com/anza-xyz/octane
- **Octane Recipes:** https://github.com/anza-xyz/octane/blob/master/docs/recipes.md
- **Circle Gas Station:** https://www.circle.com/blog/how-circles-gas-station-uses-fee-payers-to-enable-gasless-transactions-on-solana
- **Kora Relayer:** Referenced in dev.to article on gasless transactions
- **Solana Fee Docs:** https://solana.com/docs/core/fees

---

## Appendix: Glossary

- **Gasless Transaction:** A transaction where the end user doesn't directly pay the network fee
- **Fee Payer:** An account that pays the transaction fee on Solana (can be different from the transaction signer)
- **Relayer:** A backend service that co-signs and submits transactions on behalf of users
- **Meta-Transaction:** A transaction pattern where a third party submits a user-signed message on-chain
- **UTXO:** Unspent Transaction Output - Cardano's transaction model
- **SPL Token:** Solana Program Library Token - Solana's token standard (like ERC-20)
- **Lamport:** Smallest unit of SOL (1 SOL = 1 billion lamports)
- **Lovelace:** Smallest unit of ADA (1 ADA = 1 million lovelace)
