# Begin Wallet: Stablecoin Financial Services Implementation Plan

*Analysis completed: 2026-02-03*
*Based on: stablecoin-financial-services-begin-strategy.md research document*
*Codebase analyzed: begin-cli v0.1.0*

---

## Executive Summary

This document provides a detailed technical implementation plan for transforming Begin Wallet into "The Stablecoin Bank for Cardano." After analyzing the research document and the current begin-cli codebase, I've mapped each strategic opportunity to specific implementation requirements, identified existing infrastructure, and prioritized by effort vs. impact.

**Key Finding:** Begin Wallet already has ~60% of the foundational infrastructure needed. The core primitives (wallet management, swap routing via Minswap, transaction building, staking) are implemented. The main gaps are: yield aggregation, multi-protocol DeFi integration, and stablecoin-specific optimizations.

---

## Part 1: Current Codebase Analysis

### What's Already Built ✅

| Feature | Status | Files | Notes |
|---------|--------|-------|-------|
| **Wallet Management** | Complete | `lib/wallet.ts`, `lib/keystore.ts` | Encrypted storage, mnemonic support, multi-wallet |
| **Transaction Building** | Complete | `lib/transaction.ts` | MeshJS-based, supports ADA + native assets |
| **Swap Routing (Minswap)** | Complete | `lib/swap.ts`, `services/minswap.ts` | Full aggregator integration, quote + execute |
| **Staking Operations** | Partial (Mock) | `lib/staking.ts`, `commands/stake/` | Pool search, delegation UI exists but transactions are mocked |
| **Balance/UTxO Queries** | Complete | `services/blockfrost.ts` | Full Blockfrost integration |
| **CLI Framework** | Complete | `cli.tsx`, `app.tsx` | Ink-based React CLI, JSON output mode |
| **QR Code Generation** | Complete | `lib/qr.ts` | For receive addresses |
| **Network Support** | Complete | `lib/config.ts` | mainnet, preprod, preview |

### What's Missing/Needs Building ❌

| Feature | Priority | Effort | Dependencies |
|---------|----------|--------|--------------|
| **Yield Rate Aggregation** | High | Medium | Liqwid API, Lenfi API, Minswap LP data |
| **Vault/Strategy System** | High | High | Smart contracts or aggregator integration |
| **Multi-DEX Routing** | Medium | Medium | SundaeSwap, WingRiders APIs |
| **Stablecoin Token Registry** | High | Low | Token metadata service |
| **DCA Automation** | Medium | Medium | Cron/scheduler, persistent state |
| **Real Staking Transactions** | Medium | Low | Already has MeshJS, just needs implementation |
| **Yield Dashboard Data** | Medium | Medium | APY tracking, historical data |

---

## Part 2: Feature Implementation Breakdown

### 2.1 Stablecoin Swap Router (HIGH PRIORITY)

**Research Reference:** "Best-execution routing between USDA/USDM/iUSD/DJED"

**Current State:** ✅ Minswap aggregator integration complete

**What's Built:**
```typescript
// lib/swap.ts - Already supports:
- Token ID resolution (ADA, lovelace, policyId.assetName)
- Quote fetching with slippage tolerance
- Multi-hop routing via Minswap aggregator
- Transaction building and signing
- Witness extraction for submission
```

**Gaps to Fill:**

1. **Stablecoin Token Registry** - Add well-known stablecoin aliases
2. **Multi-DEX Support** - Add SundaeSwap, WingRiders as routing options
3. **Stablecoin-Specific Optimizations** - Detect stable-to-stable swaps, optimize for minimal slippage

**Implementation:**

```typescript
// lib/swap.ts - ADD to TOKEN_ALIASES:
const STABLECOIN_ALIASES: Record<string, string> = {
  // USDA (Anzens)
  'USDA': 'c48cbb3d5e57ed56e276bc45f99ab39abe94e6cd7ac39fb402da47ad0014df105553444154455354',
  'usda': 'c48cbb3d5e57ed56e276bc45f99ab39abe94e6cd7ac39fb402da47ad0014df105553444154455354',
  
  // USDM (Moneta/Mehen)
  'USDM': 'c48cbb3d5e57ed56e276bc45f99ab39abe94e6cd7ac39fb402da47ad0014df1055534d',
  'usdm': 'c48cbb3d5e57ed56e276bc45f99ab39abe94e6cd7ac39fb402da47ad0014df1055534d',
  
  // iUSD (Indigo)
  'iUSD': '...',
  'IUSD': '...',
  
  // DJED (COTI)
  'DJED': '...',
  'djed': '...',
};
```

**New CLI Commands:**
```bash
# List supported stablecoins with current rates
begin stablecoins list [--json]

# Get best route between stablecoins
begin swap quote --from USDA --to USDM --amount 1000 --json

# Execute stablecoin swap
begin swap --from USDA --to iUSD --amount 500 --slippage 0.1 --yes
```

**Mobile UI Flow:**
1. Dashboard shows stablecoin balances prominently
2. "Swap" button → Token selector with stablecoin favorites
3. Real-time rate preview
4. One-tap confirm with biometric auth
5. Transaction status + explorer link

**Effort:** Low (2-3 days) - Mostly config and UI polish
**Impact:** High - Foundation for all other features

---

### 2.2 Yield Rate Aggregation API (HIGH PRIORITY)

**Research Reference:** "Real-time view of all yield opportunities across Cardano DeFi"

**Current State:** ❌ Not implemented

**What's Needed:**

Create a new service module that aggregates yield data from multiple protocols:

```typescript
// NEW FILE: src/services/yield-aggregator.ts

export interface YieldSource {
  protocol: string;           // 'liqwid', 'lenfi', 'minswap-lp', 'indigo'
  asset: string;              // 'USDA', 'USDM', 'iUSD', 'ADA'
  type: 'lending' | 'lp' | 'staking' | 'synthetic';
  apy: number;                // Current APY as percentage
  apyHistory: ApyDataPoint[]; // Historical data
  tvl: string;                // Total value locked in lovelace
  riskScore: 'low' | 'medium' | 'high';
  audited: boolean;
  minDeposit: string;
  lockupPeriod?: number;      // Days, if any
}

export interface ApyDataPoint {
  timestamp: number;
  apy: number;
}

// Protocol-specific fetchers
export interface ProtocolAdapter {
  name: string;
  fetchYieldSources(network: string): Promise<YieldSource[]>;
}

// Liqwid Finance integration
export class LiqwidAdapter implements ProtocolAdapter {
  name = 'liqwid';
  
  async fetchYieldSources(network: string): Promise<YieldSource[]> {
    // Liqwid API: https://api.liqwid.finance/markets
    const response = await fetch('https://api.liqwid.finance/markets');
    const markets = await response.json();
    
    return markets.map(m => ({
      protocol: 'liqwid',
      asset: m.asset,
      type: 'lending',
      apy: m.supplyApy * 100,
      tvl: m.totalSupply,
      riskScore: this.calculateRiskScore(m),
      audited: true,
      minDeposit: '0',
    }));
  }
}

// Lenfi integration
export class LenfiAdapter implements ProtocolAdapter {
  name = 'lenfi';
  // Similar implementation...
}

// Minswap LP integration
export class MinswapLpAdapter implements ProtocolAdapter {
  name = 'minswap';
  
  async fetchYieldSources(network: string): Promise<YieldSource[]> {
    // Use existing MinswapClient + pool data
    // Calculate LP APY from trading fees + rewards
  }
}

// Main aggregator
export class YieldAggregator {
  private adapters: ProtocolAdapter[];
  
  constructor() {
    this.adapters = [
      new LiqwidAdapter(),
      new LenfiAdapter(),
      new MinswapLpAdapter(),
    ];
  }
  
  async getAllYields(network: string, filter?: {
    asset?: string;
    minApy?: number;
    maxRisk?: 'low' | 'medium' | 'high';
  }): Promise<YieldSource[]> {
    const results = await Promise.all(
      this.adapters.map(a => a.fetchYieldSources(network))
    );
    
    let yields = results.flat();
    
    // Apply filters
    if (filter?.asset) {
      yields = yields.filter(y => y.asset === filter.asset);
    }
    if (filter?.minApy) {
      yields = yields.filter(y => y.apy >= filter.minApy);
    }
    if (filter?.maxRisk) {
      // Risk ordering: low < medium < high
      const riskOrder = { low: 1, medium: 2, high: 3 };
      yields = yields.filter(y => riskOrder[y.riskScore] <= riskOrder[filter.maxRisk!]);
    }
    
    // Sort by APY descending
    return yields.sort((a, b) => b.apy - a.apy);
  }
  
  async getBestYield(asset: string, network: string): Promise<YieldSource | null> {
    const yields = await this.getAllYields(network, { asset });
    return yields[0] || null;
  }
}
```

**New CLI Commands:**
```bash
# List all yield opportunities
begin yield list [--asset USDA] [--min-apy 5] [--max-risk medium] [--json]

# Get best yield for a specific asset
begin yield best --asset USDA [--json]

# Show yield comparison table
begin yield compare --assets USDA,USDM,iUSD [--json]

# Historical yield data
begin yield history --asset USDA --protocol liqwid --days 30 [--json]
```

**Example Output:**
```
┌──────────┬──────────┬────────┬───────┬──────────┬────────┐
│ Protocol │ Asset    │ Type   │ APY   │ TVL      │ Risk   │
├──────────┼──────────┼────────┼───────┼──────────┼────────┤
│ Liqwid   │ USDA     │ Lend   │ 8.2%  │ $2.1M    │ Low    │
│ Lenfi    │ USDA     │ Lend   │ 7.8%  │ $890K    │ Medium │
│ Minswap  │ USDA/ADA │ LP     │ 12.4% │ $1.5M    │ Medium │
│ Indigo   │ iUSD     │ Synth  │ 5.1%  │ $3.2M    │ Low    │
└──────────┴──────────┴────────┴───────┴──────────┴────────┘
```

**Mobile UI Flow:**
1. "Earn" tab shows yield opportunities sorted by APY
2. Filter by: Asset type, Risk level, Protocol
3. Tap opportunity → Detail view with historical chart
4. "Deposit" button → Amount input → Confirm

**Effort:** Medium (1-2 weeks)
**Impact:** High - Core value proposition for stablecoin users

---

### 2.3 Stablecoin Savings Vault (HIGH PRIORITY)

**Research Reference:** "One-tap deposit → earn yield across Liqwid/Lenfi/Minswap LPs"

**Current State:** ❌ Not implemented

**Architecture Options:**

**Option A: Smart Contract Vault (Full Decentralization)**
- Deploy Plutus smart contracts that manage user deposits
- Auto-allocate to protocols based on strategy
- Complex, requires audit, higher effort

**Option B: Aggregator-Assisted (Recommended for V1)**
- Begin acts as UX layer, user holds assets directly in protocols
- No custodial risk, simpler implementation
- Can upgrade to smart contracts later

**Recommended V1 Implementation (Option B):**

```typescript
// NEW FILE: src/lib/vault.ts

export type VaultStrategy = 'conservative' | 'balanced' | 'aggressive';

export interface VaultConfig {
  strategy: VaultStrategy;
  allocations: ProtocolAllocation[];
}

export interface ProtocolAllocation {
  protocol: string;
  percentage: number;
  targetAsset: string;
}

// Strategy definitions
export const VAULT_STRATEGIES: Record<VaultStrategy, VaultConfig> = {
  conservative: {
    strategy: 'conservative',
    allocations: [
      { protocol: 'liqwid', percentage: 100, targetAsset: 'USDA' },
    ],
  },
  balanced: {
    strategy: 'balanced',
    allocations: [
      { protocol: 'liqwid', percentage: 60, targetAsset: 'USDA' },
      { protocol: 'lenfi', percentage: 40, targetAsset: 'USDA' },
    ],
  },
  aggressive: {
    strategy: 'aggressive',
    allocations: [
      { protocol: 'liqwid', percentage: 40, targetAsset: 'USDA' },
      { protocol: 'minswap-lp', percentage: 40, targetAsset: 'USDA/ADA' },
      { protocol: 'indigo', percentage: 20, targetAsset: 'iUSD' },
    ],
  },
};

export interface VaultPosition {
  protocol: string;
  asset: string;
  deposited: string;       // Amount in smallest units
  currentValue: string;    // Current value including rewards
  earnedRewards: string;
  apy: number;
  depositTxHash: string;
  depositTimestamp: number;
}

export interface UserVault {
  strategy: VaultStrategy;
  positions: VaultPosition[];
  totalDeposited: string;
  totalValue: string;
  totalEarned: string;
  weightedApy: number;
}

// Protocol interaction interfaces
export interface ProtocolDepositor {
  name: string;
  supportedAssets: string[];
  
  // Build deposit transaction
  buildDepositTx(
    wallet: MeshWallet,
    asset: string,
    amount: string
  ): Promise<{ unsignedTx: string; depositAddress: string }>;
  
  // Build withdrawal transaction  
  buildWithdrawTx(
    wallet: MeshWallet,
    asset: string,
    amount: string
  ): Promise<{ unsignedTx: string }>;
  
  // Get user's position in this protocol
  getPosition(address: string, asset: string): Promise<VaultPosition | null>;
}

// Liqwid depositor implementation
export class LiqwidDepositor implements ProtocolDepositor {
  name = 'liqwid';
  supportedAssets = ['USDA', 'USDM', 'ADA'];
  
  async buildDepositTx(wallet: MeshWallet, asset: string, amount: string) {
    // Liqwid uses specific market contracts
    // Build transaction to supply to the lending market
    
    const marketAddress = this.getMarketAddress(asset);
    const tx = new Transaction({ initiator: wallet });
    
    // Send tokens to market contract with deposit datum
    tx.sendAssets(marketAddress, [{ unit: asset, quantity: amount }]);
    
    const unsignedTx = await tx.build();
    return { unsignedTx, depositAddress: marketAddress };
  }
  
  // ... other methods
}

// Main vault service
export class VaultService {
  private depositors: Map<string, ProtocolDepositor>;
  private yieldAggregator: YieldAggregator;
  
  constructor() {
    this.depositors = new Map([
      ['liqwid', new LiqwidDepositor()],
      ['lenfi', new LenfiDepositor()],
      ['minswap-lp', new MinswapLpDepositor()],
    ]);
    this.yieldAggregator = new YieldAggregator();
  }
  
  async deposit(
    wallet: MeshWallet,
    stablecoin: string,
    amount: string,
    strategy: VaultStrategy,
    config: TransactionConfig
  ): Promise<{ txHashes: string[]; positions: VaultPosition[] }> {
    const strategyConfig = VAULT_STRATEGIES[strategy];
    const amountBigInt = BigInt(amount);
    const txHashes: string[] = [];
    const positions: VaultPosition[] = [];
    
    for (const allocation of strategyConfig.allocations) {
      const depositor = this.depositors.get(allocation.protocol);
      if (!depositor) continue;
      
      // Calculate allocation amount
      const allocAmount = (amountBigInt * BigInt(allocation.percentage)) / 100n;
      
      // Build and submit transaction
      const { unsignedTx } = await depositor.buildDepositTx(
        wallet,
        stablecoin,
        allocAmount.toString()
      );
      
      const signedTx = await wallet.signTx(unsignedTx);
      const provider = createProvider(config);
      const txHash = await provider.submitTx(signedTx);
      
      txHashes.push(txHash);
      positions.push({
        protocol: allocation.protocol,
        asset: stablecoin,
        deposited: allocAmount.toString(),
        currentValue: allocAmount.toString(),
        earnedRewards: '0',
        apy: await this.getProtocolApy(allocation.protocol, stablecoin),
        depositTxHash: txHash,
        depositTimestamp: Date.now(),
      });
    }
    
    return { txHashes, positions };
  }
  
  async getUserVault(address: string, strategy: VaultStrategy): Promise<UserVault> {
    const strategyConfig = VAULT_STRATEGIES[strategy];
    const positions: VaultPosition[] = [];
    
    for (const allocation of strategyConfig.allocations) {
      const depositor = this.depositors.get(allocation.protocol);
      if (!depositor) continue;
      
      const position = await depositor.getPosition(address, allocation.targetAsset);
      if (position) positions.push(position);
    }
    
    // Calculate totals
    const totalDeposited = positions.reduce((sum, p) => sum + BigInt(p.deposited), 0n);
    const totalValue = positions.reduce((sum, p) => sum + BigInt(p.currentValue), 0n);
    const totalEarned = totalValue - totalDeposited;
    
    // Calculate weighted APY
    const weightedApy = positions.reduce((sum, p) => {
      const weight = Number(BigInt(p.deposited)) / Number(totalDeposited);
      return sum + (p.apy * weight);
    }, 0);
    
    return {
      strategy,
      positions,
      totalDeposited: totalDeposited.toString(),
      totalValue: totalValue.toString(),
      totalEarned: totalEarned.toString(),
      weightedApy,
    };
  }
}
```

**New CLI Commands:**
```bash
# Deposit to vault
begin vault deposit --amount 1000 --stablecoin USDA --strategy conservative [--json]
begin vault deposit --amount 5000 --stablecoin USDM --strategy balanced --yes

# Check vault status
begin vault status [--json]

# Withdraw from vault
begin vault withdraw --amount 500 --stablecoin USDA [--protocol liqwid] [--json]
begin vault withdraw --all --stablecoin USDA

# Rebalance vault (change strategy)
begin vault rebalance --strategy aggressive [--json]

# View vault history
begin vault history [--limit 20] [--json]
```

**Example Vault Status Output:**
```
┌─────────────────────────────────────────────────────────────────┐
│  🏦 Stablecoin Vault - Conservative Strategy                    │
├─────────────────────────────────────────────────────────────────┤
│  Total Deposited:  1,000.00 USDA                                │
│  Current Value:    1,023.45 USDA (+2.3%)                        │
│  Earned Rewards:   23.45 USDA                                   │
│  Weighted APY:     8.2%                                         │
├─────────────────────────────────────────────────────────────────┤
│  Positions:                                                      │
│  ├─ Liqwid (USDA)  │ 1,023.45  │ 8.2% APY │ Low Risk           │
└─────────────────────────────────────────────────────────────────┘
```

**Mobile UI Flow:**
1. "Vault" tab shows current positions with live values
2. "Deposit" → Choose stablecoin → Enter amount → Select strategy
3. Strategy picker shows risk/reward comparison
4. Confirm → Sign transaction → Success with position details
5. Auto-refresh balances, show earnings growth animation

**Effort:** High (3-4 weeks)
**Impact:** Critical - This is the main "stickiness" feature

---

### 2.4 Dollar-Cost Averaging (MEDIUM PRIORITY)

**Research Reference:** "Scheduled recurring buys into ADA or other tokens"

**Current State:** ❌ Not implemented

**Implementation Approach:**

DCA requires persistent state and scheduled execution. Two approaches:

**Option A: Agent-Based (Recommended)**
- User creates DCA plan stored locally
- AI agent (via cron/heartbeat) executes on schedule
- Requires wallet to be unlocked (env var or password prompt)

**Option B: Server-Based**
- Begin runs a backend service
- User authorizes recurring transactions
- More complex, requires infrastructure

**V1 Implementation (Option A with Local State):**

```typescript
// NEW FILE: src/lib/dca.ts

export interface DcaPlan {
  id: string;
  createdAt: number;
  
  // What to buy
  tokenOut: string;           // 'ADA', 'MIN', etc.
  
  // What to spend
  tokenIn: string;            // Usually a stablecoin: 'USDA', 'USDM'
  amountPerExecution: string; // Amount to spend each time
  
  // Schedule
  frequency: 'daily' | 'weekly' | 'biweekly' | 'monthly';
  nextExecution: number;      // Unix timestamp
  
  // Limits
  totalBudget?: string;       // Optional: stop after spending this much
  endDate?: number;           // Optional: stop after this date
  
  // State
  status: 'active' | 'paused' | 'completed' | 'cancelled';
  executionHistory: DcaExecution[];
  totalSpent: string;
  totalAcquired: string;
  averagePrice: string;
}

export interface DcaExecution {
  timestamp: number;
  txHash: string;
  amountIn: string;
  amountOut: string;
  price: string;
  status: 'success' | 'failed';
  error?: string;
}

// DCA state persistence
const DCA_STATE_FILE = join(CONFIG_DIR, 'dca-plans.json');

export function loadDcaPlans(): DcaPlan[] {
  if (!existsSync(DCA_STATE_FILE)) return [];
  return JSON.parse(readFileSync(DCA_STATE_FILE, 'utf8'));
}

export function saveDcaPlans(plans: DcaPlan[]): void {
  writeFileSync(DCA_STATE_FILE, JSON.stringify(plans, null, 2), { mode: 0o600 });
}

// Frequency to milliseconds
const FREQUENCY_MS: Record<string, number> = {
  daily: 24 * 60 * 60 * 1000,
  weekly: 7 * 24 * 60 * 60 * 1000,
  biweekly: 14 * 24 * 60 * 60 * 1000,
  monthly: 30 * 24 * 60 * 60 * 1000,
};

export class DcaService {
  async createPlan(params: {
    tokenIn: string;
    tokenOut: string;
    amountPerExecution: string;
    frequency: DcaPlan['frequency'];
    totalBudget?: string;
    endDate?: number;
  }): Promise<DcaPlan> {
    const plans = loadDcaPlans();
    
    const plan: DcaPlan = {
      id: crypto.randomUUID(),
      createdAt: Date.now(),
      ...params,
      nextExecution: Date.now() + FREQUENCY_MS[params.frequency],
      status: 'active',
      executionHistory: [],
      totalSpent: '0',
      totalAcquired: '0',
      averagePrice: '0',
    };
    
    plans.push(plan);
    saveDcaPlans(plans);
    
    return plan;
  }
  
  async executeDuePlans(
    walletOptions: WalletOptions,
    config: TransactionConfig
  ): Promise<{ executed: number; results: DcaExecution[] }> {
    const plans = loadDcaPlans();
    const now = Date.now();
    const results: DcaExecution[] = [];
    
    for (const plan of plans) {
      if (plan.status !== 'active') continue;
      if (plan.nextExecution > now) continue;
      
      // Check budget/end date limits
      if (plan.totalBudget && BigInt(plan.totalSpent) >= BigInt(plan.totalBudget)) {
        plan.status = 'completed';
        continue;
      }
      if (plan.endDate && now >= plan.endDate) {
        plan.status = 'completed';
        continue;
      }
      
      // Execute the swap
      const execution = await this.executeSwap(plan, walletOptions, config);
      results.push(execution);
      
      // Update plan state
      plan.executionHistory.push(execution);
      plan.nextExecution = now + FREQUENCY_MS[plan.frequency];
      
      if (execution.status === 'success') {
        plan.totalSpent = (BigInt(plan.totalSpent) + BigInt(execution.amountIn)).toString();
        plan.totalAcquired = (BigInt(plan.totalAcquired) + BigInt(execution.amountOut)).toString();
        plan.averagePrice = this.calculateAveragePrice(plan);
      }
    }
    
    saveDcaPlans(plans);
    return { executed: results.length, results };
  }
  
  private async executeSwap(
    plan: DcaPlan,
    walletOptions: WalletOptions,
    config: TransactionConfig
  ): Promise<DcaExecution> {
    try {
      const quote = await getSwapQuote(
        plan.tokenIn,
        plan.tokenOut,
        plan.amountPerExecution,
        0.5,
        config.network,
        { amountInDecimal: false }
      );
      
      const result = await executeSwap(walletOptions, config, quote);
      
      return {
        timestamp: Date.now(),
        txHash: result.txHash!,
        amountIn: quote.amountIn,
        amountOut: quote.amountOut,
        price: (parseFloat(quote.amountIn) / parseFloat(quote.amountOut)).toString(),
        status: 'success',
      };
    } catch (error) {
      return {
        timestamp: Date.now(),
        txHash: '',
        amountIn: '0',
        amountOut: '0',
        price: '0',
        status: 'failed',
        error: error instanceof Error ? error.message : 'Unknown error',
      };
    }
  }
}
```

**New CLI Commands:**
```bash
# Create DCA plan
begin dca create --token ADA --amount 50 --funding USDA --frequency weekly [--budget 1000] [--json]

# List active DCA plans
begin dca list [--json]

# View specific plan
begin dca status <plan-id> [--json]

# Pause/resume plan
begin dca pause <plan-id>
begin dca resume <plan-id>

# Cancel plan
begin dca cancel <plan-id>

# Execute due plans (for cron/agent)
begin dca execute [--json]
```

**Cron/Agent Integration:**
```bash
# Add to crontab for daily execution check
0 9 * * * BEGIN_CLI_MNEMONIC="..." /usr/local/bin/begin dca execute --json >> /var/log/begin-dca.log
```

**Mobile UI Flow:**
1. "DCA" tab shows active plans with progress bars
2. "Create Plan" → Select token to buy → Select funding source → Set amount → Set frequency
3. Optional: Set budget limit, end date
4. Confirm → Plan created, shows next execution
5. History view shows all executions with prices, average price chart

**Effort:** Medium (1-2 weeks)
**Impact:** Medium - Good retention feature, builds habit

---

### 2.5 Multi-DEX Routing (MEDIUM PRIORITY)

**Research Reference:** "Query Minswap, SundaeSwap, WingRiders pools"

**Current State:** Partial - Minswap only

**Implementation:**

```typescript
// NEW FILE: src/services/sundaeswap.ts

export interface SundaeSwapClient {
  getQuote(tokenIn: string, tokenOut: string, amount: string): Promise<SwapQuote>;
  buildTx(quote: SwapQuote, sender: string): Promise<{ cbor: string }>;
}

// SundaeSwap API integration
export class SundaeSwapClient implements SundaeSwapClient {
  private baseUrl = 'https://api.sundaeswap.finance';
  
  async getQuote(tokenIn: string, tokenOut: string, amount: string): Promise<SwapQuote> {
    // SundaeSwap quote API
    const response = await fetch(`${this.baseUrl}/v1/quote`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        tokenA: tokenIn,
        tokenB: tokenOut,
        amount,
      }),
    });
    return response.json();
  }
}

// NEW FILE: src/services/wingriders.ts
// Similar implementation for WingRiders

// UPDATE: src/lib/swap.ts - Add multi-DEX routing

export interface DexQuote {
  dex: 'minswap' | 'sundaeswap' | 'wingriders';
  quote: SwapQuote;
}

export async function getBestSwapQuote(
  tokenIn: string,
  tokenOut: string,
  amount: string,
  slippage: number,
  network: string
): Promise<DexQuote> {
  const minswapClient = createMinswapClient(network);
  const sundaeClient = new SundaeSwapClient();
  const wingridersClient = new WingRidersClient();
  
  // Fetch quotes from all DEXs in parallel
  const [minswapQuote, sundaeQuote, wingridersQuote] = await Promise.allSettled([
    minswapClient.estimate({ token_in: tokenIn, token_out: tokenOut, amount, slippage }),
    sundaeClient.getQuote(tokenIn, tokenOut, amount),
    wingridersClient.getQuote(tokenIn, tokenOut, amount),
  ]);
  
  // Compare and return best quote
  const quotes: DexQuote[] = [];
  
  if (minswapQuote.status === 'fulfilled') {
    quotes.push({ dex: 'minswap', quote: minswapQuote.value });
  }
  if (sundaeQuote.status === 'fulfilled') {
    quotes.push({ dex: 'sundaeswap', quote: sundaeQuote.value });
  }
  if (wingridersQuote.status === 'fulfilled') {
    quotes.push({ dex: 'wingriders', quote: wingridersQuote.value });
  }
  
  // Sort by output amount (descending)
  quotes.sort((a, b) => 
    BigInt(b.quote.amountOut) - BigInt(a.quote.amountOut) > 0n ? 1 : -1
  );
  
  if (quotes.length === 0) {
    throw new Error('No DEX could provide a quote for this swap');
  }
  
  return quotes[0];
}
```

**New CLI Commands:**
```bash
# Get quotes from all DEXs
begin swap compare --from ADA --to MIN --amount 100 [--json]

# Force specific DEX
begin swap --from ADA --to MIN --amount 100 --dex sundaeswap

# Auto-route to best DEX (default)
begin swap --from ADA --to MIN --amount 100 --best-route
```

**Effort:** Medium (2-3 weeks) - Depends on DEX API documentation
**Impact:** Medium - Better rates, competitive advantage

---

### 2.6 Real Staking Transactions (MEDIUM PRIORITY)

**Research Reference:** Foundation feature - Current implementation is mocked

**Current State:** Mock implementation exists in `commands/stake/delegate.tsx`

**What's Needed:** Replace mock with real MeshJS transactions

```typescript
// UPDATE: src/commands/stake/delegate.tsx

// Replace simulateDelegation() with:
const executeDelegation = async () => {
  try {
    setState('building');
    
    // Load wallet
    const wallet = await loadWallet({ walletName, password }, { network });
    
    // Get stake address from wallet
    const rewardAddresses = await wallet.getRewardAddresses();
    const stakeAddress = rewardAddresses[0];
    
    if (!stakeAddress) {
      throw new Error('Could not derive stake address from wallet');
    }
    
    // Build delegation transaction
    const tx = new Transaction({ initiator: wallet });
    
    // Register stake key if needed
    if (needsRegistration) {
      tx.registerStake(stakeAddress);
    }
    
    // Delegate to pool
    tx.delegateStake(stakeAddress, poolId);
    
    const unsignedTx = await tx.build();
    
    setState('signing');
    const signedTx = await wallet.signTx(unsignedTx);
    
    setState('submitting');
    const provider = createProvider({ network });
    const txHash = await provider.submitTx(signedTx);
    
    setTxHash(txHash);
    setState('success');
  } catch (error) {
    setError(error instanceof Error ? error.message : 'Delegation failed');
    setState('error');
  }
};
```

**Effort:** Low (2-3 days) - Infrastructure exists, just needs implementation
**Impact:** Medium - Required for complete wallet functionality

---

### 2.7 Stablecoin Payments (LOWER PRIORITY, FUTURE)

**Research Reference:** "Send stablecoins to contacts, pay invoices"

**Current State:** Partial - Send command exists but not stablecoin-optimized

**Future Implementation:**

```bash
# Payment request (generates QR/link)
begin pay request --amount 100 --currency USDA --memo "Invoice #123" [--json]

# Send to contact
begin pay send --to "alice.ada" --amount 50 --currency USDA

# Pay invoice (from link/QR)
begin pay invoice "cardano:pay?amount=100&token=USDA&to=addr1..."

# Payment history
begin pay history [--limit 20] [--json]
```

**Effort:** High (4+ weeks) - Requires contact system, invoice format spec
**Impact:** Medium - Natural extension but not core differentiator initially

---

## Part 3: Prioritized Roadmap

### Phase 1: Foundation (Weeks 1-2) ⚡
**Focus: Make stablecoin swaps excellent**

| Task | Effort | Files to Create/Modify |
|------|--------|------------------------|
| Add stablecoin token aliases | 2h | `lib/swap.ts` |
| Create stablecoin list command | 4h | `commands/stablecoins/list.tsx` |
| Add swap comparison output | 4h | `commands/swap/compare.tsx` |
| Implement real staking tx | 1d | `commands/stake/delegate.tsx` |
| Add tests for stablecoin swaps | 4h | `tests/stablecoin-swap.test.ts` |

**Deliverables:**
- `begin stablecoins list` - Show supported stablecoins with current balances
- Enhanced swap with stablecoin aliases working
- Real staking transactions (not mocked)

### Phase 2: Yield Intelligence (Weeks 3-4) 📊
**Focus: Build the yield aggregation layer**

| Task | Effort | Files to Create/Modify |
|------|--------|------------------------|
| Create YieldAggregator service | 3d | `services/yield-aggregator.ts` |
| Implement Liqwid adapter | 2d | `services/adapters/liqwid.ts` |
| Implement Lenfi adapter | 2d | `services/adapters/lenfi.ts` |
| Implement Minswap LP adapter | 2d | `services/adapters/minswap-lp.ts` |
| Create yield CLI commands | 2d | `commands/yield/*.tsx` |
| Add yield comparison UI | 1d | `commands/yield/compare.tsx` |

**Deliverables:**
- `begin yield list` - View all yield opportunities
- `begin yield best --asset USDA` - Find best rate
- `begin yield compare --assets USDA,USDM` - Comparison table

### Phase 3: Vault System (Weeks 5-8) 🏦
**Focus: Implement savings vault**

| Task | Effort | Files to Create/Modify |
|------|--------|------------------------|
| Design vault architecture | 2d | `lib/vault.ts` |
| Implement LiqwidDepositor | 3d | `services/depositors/liqwid.ts` |
| Implement LenfiDepositor | 3d | `services/depositors/lenfi.ts` |
| Implement MinswapLpDepositor | 3d | `services/depositors/minswap-lp.ts` |
| Create VaultService | 3d | `lib/vault.ts` |
| Create vault CLI commands | 3d | `commands/vault/*.tsx` |
| Add vault state persistence | 1d | `lib/vault-state.ts` |
| Testing & edge cases | 3d | `tests/vault.test.ts` |

**Deliverables:**
- `begin vault deposit` - Deposit to vault with strategy
- `begin vault status` - View positions and earnings
- `begin vault withdraw` - Withdraw from vault
- Three strategies: conservative, balanced, aggressive

### Phase 4: DCA & Advanced (Weeks 9-12) 🔄
**Focus: Automation and advanced features**

| Task | Effort | Files to Create/Modify |
|------|--------|------------------------|
| Implement DCA service | 3d | `lib/dca.ts` |
| Create DCA CLI commands | 2d | `commands/dca/*.tsx` |
| Add DCA state persistence | 1d | `lib/dca-state.ts` |
| Multi-DEX routing setup | 1w | `services/sundaeswap.ts`, `services/wingriders.ts` |
| Best-route comparison | 2d | `lib/swap.ts` update |
| Agent integration docs | 1d | `docs/dca-automation.md` |

**Deliverables:**
- `begin dca create/list/execute` - Full DCA functionality
- `begin swap compare` - Multi-DEX quote comparison
- Documentation for cron/agent integration

---

## Part 4: Technical Architecture

### Directory Structure (Proposed)

```
begin-cli/
├── src/
│   ├── cli.tsx
│   ├── app.tsx
│   ├── index.ts
│   │
│   ├── commands/
│   │   ├── cardano/           # Existing
│   │   ├── stake/             # Existing (needs tx fix)
│   │   ├── swap/              # Existing
│   │   ├── wallet/            # Existing
│   │   │
│   │   ├── stablecoins/       # NEW
│   │   │   └── list.tsx
│   │   │
│   │   ├── yield/             # NEW
│   │   │   ├── list.tsx
│   │   │   ├── best.tsx
│   │   │   ├── compare.tsx
│   │   │   └── history.tsx
│   │   │
│   │   ├── vault/             # NEW
│   │   │   ├── deposit.tsx
│   │   │   ├── status.tsx
│   │   │   ├── withdraw.tsx
│   │   │   ├── rebalance.tsx
│   │   │   └── history.tsx
│   │   │
│   │   └── dca/               # NEW
│   │       ├── create.tsx
│   │       ├── list.tsx
│   │       ├── status.tsx
│   │       ├── pause.tsx
│   │       ├── cancel.tsx
│   │       └── execute.tsx
│   │
│   ├── lib/
│   │   ├── wallet.ts          # Existing
│   │   ├── keystore.ts        # Existing
│   │   ├── transaction.ts     # Existing
│   │   ├── swap.ts            # Existing (extend)
│   │   ├── staking.ts         # Existing (fix)
│   │   │
│   │   ├── vault.ts           # NEW
│   │   ├── vault-state.ts     # NEW
│   │   ├── dca.ts             # NEW
│   │   ├── dca-state.ts       # NEW
│   │   └── stablecoins.ts     # NEW (token registry)
│   │
│   └── services/
│       ├── blockfrost.ts      # Existing
│       ├── minswap.ts         # Existing
│       │
│       ├── yield-aggregator.ts  # NEW
│       ├── sundaeswap.ts        # NEW
│       ├── wingriders.ts        # NEW
│       │
│       └── adapters/            # NEW
│           ├── liqwid.ts
│           ├── lenfi.ts
│           └── minswap-lp.ts
│
├── tests/
│   ├── wallet.test.ts         # Existing
│   ├── swap.test.ts           # Existing
│   │
│   ├── stablecoin-swap.test.ts  # NEW
│   ├── yield-aggregator.test.ts # NEW
│   ├── vault.test.ts            # NEW
│   └── dca.test.ts              # NEW
│
└── docs/
    ├── README.md
    ├── stablecoin-guide.md      # NEW
    ├── vault-strategies.md       # NEW
    └── dca-automation.md         # NEW
```

### State Persistence

All new features requiring state should use the existing config directory pattern:

```
~/.begin-cli/
├── config.json              # Existing
├── wallets/                 # Existing
│   └── *.json
│
├── vault-state.json         # NEW - User's vault positions
├── dca-plans.json           # NEW - DCA plan configurations
├── yield-cache.json         # NEW - Cached yield data (with TTL)
└── execution-history.json   # NEW - Transaction history for DCA/vault
```

---

## Part 5: Risk Assessment

### Technical Risks

| Risk | Mitigation |
|------|------------|
| Protocol API changes | Abstract behind adapters, version lock |
| Transaction failures | Retry logic, clear error messages, dry-run mode |
| Wallet security | Never log mnemonics, use existing encryption |
| State corruption | Atomic writes, backup before operations |

### Smart Contract Risks

| Risk | Mitigation |
|------|------------|
| Protocol exploits | Only integrate audited protocols |
| Rug pulls | TVL thresholds, protocol age requirements |
| IL on LPs | Clear warnings, conservative strategy default |

### Regulatory Risks

| Risk | Mitigation |
|------|------------|
| Custody classification | Non-custodial design, user holds keys |
| Securities issues | No yield guarantees, clear disclaimers |
| AML/KYC | Consider optional identity verification tier |

---

## Part 6: Success Metrics

### Phase 1 Targets
- [ ] 100+ stablecoin swaps executed via CLI
- [ ] Stablecoin swap success rate > 95%
- [ ] Real staking transactions working on mainnet

### Phase 2 Targets
- [ ] Yield data from 3+ protocols aggregated
- [ ] < 5 minute data freshness for APY rates
- [ ] Documentation for all yield commands

### Phase 3 Targets
- [ ] $10K+ total value in vault positions (test users)
- [ ] Vault operations success rate > 99%
- [ ] 3 strategies available and tested

### Phase 4 Targets
- [ ] 50+ active DCA plans
- [ ] DCA execution success rate > 98%
- [ ] Multi-DEX routing providing measurably better rates

---

## Appendix A: Stablecoin Token Registry

```typescript
// src/lib/stablecoins.ts

export interface Stablecoin {
  symbol: string;
  name: string;
  policyId: string;
  assetName: string;  // Hex encoded
  decimals: number;
  issuer: string;
  type: 'fiat-backed' | 'algorithmic' | 'synthetic';
  website: string;
  isVerified: boolean;
}

export const CARDANO_STABLECOINS: Record<string, Stablecoin> = {
  USDA: {
    symbol: 'USDA',
    name: 'Anzens USDA',
    policyId: 'c48cbb3d5e57ed56e276bc45f99ab39abe94e6cd7ac39fb402da47ad',
    assetName: '0014df105553444154455354',
    decimals: 6,
    issuer: 'Anzens',
    type: 'fiat-backed',
    website: 'https://anzens.io',
    isVerified: true,
  },
  USDM: {
    symbol: 'USDM',
    name: 'Mehen USDM',
    policyId: '...',
    assetName: '...',
    decimals: 6,
    issuer: 'Mehen/Moneta',
    type: 'fiat-backed',
    website: 'https://mehen.io',
    isVerified: true,
  },
  iUSD: {
    symbol: 'iUSD',
    name: 'Indigo iUSD',
    policyId: '...',
    assetName: '...',
    decimals: 6,
    issuer: 'Indigo Protocol',
    type: 'synthetic',
    website: 'https://indigoprotocol.io',
    isVerified: true,
  },
  DJED: {
    symbol: 'DJED',
    name: 'Djed Stablecoin',
    policyId: '...',
    assetName: '...',
    decimals: 6,
    issuer: 'COTI/IOG',
    type: 'algorithmic',
    website: 'https://djed.xyz',
    isVerified: true,
  },
};

// Helper to get token ID from symbol
export function getStablecoinTokenId(symbol: string): string | null {
  const stable = CARDANO_STABLECOINS[symbol.toUpperCase()];
  if (!stable) return null;
  return stable.policyId + stable.assetName;
}

// Helper to identify if a token ID is a known stablecoin
export function isKnownStablecoin(tokenId: string): boolean {
  return Object.values(CARDANO_STABLECOINS).some(
    s => s.policyId + s.assetName === tokenId
  );
}
```

---

## Appendix B: Sample Mobile UI Wireframes

### Vault Dashboard
```
┌─────────────────────────────────┐
│  ← Back          Vault          │
├─────────────────────────────────┤
│                                 │
│    💰 Total Value               │
│    $1,023.45                    │
│    ▲ +$23.45 (2.3%)            │
│                                 │
├─────────────────────────────────┤
│  Strategy: Conservative   [⚙]  │
│  APY: 8.2%                      │
├─────────────────────────────────┤
│                                 │
│  Positions                      │
│  ┌─────────────────────────┐   │
│  │ 🏦 Liqwid (USDA)        │   │
│  │    $1,023.45 | 8.2% APY │   │
│  └─────────────────────────┘   │
│                                 │
│  ┌──────────┐  ┌──────────┐    │
│  │ Deposit  │  │ Withdraw │    │
│  └──────────┘  └──────────┘    │
│                                 │
└─────────────────────────────────┘
```

### DCA Plan View
```
┌─────────────────────────────────┐
│  ← Back           DCA           │
├─────────────────────────────────┤
│                                 │
│  Buy ADA with USDA              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                 │
│  Amount: $50/week               │
│  Next: Feb 10, 2026 @ 9:00 AM  │
│                                 │
│  Progress: $450/$1000          │
│  [███████████░░░░░░] 45%       │
│                                 │
├─────────────────────────────────┤
│  Stats                          │
│  Total bought: 892.5 ADA        │
│  Avg price: $0.504              │
│  Current price: $0.52           │
│  P/L: +$14.28 (+3.2%)          │
│                                 │
├─────────────────────────────────┤
│  Recent Executions              │
│  • Feb 3: 50 USDA → 96.2 ADA   │
│  • Jan 27: 50 USDA → 98.1 ADA  │
│  • Jan 20: 50 USDA → 94.8 ADA  │
│                                 │
│  ┌──────────┐  ┌──────────┐    │
│  │  Pause   │  │  Cancel  │    │
│  └──────────┘  └──────────┘    │
│                                 │
└─────────────────────────────────┘
```

---

## Conclusion

This implementation plan provides a roadmap for Begin Wallet to become the primary stablecoin interface for Cardano. The phased approach allows for:

1. **Quick wins** - Stablecoin swap improvements can ship in days
2. **Core value** - Yield aggregation and vaults build the moat
3. **Retention** - DCA creates recurring engagement
4. **Expansion** - Multi-DEX routing improves over time

The existing codebase is solid - MeshJS integration, wallet management, and Minswap swap routing are all production-ready. The main work is building the yield/vault layer and connecting to additional protocols.

**Recommended Next Steps:**
1. Review this plan with the team
2. Validate Liqwid/Lenfi API availability
3. Start Phase 1 implementation
4. Set up monitoring for swap success rates

---

*Document prepared for Begin Wallet development team*
*Questions? Review with Francis for protocol integration priorities*
