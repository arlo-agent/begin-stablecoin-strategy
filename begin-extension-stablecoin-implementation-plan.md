# Begin Wallet Extension: Stablecoin Financial Services Implementation Plan

*Document created: 2026-02-03*
*Based on: Stablecoin Financial Services Research & Begin Extension Codebase Analysis*

---

## Executive Summary

This document provides a detailed technical implementation plan for building stablecoin financial services into the Begin Wallet browser extension. After analyzing both the strategic research and the existing codebase, I've identified **significant existing infrastructure** that can be leveraged, along with specific gaps that need to be filled.

### Key Findings

| Category | Status | Notes |
|----------|--------|-------|
| Stablecoin Definitions | ✅ Exists | `lib/chains.ts` - USDA, USDM, iUSD, DJED, USDC, USDT |
| Swap Infrastructure | ✅ Exists | DexHunter integration, multi-DEX routing |
| Lending/Borrowing | ✅ Exists | Liqwid integration in `views/invest/view-lend.tsx` |
| Yield Dashboard | 🟡 Partial | Market APYs shown, needs aggregation view |
| Savings Vaults | ❌ Missing | Auto-compound, strategy tiers not implemented |
| Stablecoin-Specific Swap | 🟡 Partial | Generic swap exists, needs stable-optimized UX |
| DCA Automation | ❌ Missing | No recurring purchase functionality |
| Unified Yield View | ❌ Missing | No cross-protocol yield comparison |

---

## Part 1: Architecture Overview

### Current Tech Stack
```
Framework:     React 18 + TypeScript
UI Libraries:  Ionic 7, MUI 5, Framer Motion
State:         Apollo Client (GraphQL), localforage (persistence)
Routing:       React Router 5 + Ionic Router
Build:         CRACO + Webpack
Platform:      Browser Extension + Capacitor (iOS/Android)
```

### Relevant Existing Components

| Component | Path | Purpose |
|-----------|------|---------|
| `IonViewContainer` | `components/ion-view-container.tsx` | Page wrapper with navigation |
| `AppAlert` | `components/app-alert.tsx` | Bottom sheet/drawer modal |
| `InvestListItem` | `components/invest-listitem.tsx` | Investment product cards |
| `BarChartAsset` | `components/bar-chart-asset.tsx` | Yield visualization charts |
| `NumberInput` | `components/NumberInput.tsx` | Token amount input |
| `DisplaySigner` | `components/display-signer.tsx` | Transaction signing UI |
| `WalletBalance` | `components/wallet-balance.tsx` | Balance display with currency |

### Relevant Existing Hooks

| Hook | Path | Purpose |
|------|------|---------|
| `useLiqwid` | `hooks/useLiqwid.ts` | Liqwid protocol integration |
| `useSwap` | `hooks/useSwap.ts` | DEX settings & configurations |
| `useMynth` | `hooks/useMynth.ts` | Cross-chain stablecoin bridge |
| `useTransaction` | `hooks/useTransaction.ts` | Tx building & signing |
| `useToken` | `hooks/useToken.ts` | Token data fetching |
| `usePrice` | `hooks/usePrice.ts` | Price data |

### Stablecoin Registry (lib/chains.ts)
```typescript
// Already defined stablecoins
export const stableTokens = [
  { token_id: 'c48cbb3d...', name: 'USDM' },
  { token_id: 'fe7c786a...', name: 'USDA' },
  { token_id: 'f66d78b4...', name: 'iUSD' },
  { token_id: '8db269c3...', name: 'DJED' },
  { token_id: '25c5de5f...', name: 'USDC' },
  { token_id: '25c5de5f...', name: 'USDT' },
];
```

---

## Part 2: Feature Implementation Breakdown

### Feature 1: Stablecoin Savings Vault (HIGH PRIORITY)

**Strategic Goal:** One-tap deposit → earn yield across Liqwid/FluidTokens with auto-compound

#### What Exists
- Liqwid lending integration (`view-lend.tsx`)
- Supply/withdraw transactions
- APY display and calculations
- Balance tracking with `netAPY`

#### What Needs Building

##### 1.1 Vault Strategy System
```typescript
// New file: src/models/vault.ts
interface VaultStrategy {
  id: 'conservative' | 'balanced' | 'aggressive';
  name: string;
  description: string;
  riskScore: 1 | 2 | 3;
  protocols: ProtocolAllocation[];
  expectedAPY: { min: number; max: number };
  rebalanceFrequency: 'daily' | 'weekly';
}

interface ProtocolAllocation {
  protocol: 'liqwid' | 'fluidtokens' | 'minswap';
  percentage: number;
  stablecoins: string[]; // token_ids
}

// Conservative: 100% Liqwid lending (audited, highest TVL)
// Balanced: 70% Liqwid, 30% FluidTokens
// Aggressive: 50% Liqwid, 30% FluidTokens, 20% LP positions
```

##### 1.2 New Hook: useVault
```typescript
// New file: src/hooks/useVault.ts
const useVault = () => {
  const { getSupplyTransctionLQ, getWithdrawTransctionLQ, getMarketsLQ } = useLiqwid();
  
  return {
    getStrategies: () => VaultStrategy[],
    getVaultBalance: (address: string) => Promise<VaultBalance>,
    deposit: (strategy: string, amount: number, stablecoin: string) => Promise<string>,
    withdraw: (strategy: string, amount: number) => Promise<string>,
    getProjectedYield: (amount: number, strategy: string, days: number) => YieldProjection,
    autoCompound: () => Promise<void>, // Claims and reinvests rewards
  };
};
```

##### 1.3 Vault UI Components

**New View: `views/vault/index.tsx`**
```
┌─────────────────────────────────────────┐
│ ← Savings Vault                    [?]  │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  Your Vault Balance             │   │
│  │  $1,234.56                      │   │
│  │  ↑ +$12.34 this month (1.2%)   │   │
│  │  [███████████░░░░] 3.2% APY     │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ Strategy: Conservative          │   │
│  │ 100% Liqwid Lending             │   │
│  │ Risk: ●○○  Expected: 2-4% APY   │   │
│  │                    [Change →]   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Allocation                             │
│  ├── USDM (Liqwid)  60%  $740.74   │   │
│  ├── USDA (Liqwid)  25%  $308.64   │   │
│  └── iUSD (Liqwid)  15%  $185.18   │   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 📈 Projected 1-Year Earnings    │   │
│  │     [Bar chart showing growth]  │   │
│  └─────────────────────────────────┘   │
│                                         │
├─────────────────────────────────────────┤
│  [  Deposit  ]    [  Withdraw  ]       │
└─────────────────────────────────────────┘
```

**Deposit Bottom Sheet (reuse AppAlert pattern):**
```
┌─────────────────────────────────────────┐
│ ────                                    │  (drag handle)
│                                         │
│  Deposit to Vault                       │
│                                         │
│  Select Stablecoin                      │
│  ┌─────────────────────────────────┐   │
│  │ [USDM] [USDA] [iUSD] [DJED]    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Amount                                 │
│  ┌─────────────────────────────────┐   │
│  │  $ 500.00          [MAX]       │   │
│  │  Balance: 1,234.56 USDM        │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Strategy                               │
│  ○ Conservative (2-4% APY)              │
│  ● Balanced (4-6% APY)                  │
│  ○ Aggressive (6-10% APY)               │
│                                         │
│  Est. Annual Earnings: $25.00 - $50.00  │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │        [ Deposit $500 ]        │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

##### 1.4 Implementation Steps

1. **Create vault data models** (`src/models/vault.ts`)
2. **Build useVault hook** leveraging existing `useLiqwid`
3. **Add FluidTokens integration** (new hook `useFluidTokens.ts`)
4. **Create VaultCard component** for home screen
5. **Build full vault view** with strategy selection
6. **Implement deposit/withdraw bottom sheets**
7. **Add auto-compound cron job** (background service)

**Effort Estimate:** 3-4 weeks
**Impact:** HIGH - Stickiest feature, drives AUM

---

### Feature 2: Stablecoin Swap Router (HIGH PRIORITY)

**Strategic Goal:** Best-execution routing specifically for stablecoin-to-stablecoin swaps

#### What Exists
- Full swap implementation (`views/swap/index.tsx`)
- Multi-DEX support via DexHunter API
- Slippage settings
- Order history

#### What Needs Building

##### 2.1 Stablecoin-Optimized Swap View

The current swap view is generic. Create a specialized stablecoin swap with:
- Preset 0.1% slippage (stables have minimal volatility)
- Visual depeg warning if any stable is off-peg
- Quick-swap shortcuts (e.g., "Swap all USDA to USDM")
- Fee comparison across DEXes

**New View: `views/swap/stable-swap.tsx`**
```
┌─────────────────────────────────────────┐
│ ← Stablecoin Swap                       │
├─────────────────────────────────────────┤
│                                         │
│  From                                   │
│  ┌─────────────────────────────────┐   │
│  │ [USDM ▼]           1,000.00    │   │
│  │ Balance: 1,234.56              │   │
│  │ Peg: $1.001 ✓                  │   │
│  └─────────────────────────────────┘   │
│                        ↕                │
│  To                                     │
│  ┌─────────────────────────────────┐   │
│  │ [USDA ▼]           ≈ 999.80    │   │
│  │ Peg: $0.998 ⚠️ (slight depeg)   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Route                                  │
│  ┌─────────────────────────────────┐   │
│  │ Minswap V2 • 0.05% fee          │   │
│  │ Best rate of 4 DEXes checked    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Price Impact: <0.01%                   │
│  Network Fee: ~0.2 ADA                  │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │      [ Swap 1,000 USDM ]       │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

##### 2.2 Depeg Monitoring Hook

```typescript
// New file: src/hooks/useStablePeg.ts
interface PegStatus {
  token: string;
  currentPrice: number;
  pegTarget: 1.0;
  deviation: number;
  status: 'healthy' | 'slight-depeg' | 'significant-depeg';
  lastUpdate: Date;
}

const useStablePeg = () => {
  return {
    getPegStatus: (tokenId: string) => Promise<PegStatus>,
    getAllPegStatuses: () => Promise<PegStatus[]>,
    subscribeToDepegAlerts: (threshold: number, callback: Function) => void,
  };
};
```

##### 2.3 Arbitrage Opportunity Indicator

When stables have different pegs, show arbitrage opportunity:
```
┌─────────────────────────────────────────┐
│ 💡 Arbitrage Opportunity                │
│ USDM is +0.3% above USDA peg            │
│ Swap now to gain ~$3.00 on $1,000       │
│                         [Quick Swap →]  │
└─────────────────────────────────────────┘
```

##### 2.4 Implementation Steps

1. **Create `stable-swap.tsx`** view (fork from existing swap)
2. **Build `useStablePeg` hook** with price feed
3. **Add stablecoin filter** to token selector
4. **Implement route comparison UI** showing all DEX options
5. **Add "Quick Swap" shortcuts** on wallet view
6. **Create depeg alert notifications**

**Effort Estimate:** 2 weeks
**Impact:** HIGH - Frequent use case, competitive differentiator

---

### Feature 3: Yield Dashboard (MEDIUM PRIORITY)

**Strategic Goal:** Real-time view of all yield opportunities across Cardano DeFi

#### What Exists
- Liqwid market data (`getMarketsLQ`)
- APY calculations in `view-lend.tsx`
- Basic bar charts

#### What Needs Building

##### 3.1 Yield Aggregation Hook

```typescript
// New file: src/hooks/useYieldAggregator.ts
interface YieldOpportunity {
  protocol: string;
  protocolLogo: string;
  type: 'lending' | 'lp' | 'staking' | 'vault';
  token: string;
  apy: number;
  apyBreakdown: {
    base: number;
    rewards: number;
    total: number;
  };
  tvl: number;
  risk: 'low' | 'medium' | 'high';
  audited: boolean;
  minDeposit: number;
  url: string;
}

const useYieldAggregator = () => {
  const { getMarketsLQ } = useLiqwid();
  // Add more protocol hooks as integrated
  
  return {
    getAllOpportunities: () => Promise<YieldOpportunity[]>,
    getByToken: (tokenId: string) => Promise<YieldOpportunity[]>,
    getByRisk: (risk: string) => Promise<YieldOpportunity[]>,
    getHistoricalAPY: (protocolId: string, days: number) => Promise<APYHistory[]>,
  };
};
```

##### 3.2 Yield Dashboard View

**New View: `views/yield/index.tsx`**
```
┌─────────────────────────────────────────┐
│ ← Yield Opportunities                   │
├─────────────────────────────────────────┤
│  Filter: [All ▼] [Stables ▼] [Low Risk] │
├─────────────────────────────────────────┤
│                                         │
│  Top Yields                             │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🏦 Liqwid • USDM Lending        │   │
│  │ 4.2% APY         TVL: $12.5M    │   │
│  │ Risk: Low ●○○    Audited ✓      │   │
│  │                     [Deposit →] │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🌊 Minswap • USDM/USDA LP       │   │
│  │ 6.8% APY         TVL: $3.2M     │   │
│  │ Risk: Medium ●●○  Audited ✓     │   │
│  │                     [Provide →] │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 📈 FluidTokens • USDA Lending         │   │
│  │ 3.8% APY         TVL: $2.1M     │   │
│  │ Risk: Low ●○○    Audited ✓      │   │
│  │                     [Deposit →] │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Historical APY Comparison              │
│  ┌─────────────────────────────────┐   │
│  │    [Line chart: 30-day APYs]    │   │
│  │    --- Liqwid  --- FluidTokens        │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

##### 3.3 Yield Card for Home Screen

Add to the main wallet view (similar to existing stables grouping):
```typescript
// In views/wallet/index.tsx, add yield summary card
<YieldSummaryCard
  totalDeposited={vaultBalance}
  currentAPY={weightedAPY}
  monthlyEarnings={projectedMonthly}
  onPress={() => history.push('/wallet/yield')}
/>
```

##### 3.4 Implementation Steps

1. **Create `useYieldAggregator` hook**
2. **Build protocol adapter interfaces** for extensibility
3. **Create yield dashboard view**
4. **Add `YieldOpportunityCard` component**
5. **Build historical APY charts** (extend BarChartAsset)
6. **Add yield summary to home screen**

**Effort Estimate:** 2-3 weeks
**Impact:** MEDIUM - Information drives conversions to vault

---

### Feature 4: Dollar-Cost Averaging (MEDIUM PRIORITY)

**Strategic Goal:** Scheduled recurring buys into ADA or stablecoins

#### What Exists
- Transaction building infrastructure
- Basic scheduling via background extension

#### What Needs Building

##### 4.1 DCA Data Model

```typescript
// New file: src/models/dca.ts
interface DCASchedule {
  id: string;
  status: 'active' | 'paused' | 'completed';
  fromToken: string;  // Usually a stablecoin
  toToken: string;    // ADA, BTC, or other
  amount: number;
  frequency: 'daily' | 'weekly' | 'biweekly' | 'monthly';
  nextExecution: Date;
  executionHistory: DCAExecution[];
  totalInvested: number;
  totalAcquired: number;
  averagePrice: number;
  createdAt: Date;
}

interface DCAExecution {
  date: Date;
  amountIn: number;
  amountOut: number;
  price: number;
  txHash: string;
  status: 'success' | 'failed' | 'pending';
}
```

##### 4.2 DCA Hook

```typescript
// New file: src/hooks/useDCA.ts
const useDCA = () => {
  return {
    createSchedule: (params: DCAParams) => Promise<DCASchedule>,
    pauseSchedule: (id: string) => Promise<void>,
    resumeSchedule: (id: string) => Promise<void>,
    cancelSchedule: (id: string) => Promise<void>,
    getSchedules: () => Promise<DCASchedule[]>,
    getExecutionHistory: (id: string) => Promise<DCAExecution[]>,
    executeNow: (id: string) => Promise<DCAExecution>,  // Manual trigger
  };
};
```

##### 4.3 DCA View

**New View: `views/dca/index.tsx`**
```
┌─────────────────────────────────────────┐
│ ← Auto-Invest (DCA)                 [+] │
├─────────────────────────────────────────┤
│                                         │
│  Active Schedules                       │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ USDM → ADA                      │   │
│  │ $50 weekly • Next: Mon, Feb 10  │   │
│  │                                 │   │
│  │ Total Invested: $450            │   │
│  │ ADA Acquired: 1,234.56          │   │
│  │ Avg Price: $0.365               │   │
│  │                                 │   │
│  │ [Pause]     [Edit]    [View →] │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Performance                            │
│  ┌─────────────────────────────────┐   │
│  │  [Chart: DCA vs lump sum]       │   │
│  │  Your DCA: +12.3% vs market     │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

**Create DCA Bottom Sheet:**
```
┌─────────────────────────────────────────┐
│ Create Auto-Invest Schedule             │
├─────────────────────────────────────────┤
│                                         │
│  From (Funding)                         │
│  ┌─────────────────────────────────┐   │
│  │ [USDM ▼]                        │   │
│  └─────────────────────────────────┘   │
│                                         │
│  To (Buying)                            │
│  ┌─────────────────────────────────┐   │
│  │ [ADA ▼]                         │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Amount per purchase                    │
│  ┌─────────────────────────────────┐   │
│  │ $ [50]                          │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Frequency                              │
│  [Daily] [Weekly] [Bi-weekly] [Monthly] │
│              ↑ selected                 │
│                                         │
│  Preview: Buy ~125 ADA every week       │
│  Est. annual investment: $2,600         │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │     [ Start Auto-Invest ]      │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

##### 4.4 Background Execution Service

```typescript
// New file: src/services/dcaExecutor.ts
// Runs via extension alarms API
chrome.alarms.create('dca-executor', { periodInMinutes: 60 });

chrome.alarms.onAlarm.addListener(async (alarm) => {
  if (alarm.name === 'dca-executor') {
    const schedules = await getDueSchedules();
    for (const schedule of schedules) {
      await executeDCAOrder(schedule);
    }
  }
});
```

##### 4.5 Implementation Steps

1. **Create DCA data models and storage**
2. **Build `useDCA` hook**
3. **Implement background executor service**
4. **Create DCA management view**
5. **Add create schedule bottom sheet**
6. **Build performance tracking charts**
7. **Add notification on execution**

**Effort Estimate:** 2-3 weeks
**Impact:** MEDIUM - Recurring engagement, steady AUM growth

---

### Feature 5: Enhanced Stablecoin Wallet View (LOW EFFORT, HIGH VISIBILITY)

**Strategic Goal:** Position stablecoins prominently, show yield opportunities inline

#### What Exists
- Wallet view groups stablecoins as "Cash" (`views/wallet/index.tsx`)
- Basic stablecoin balance display

#### What Needs Building

##### 5.1 Enhanced Cash Section

Transform the existing "stables" grouping into a prominent card:

```
┌─────────────────────────────────────────┐
│  💵 Cash Balance                    [→] │
│                                         │
│  Total: $1,234.56                       │
│                                         │
│  ┌────────┐ ┌────────┐ ┌────────┐      │
│  │  USDM  │ │  USDA  │ │  iUSD  │      │
│  │ $500   │ │ $450   │ │ $284   │      │
│  │ 3.2%↗  │ │ 2.8%↗  │ │ 4.1%↗  │      │
│  └────────┘ └────────┘ └────────┘      │
│                                         │
│  💡 Earn 3.5% APY on idle stables       │
│                      [Start Earning →]  │
└─────────────────────────────────────────┘
```

##### 5.2 Quick Actions for Stablecoins

Add swipe actions or long-press menu:
- Quick Swap (to another stable)
- Deposit to Vault
- Send
- View Details

##### 5.3 Implementation Steps

1. **Enhance `stables` rendering** in wallet view
2. **Add inline APY display** per stablecoin
3. **Create "Start Earning" CTA** linking to vault
4. **Add quick action gestures**

**Effort Estimate:** 1 week
**Impact:** HIGH visibility - Users see opportunity immediately

---

## Part 3: Component Library Additions

### New Shared Components Needed

```typescript
// 1. Risk Indicator
// src/components/risk-indicator.tsx
<RiskIndicator level="low" /> // Renders: ●○○

// 2. APY Badge
// src/components/apy-badge.tsx  
<APYBadge value={4.2} trend="up" /> // Renders: 4.2% ↗

// 3. Strategy Card
// src/components/strategy-card.tsx
<StrategyCard
  name="Conservative"
  description="100% Liqwid lending"
  riskLevel="low"
  apyRange={{ min: 2, max: 4 }}
  selected={true}
  onSelect={() => {}}
/>

// 4. Token Amount Input (Enhanced)
// src/components/token-amount-input.tsx
<TokenAmountInput
  token={selectedToken}
  value={amount}
  onValueChange={setAmount}
  showBalance={true}
  showUSDValue={true}
  showMaxButton={true}
/>

// 5. Yield Chart
// src/components/yield-chart.tsx
<YieldChart
  data={historicalAPYs}
  tokens={['USDM', 'USDA']}
  timeRange="30d"
/>

// 6. Protocol Badge
// src/components/protocol-badge.tsx
<ProtocolBadge
  name="Liqwid"
  logo={liqwidLogo}
  audited={true}
  tvl={12500000}
/>
```

---

## Part 4: Navigation & Routing Updates

### New Routes Needed

```typescript
// In views/main.tsx, add routes:

// Vault
<Route path="/wallet/vault" component={Vault} />
<Route path="/wallet/vault/deposit" component={VaultDeposit} />
<Route path="/wallet/vault/withdraw" component={VaultWithdraw} />
<Route path="/wallet/vault/strategy" component={VaultStrategy} />

// Stable Swap
<Route path="/wallet/swap/stable" component={StableSwap} />

// Yield Dashboard
<Route path="/wallet/yield" component={YieldDashboard} />
<Route path="/wallet/yield/:protocol" component={YieldProtocolDetail} />

// DCA
<Route path="/wallet/dca" component={DCAManager} />
<Route path="/wallet/dca/create" component={DCACreate} />
<Route path="/wallet/dca/:id" component={DCADetail} />
```

### Navigation Entry Points

| Entry Point | Target | Location |
|-------------|--------|----------|
| Invest tab → "Savings Vault" | `/wallet/vault` | `views/invest/index.tsx` |
| Wallet Cash section CTA | `/wallet/vault/deposit` | `views/wallet/index.tsx` |
| Swap → "Stablecoin Swap" | `/wallet/swap/stable` | `views/swap/index.tsx` |
| Invest tab → "Yield Dashboard" | `/wallet/yield` | `views/invest/index.tsx` |
| Settings → "Auto-Invest" | `/wallet/dca` | `views/user-settings/index.tsx` |

---

## Part 5: Backend/API Requirements

### New API Endpoints Needed

| Endpoint | Purpose | Provider |
|----------|---------|----------|
| `/api/yields` | Aggregated yield data | Begin backend (new) |
| `/api/yields/:protocol` | Protocol-specific yields | Begin backend (new) |
| `/api/pegs` | Stablecoin peg status | Begin backend (new) |
| `/api/dca/schedules` | DCA schedule storage | Begin backend (new) |
| FluidTokens GraphQL | FluidTokens protocol data | FluidTokens (integration needed) |

### Existing APIs to Leverage

| API | Current Use | Extended Use |
|-----|-------------|--------------|
| Liqwid GraphQL | Lending markets | Yield aggregation |
| DexHunter | Swap routing | Stable-specific routes |
| Maestro | UTxO queries | DCA execution |

---

## Part 6: Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)

| Week | Deliverables |
|------|--------------|
| 1 | Enhanced wallet stablecoin view, useStablePeg hook |
| 2 | Stable-optimized swap view |
| 3 | useYieldAggregator hook, basic yield dashboard |
| 4 | Testing, refinement, integration |

### Phase 2: Vault MVP (Weeks 5-8)

| Week | Deliverables |
|------|--------------|
| 5 | Vault data models, useVault hook |
| 6 | Vault deposit/withdraw UI |
| 7 | Strategy selection, projections |
| 8 | Auto-compound service, testing |

### Phase 3: DCA & Polish (Weeks 9-12)

| Week | Deliverables |
|------|--------------|
| 9 | DCA data models, useDCA hook |
| 10 | DCA management UI |
| 11 | Background execution service |
| 12 | Full testing, bug fixes, launch prep |

---

## Part 7: Effort vs Impact Matrix

```
                    IMPACT
                    HIGH
                      │
     ┌────────────────┼────────────────┐
     │                │                │
     │  Wallet Cash   │  Savings       │
     │  Enhancement   │  Vault         │
     │  (1 week)      │  (4 weeks)     │
     │                │                │
LOW ─┼────────────────┼────────────────┼─ HIGH
     │                │                │  EFFORT
     │  Yield         │  DCA           │
     │  Dashboard     │  Automation    │
     │  (3 weeks)     │  (3 weeks)     │
     │                │                │
     └────────────────┼────────────────┘
                      │
                    LOW
```

### Recommended Priority Order

1. **Wallet Cash Enhancement** - Low effort, high visibility, immediate impact
2. **Stable Swap Optimization** - Builds on existing code, high-frequency use
3. **Yield Dashboard** - Information drives action, needed for vault
4. **Savings Vault** - Core differentiator, requires yield dashboard
5. **DCA Automation** - Nice-to-have, builds recurring engagement

---

## Part 8: Technical Considerations

### State Management

```typescript
// Consider adding vault-specific Apollo cache
import { makeVar } from '@apollo/client';

export const vaultBalanceVar = makeVar<VaultBalance | null>(null);
export const yieldOpportunitiesVar = makeVar<YieldOpportunity[]>([]);
export const dcaSchedulesVar = makeVar<DCASchedule[]>([]);
```

### Error Handling

```typescript
// Standardize DeFi error handling
interface DeFiError {
  type: 'slippage' | 'insufficient-funds' | 'protocol-error' | 'network';
  message: string;
  recoverable: boolean;
  suggestedAction?: string;
}

const handleDeFiError = (error: DeFiError) => {
  showMessage(error.message, error.recoverable ? 'warning' : 'error');
  if (error.suggestedAction) {
    // Show action button
  }
};
```

### Security Considerations

1. **Transaction Preview** - Always show full transaction details before signing
2. **Protocol Verification** - Display audit status and TVL for all protocols
3. **Slippage Protection** - Default to conservative slippage for vaults
4. **DCA Spending Limits** - Set maximum per-execution and total limits
5. **Emergency Withdrawal** - Always allow users to exit positions

### Testing Strategy

1. **Unit Tests** - All hooks and utility functions
2. **Integration Tests** - Protocol interactions (mock API responses)
3. **E2E Tests** - Full user flows (Cypress)
4. **Testnet Deployment** - Full testing on Cardano Preprod

---

## Part 9: Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Vault AUM | $500K in 6 months | On-chain tracking |
| Stable Swaps | 1,000/month | Transaction logs |
| DCA Active Users | 500 schedules | Backend metrics |
| Yield Dashboard DAU | 20% of active users | Analytics |
| Vault Retention | 80% after 30 days | Cohort analysis |

---

## Appendix A: File Structure

```
src/
├── hooks/
│   ├── useVault.ts          (NEW)
│   ├── useYieldAggregator.ts (NEW)
│   ├── useStablePeg.ts      (NEW)
│   ├── useDCA.ts            (NEW)
│   ├── useFluidTokens.ts          (NEW)
│   └── ... existing hooks
├── models/
│   ├── vault.ts             (NEW)
│   └── dca.ts               (NEW)
├── views/
│   ├── vault/
│   │   ├── index.tsx        (NEW)
│   │   ├── deposit.tsx      (NEW)
│   │   ├── withdraw.tsx     (NEW)
│   │   └── strategy.tsx     (NEW)
│   ├── yield/
│   │   ├── index.tsx        (NEW)
│   │   └── protocol.tsx     (NEW)
│   ├── dca/
│   │   ├── index.tsx        (NEW)
│   │   ├── create.tsx       (NEW)
│   │   └── detail.tsx       (NEW)
│   └── swap/
│       ├── index.tsx        (existing)
│       └── stable-swap.tsx  (NEW)
├── components/
│   ├── risk-indicator.tsx   (NEW)
│   ├── apy-badge.tsx        (NEW)
│   ├── strategy-card.tsx    (NEW)
│   ├── yield-chart.tsx      (NEW)
│   └── protocol-badge.tsx   (NEW)
└── services/
    └── dcaExecutor.ts       (NEW)
```

---

## Appendix B: Design Tokens

To maintain consistency with existing Begin UI:

```typescript
// Recommended additions to theme
const stablecoinTheme = {
  colors: {
    vault: {
      primary: '#3414FC',      // Begin purple
      success: '#39AE2E',       // Green for positive APY
      warning: '#F5A623',       // Orange for depegs
      background: '#1A1A2E',    // Dark card background
    }
  },
  borders: {
    card: '12px',               // Match existing roundness
  },
  spacing: {
    cardPadding: 16,
    sectionGap: 24,
  }
};
```

---

*End of Implementation Plan*

**Next Steps:**
1. Review with development team
2. Prioritize Phase 1 features
3. Create Jira/Linear tickets
4. Begin implementation of wallet cash enhancement
