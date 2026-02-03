# Begin Wallet Premium Strategy

*Document created: 2026-02-03*
*Author: Research Analysis*
*Status: Ready for Implementation*

---

## Executive Summary

**Positioning: "Begin — Your Stablecoin Home Onchain"**

Begin isn't another multi-chain wallet. It's the **onchain complement to your bank** — focused on stablecoins, yield, and financial services. This document outlines a premium subscription strategy based on:
1. Competitor analysis (CoinStats, Zerion, Ledger, Exodus)
2. Current Begin codebase capabilities (b58-extension analysis)
3. Stablecoin home market opportunity ($84B+ by 2026)
4. User acquisition economics

**Key Recommendation:** Launch a 3-tier subscription model (Free / Pro / Enterprise) with **Begin Pro at $9.99/month** ($7.99/month billed annually), monetizing features that are either already built or can be implemented within 30 days.

**Conservative Revenue Projection:** $50,000-150,000 ARR in Year 1 with 500-1,500 Pro subscribers.

---

## Part 1: Competitive Analysis

### Direct Competitors

| Product | Free Tier | Pro Tier | Premium/Enterprise | Lifetime Option |
|---------|-----------|----------|-------------------|-----------------|
| **CoinStats** | 10 portfolios, 20K tx | $13.99/mo (yearly) | $62.91/mo (Degen) | $399 Premium, $2,600 Degen |
| **Zerion** | Basic tracking | NFT-based (DNA) | Fee discounts, P&L, CSV | 12-month subscriptions |
| **Exodus** | Basic wallet | Built-in swap fees 0.5% | N/A | N/A |
| **Ledger Recover** | N/A | $9.99/mo | Seed recovery service | N/A |

### Key Insights

1. **Price Anchoring:** CoinStats at $13.99/mo sets a ceiling; Ledger Recover at $9.99/mo is a floor for premium crypto services
2. **Value Drivers:** P&L tracking, CSV export (tax), advanced analytics consistently appear in premium tiers
3. **User Acquisition Cost:** Industry benchmark $14-31/active trader means lifetime value must exceed ~$50 to be profitable
4. **Lifetime Options:** Popular at $399+ — creates one-time revenue boost and locks in users

### Begin's Competitive Advantages

| Advantage | Description | Premium Opportunity |
|-----------|-------------|---------------------|
| **Stablecoin-First** | Built around USDC, USDT, USDA, USDM | Savings vaults, yield optimization |
| **Bank-Grade UX** | Clean, simple, no crypto jargon | "Feels like home, earns like DeFi" |
| **Yield Aggregation** | Best rates across Liqwid, FluidTokens, LPs | Auto-compound, set-and-forget savings |
| **Multi-Chain Coverage** | Solana + Cardano + Bitcoin under the hood | Users don't need to think about chains |
| **DCA & Automation** | Recurring buys, automated strategies | "Savings account that grows itself" |
| **Self-Custody Security** | Your keys, your coins — but easy | Home convenience + DeFi ownership |

---

## Part 2: Tier Structure

### Tier Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           BEGIN WALLET TIERS                                │
├─────────────────┬─────────────────────┬─────────────────────────────────────┤
│    FREE         │      PRO            │         ENTERPRISE                  │
│    $0/mo        │    $9.99/mo         │        $49.99/mo                    │
│                 │  ($7.99/mo annual)  │     ($39.99/mo annual)              │
├─────────────────┼─────────────────────┼─────────────────────────────────────┤
│ • Basic wallet  │ • Everything Free + │ • Everything Pro +                  │
│ • Send/receive  │ • P&L tracking      │ • API access                        │
│ • Basic swap    │ • CSV tax export    │ • Multi-wallet (10+)                │
│ • 3 wallets     │ • Best route PRO    │ • White-label option                │
│ • Basic DeFi    │ • 10 wallets        │ • Dedicated support                 │
│ • Community     │ • Price alerts (∞)  │ • Custom integrations               │
│   support       │ • Yield optimizer   │ • SLA guarantees                    │
│                 │ • Priority support  │ • Team management                   │
│                 │ • Auto-compound*    │                                     │
│                 │ • DCA automation*   │                                     │
└─────────────────┴─────────────────────┴─────────────────────────────────────┘
                        * Coming in v2.5
```

### Pricing Justification

| Factor | Analysis | Impact on Pricing |
|--------|----------|-------------------|
| CoinStats benchmark | $13.99/mo for similar features | Sets ceiling |
| Ledger Recover | $9.99/mo for single feature | Sets floor for "premium feel" |
| Cardano user base | ~growing multi-chain user base (Cardano + Bitcoin + Solana) | Growing market = premium positioning possible |
| User acquisition cost | $14-31 per trader | Need LTV > $50 = 5+ months retention |
| Competitor-free zone | No stablecoin home alternatives | First-mover advantage |

**Recommended: $9.99/mo Pro** — below CoinStats, at Ledger level, achievable LTV with 6-month retention.

### Lifetime Options

| Tier | Lifetime Price | Break-Even | Rationale |
|------|---------------|------------|-----------|
| Pro Lifetime | $199 | 20 months | Lower than CoinStats ($399), impulse-buy friendly |
| Enterprise Lifetime | $999 | 20 months | For DAOs, funds, power users |

---

## Part 3: Feature Matrix

### Detailed Feature Breakdown

| Feature | Free | Pro | Enterprise | Implementation Status |
|---------|------|-----|------------|----------------------|
| **WALLET CORE** |
| Multi-chain wallet (ADA, BTC) | ✅ | ✅ | ✅ | ✅ Built |
| Hardware wallet (Ledger, Keystone) | ✅ | ✅ | ✅ | ✅ Built |
| Biometric unlock (mobile) | ✅ | ✅ | ✅ | ✅ Built |
| Wallet limit | 3 | 10 | Unlimited | 🟡 Config change |
| **TRADING** |
| Basic swap (single DEX) | ✅ | ✅ | ✅ | ✅ Built |
| Multi-DEX routing | Basic | **Best Route PRO** | ✅ | ✅ Built, needs gate |
| Slippage optimization | ❌ | ✅ | ✅ | 🟡 2 days |
| Trade history | 30 days | Unlimited | Unlimited | 🟡 Config change |
| **DEFI** |
| Liqwid lend/borrow | ✅ | ✅ | ✅ | ✅ Built |
| FluidTokens boosted | ✅ | ✅ | ✅ | ✅ Built |
| Yield comparison | Basic | **Optimizer PRO** | ✅ | 🟡 1 week |
| Auto-compound | ❌ | ✅ | ✅ | 🔴 Phase 2 |
| **ANALYTICS** |
| Basic balance view | ✅ | ✅ | ✅ | ✅ Built |
| P&L tracking | ❌ | ✅ | ✅ | 🟡 2 weeks |
| Portfolio history | 7 days | 1 year | Unlimited | 🟡 Config + backend |
| **TAX & EXPORT** |
| Transaction history | ✅ | ✅ | ✅ | ✅ Built |
| CSV export | ❌ | ✅ | ✅ | 🟡 3 days |
| Tax report format | ❌ | ✅ | ✅ | 🟡 1 week |
| **ALERTS** |
| Price alerts | 3 | Unlimited | Unlimited | ✅ Built |
| Depeg alerts | ❌ | ✅ | ✅ | 🟡 3 days |
| Yield alerts | ❌ | ✅ | ✅ | 🟡 3 days |
| **AUTOMATION** |
| DCA scheduling | ❌ | ✅ | ✅ | 🔴 Phase 2 |
| Auto-rebalance | ❌ | ❌ | ✅ | 🔴 Phase 3 |
| **SUPPORT** |
| Community (Discord) | ✅ | ✅ | ✅ | ✅ Exists |
| Priority email | ❌ | ✅ | ✅ | 🟡 Process |
| Dedicated support | ❌ | ❌ | ✅ | 🟡 Process |
| **ENTERPRISE** |
| API access | ❌ | ❌ | ✅ | 🔴 Phase 3 |
| Team management | ❌ | ❌ | ✅ | 🔴 Phase 3 |
| Custom branding | ❌ | ❌ | ✅ | 🔴 Phase 3 |

### Legend
- ✅ Built — Feature exists in codebase
- 🟡 Quick win — Can implement in 1-2 weeks
- 🔴 Phase 2/3 — Requires significant development

---

## Part 4: Launch-Ready Features (Day 1)

These features exist in the codebase and can be gated behind subscription with minimal effort:

### 4.1 Multi-DEX "Best Route PRO"

**Current State:** DexHunter integration queries 14+ DEXs (SundaeSwap V3, Minswap V2, Splash, WingRiders V2, etc.)

**Premium Gate:**
- Free: Single DEX swap (user's default choice)
- Pro: "Best Route" shows comparison across all DEXs, auto-selects optimal

**Implementation:**
```typescript
// useSwap.ts already has dexes object with 14+ DEXs
// Add subscription check before showing route comparison
const showBestRoute = user.subscription === 'pro' || user.subscription === 'enterprise';
```

**Effort:** 1-2 days (UI gating + subscription check)

### 4.2 Unlimited Price Alerts

**Current State:** `usePriceAlert.ts` hook exists with full alert functionality

**Premium Gate:**
- Free: 3 active alerts
- Pro: Unlimited alerts

**Implementation:**
```typescript
// Add alert count check in usePriceAlert.ts
const MAX_FREE_ALERTS = 3;
if (alerts.length >= MAX_FREE_ALERTS && !isPro) {
  throw new Error('Upgrade to Pro for unlimited alerts');
}
```

**Effort:** 2 hours

### 4.3 Extended Wallet Support

**Current State:** Multi-wallet support exists in account management

**Premium Gate:**
- Free: 3 wallets
- Pro: 10 wallets
- Enterprise: Unlimited

**Implementation:** Configuration change in account creation flow

**Effort:** 4 hours

### 4.4 Extended Trade History

**Current State:** Transaction history views exist

**Premium Gate:**
- Free: 30 days
- Pro: 1 year
- Enterprise: Unlimited

**Implementation:** Query parameter filtering based on subscription

**Effort:** 4 hours

---

## Part 5: Near-Term Features (1-3 Months)

### 5.1 P&L Tracking (Pro Feature) — 2 weeks

**What:** Real-time profit/loss calculation across portfolio

**Components Needed:**
1. Historical price data service
2. Cost basis tracking per asset
3. P&L calculation engine
4. UI components for dashboard

**Leverages:** Existing `usePrice.ts`, `useAsset.ts` hooks

**Estimated Value:** High — consistently cited as top reason for premium upgrades

### 5.2 CSV Tax Export (Pro Feature) — 1 week

**What:** Export transaction history in tax-software-compatible format

**Components Needed:**
1. Transaction data formatter
2. CSV generation service
3. Format presets (TurboTax, Koinly, etc.)
4. Download trigger

**Leverages:** Existing transaction history data

**Estimated Value:** High during tax season (Q1)

### 5.3 Yield Optimizer PRO — 1 week

**What:** Enhanced yield comparison with recommendations

**Current State:** `useLiqwid.ts` has `getMarketsLQ()` returning APY data

**Enhancement:**
1. Aggregate yields from Liqwid + FluidTokens
2. Risk-adjusted recommendations
3. "Optimal allocation" suggestions
4. Push notifications for yield changes

**Leverages:** Existing protocol hooks

### 5.4 Depeg & Yield Alerts — 3 days

**What:** Automatic alerts when stablecoins deviate from peg or yields change significantly

**Leverages:** Existing `usePriceAlert.ts` pattern, `stableTokens` registry

### 5.5 Auto-Compound (Phase 2) — 3 weeks

**What:** Automatically claim and reinvest lending rewards

**Requires:**
1. Background service for monitoring claimable rewards
2. Transaction batching logic
3. User preferences for compound frequency
4. Notification on successful compound

**Leverages:** Existing Liqwid/FluidTokens transaction builders

### 5.6 DCA Automation (Phase 2) — 3 weeks

**What:** Scheduled recurring purchases

**Requires:**
1. Schedule storage and management
2. Background execution service (Chrome alarms API)
3. Execution history tracking
4. Failure handling and notifications

**Leverages:** Existing swap infrastructure

---

## Part 6: Revenue Projections

### Assumptions

| Metric | Conservative | Moderate | Optimistic |
|--------|--------------|----------|------------|
| Begin target users | 500,000 | 500,000 | 500,000 |
| Begin market share | 2% | 5% | 10% |
| Begin active users | 10,000 | 25,000 | 50,000 |
| Pro conversion rate | 5% | 7% | 10% |
| Pro subscribers | 500 | 1,750 | 5,000 |
| Monthly ARPU | $8.50* | $8.50 | $8.50 |

*Blended ARPU accounts for annual discounts

### Year 1 Revenue Projection

| Scenario | Pro Subscribers | Monthly Revenue | Annual Revenue |
|----------|-----------------|-----------------|----------------|
| Conservative | 500 | $4,250 | $51,000 |
| Moderate | 1,750 | $14,875 | $178,500 |
| Optimistic | 5,000 | $42,500 | $510,000 |

### Revenue Breakdown by Source

| Source | % of Revenue | Year 1 (Moderate) |
|--------|--------------|-------------------|
| Pro Monthly ($9.99) | 40% | $71,400 |
| Pro Annual ($95.88) | 35% | $62,475 |
| Pro Lifetime ($199) | 15% | $26,775 |
| Enterprise | 10% | $17,850 |
| **Total** | **100%** | **$178,500** |

### Break-Even Analysis

| Cost Category | Monthly | Annual |
|---------------|---------|--------|
| Payment processing (3%) | $446 | $5,355 |
| Backend infrastructure | $500 | $6,000 |
| Support (part-time) | $2,000 | $24,000 |
| Marketing/UA | $3,000 | $36,000 |
| **Total Costs** | **$5,946** | **$71,355** |
| **Break-even subscribers** | **~700** | — |

---

## Part 7: Implementation Roadmap

### Phase 1: MVP Launch (Weeks 1-2) 🚀

**Goal:** Ship subscription system with existing features gated

| Task | Owner | Days | Dependencies |
|------|-------|------|--------------|
| Subscription data model | Backend | 2 | None |
| Stripe/payment integration | Backend | 3 | Data model |
| Subscription UI (settings) | Frontend | 2 | Backend API |
| Feature gating (alerts, wallets) | Frontend | 2 | Subscription state |
| "Best Route PRO" gate | Frontend | 1 | Subscription state |
| Testing & QA | QA | 2 | All above |
| **Total** | | **12 days** | |

**Deliverables:**
- Subscription purchase flow
- Basic feature gating
- "Upgrade to Pro" prompts at gate points

### Phase 2: Core Premium (Weeks 3-6) 📊

**Goal:** Ship high-value Pro features

| Task | Owner | Days | Dependencies |
|------|-------|------|--------------|
| CSV export service | Backend | 3 | None |
| CSV export UI | Frontend | 2 | Service |
| P&L tracking backend | Backend | 7 | Price history |
| P&L dashboard UI | Frontend | 5 | Backend |
| Yield optimizer | Frontend | 5 | Existing hooks |
| Depeg alerts | Frontend | 2 | Price data |
| **Total** | | **24 days** | |

**Deliverables:**
- Full P&L tracking
- Tax export functionality
- Yield optimization recommendations

### Phase 3: Automation (Weeks 7-12) 🤖

**Goal:** Ship automation features, enterprise tier

| Task | Owner | Days | Dependencies |
|------|-------|------|--------------|
| Auto-compound service | Backend | 10 | Liqwid integration |
| Auto-compound UI | Frontend | 5 | Service |
| DCA scheduler | Backend | 10 | Swap integration |
| DCA management UI | Frontend | 5 | Scheduler |
| Enterprise features spec | Product | 3 | Phase 2 complete |
| API documentation | Backend | 5 | Enterprise spec |
| **Total** | | **38 days** | |

**Deliverables:**
- Auto-compound for lending positions
- DCA scheduling and execution
- Enterprise API access

### Timeline Summary

```
Week  1  2  3  4  5  6  7  8  9  10  11  12
      ├──┴──┤  ├──┴──┴──┴──┤  ├──┴──┴──┴──┴──┤
      Phase 1    Phase 2          Phase 3
       MVP        Core           Automation
      Launch    Premium
        │          │                │
        ▼          ▼                ▼
    Sub system   P&L, CSV      Auto-compound
    Feature gates Yield opt.       DCA
```

---

## Part 8: Technical Requirements

### Subscription System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      SUBSCRIPTION FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌─────────────┐    ┌───────────────────────┐  │
│  │ User     │───►│ Stripe/     │───►│ Begin Backend        │  │
│  │ clicks   │    │ Payment     │    │ - Webhook handler    │  │
│  │ upgrade  │    │ Provider    │    │ - Subscription DB    │  │
│  └──────────┘    └─────────────┘    │ - User entitlements  │  │
│                                      └───────────┬───────────┘  │
│                                                  │               │
│                                                  ▼               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Extension/App                          │  │
│  │  ┌────────────┐    ┌────────────┐    ┌───────────────┐   │  │
│  │  │ Check sub  │───►│ Feature    │───►│ Enable/       │   │  │
│  │  │ status API │    │ flags      │    │ disable UI    │   │  │
│  │  └────────────┘    └────────────┘    └───────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Data Model

```typescript
// Subscription model
interface UserSubscription {
  userId: string;
  tier: 'free' | 'pro' | 'enterprise';
  status: 'active' | 'canceled' | 'past_due' | 'trialing';
  billingCycle: 'monthly' | 'annual' | 'lifetime';
  currentPeriodStart: Date;
  currentPeriodEnd: Date;
  stripeCustomerId?: string;
  stripeSubscriptionId?: string;
  features: FeatureFlags;
}

interface FeatureFlags {
  maxWallets: number;
  maxAlerts: number;
  bestRouteEnabled: boolean;
  plTrackingEnabled: boolean;
  csvExportEnabled: boolean;
  autoCompoundEnabled: boolean;
  dcaEnabled: boolean;
  apiAccessEnabled: boolean;
  prioritySupport: boolean;
}

// Feature flag defaults by tier
const TIER_FEATURES: Record<string, FeatureFlags> = {
  free: {
    maxWallets: 3,
    maxAlerts: 3,
    bestRouteEnabled: false,
    plTrackingEnabled: false,
    csvExportEnabled: false,
    autoCompoundEnabled: false,
    dcaEnabled: false,
    apiAccessEnabled: false,
    prioritySupport: false,
  },
  pro: {
    maxWallets: 10,
    maxAlerts: Infinity,
    bestRouteEnabled: true,
    plTrackingEnabled: true,
    csvExportEnabled: true,
    autoCompoundEnabled: true,
    dcaEnabled: true,
    apiAccessEnabled: false,
    prioritySupport: true,
  },
  enterprise: {
    maxWallets: Infinity,
    maxAlerts: Infinity,
    bestRouteEnabled: true,
    plTrackingEnabled: true,
    csvExportEnabled: true,
    autoCompoundEnabled: true,
    dcaEnabled: true,
    apiAccessEnabled: true,
    prioritySupport: true,
  },
};
```

### Backend Requirements

| Component | Technology | Purpose |
|-----------|------------|---------|
| Payment processing | Stripe | Subscriptions, webhooks |
| User database | PostgreSQL | Subscription state |
| Price history | TimescaleDB/InfluxDB | P&L calculations |
| Background jobs | Redis + Bull | Auto-compound, DCA |
| API | Express/Fastify | Enterprise API access |

### Extension Changes

| Change | Files Affected | Effort |
|--------|---------------|--------|
| Subscription context provider | `app-context/` | Medium |
| Feature gate HOC/hooks | `hooks/useSubscription.ts` | Low |
| Upgrade prompt component | `components/` | Low |
| Settings subscription view | `views/user-settings/` | Medium |
| Stripe Checkout integration | `views/subscription/` | Medium |

---

## Part 9: Competitive Positioning

### Messaging Framework

**Tagline:** "Begin Pro: Your Stablecoin Home Onchain"

**Key Differentiators:**

| vs. Competitor | Begin Advantage | Message |
|----------------|-----------------|---------|
| vs. CoinStats | Cardano-native DeFi integration | "Built for DeFi (multi-chain), not just tracking" |
| vs. Zerion | Lower price, stablecoin focus | "Financial features, DeFi yields" |
| vs. Eternl/Nami | Premium features they don't have | "The features serious crypto users need" |
| vs. Lace | DeFi-native, yield optimization | "Beyond IOG's basic wallet" |

### Positioning Matrix

```
                         DEFI DEPTH
                            HIGH
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              │    BEGIN      │   Protocol    │
              │     PRO       │    Native     │
              │               │   (Liqwid)    │
              │               │               │
    PRICE ────┼───────────────┼───────────────┼──── PRICE
     LOW      │               │               │     HIGH
              │               │               │
              │   Free        │  CoinStats/   │
              │   Wallets     │   Zerion      │
              │  (Nami,Eternl)│               │
              │               │               │
              └───────────────┼───────────────┘
                              │
                            LOW
                         DEFI DEPTH
```

**Begin Pro occupies the sweet spot:** Deep DeFi integration at accessible pricing.

---

## Part 10: Launch Strategy

### Pre-Launch (2 weeks before)

1. **Beta program** — Invite 50 power users to test Pro features
2. **Content creation** — Blog posts, Twitter threads on Pro features
3. **Partner outreach** — Coordinate with Liqwid, FluidTokens for co-marketing
4. **Pricing validation** — Survey beta users on price sensitivity

### Launch Week

| Day | Activity |
|-----|----------|
| Monday | Soft launch to beta users |
| Tuesday | Public announcement (Twitter, Discord) |
| Wednesday | Tutorial content release |
| Thursday | Partner amplification |
| Friday | Reddit AMA |

### Launch Offers

| Offer | Duration | Discount |
|-------|----------|----------|
| Early Bird Annual | First 30 days | 40% off ($4.79/mo) |
| Lifetime Founders | First 100 users | 30% off ($139) |

### Success Metrics (30 days post-launch)

| Metric | Target |
|--------|--------|
| Pro trial signups | 500 |
| Paid conversions | 100 (20% conversion) |
| MRR | $850 |
| Churn rate | <10% |
| NPS score | >40 |

---

## Part 11: Risk Assessment

### Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Payment integration delays | Medium | High | Start Stripe integration Week 1 |
| Feature gate bypassed | Low | Medium | Server-side validation |
| Backend scaling issues | Low | Medium | Start with serverless functions |

### Market Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Low conversion rate | Medium | High | A/B test pricing, features |
| Competitor response | Low | Medium | First-mover advantage, iterate fast |
| Crypto bear market | Medium | Medium | Focus on utility over speculation |

### Operational Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Support overwhelm | Medium | Medium | Clear documentation, Discord community |
| Feature requests overload | High | Low | Roadmap transparency, voting system |

---

## Appendix A: Feature Comparison with Competitors

| Feature | Begin Free | Begin Pro | CoinStats Premium | Zerion Premium |
|---------|------------|-----------|-------------------|----------------|
| Price | $0 | $9.99/mo | $13.99/mo | NFT-based |
| Portfolios/Wallets | 3 | 10 | 100 | Unlimited |
| Transactions | Unlimited | Unlimited | 100K | Unlimited |
| P&L Tracking | ❌ | ✅ | ✅ | ✅ |
| Tax Export | ❌ | ✅ | ✅ | ✅ |
| Price Alerts | 3 | Unlimited | Custom | Limited |
| **DeFi (multi-chain)** | ✅ | ✅ | ❌ | ❌ |
| **Multi-DEX Routing** | Basic | Pro | ❌ | ❌ |
| **Liqwid Integration** | ✅ | ✅ | ❌ | ❌ |
| **Yield Optimization** | ❌ | ✅ | ❌ | ❌ |
| **Auto-Compound** | ❌ | ✅ | ❌ | ❌ |

---

## Appendix B: Pricing Sensitivity Analysis

### Price Point Testing

| Price | Expected Conversion | MRR (1000 users) | Annual |
|-------|---------------------|------------------|--------|
| $4.99/mo | 10% | $499 | $5,988 |
| $7.99/mo | 7% | $559 | $6,708 |
| **$9.99/mo** | **5%** | **$500** | **$6,000** |
| $12.99/mo | 4% | $520 | $6,240 |
| $14.99/mo | 3% | $450 | $5,400 |

**Analysis:** $9.99 hits the psychological threshold while maintaining competitive positioning. Lower prices don't significantly increase conversion enough to offset revenue loss.

### Annual vs Monthly

| Plan | Monthly Price | Annual Price | Annual Discount | Expected Mix |
|------|---------------|--------------|-----------------|--------------|
| Monthly | $9.99 | $119.88 | — | 40% |
| Annual | $7.99 | $95.88 | 20% | 50% |
| Lifetime | — | $199 | — | 10% |

---

## Appendix C: Existing Codebase Assets

### Hooks That Can Be Leveraged

| Hook | Current Purpose | Premium Application |
|------|-----------------|---------------------|
| `useLiqwid.ts` | Liqwid protocol integration | Yield optimization, auto-compound |
| `useFluid.ts` | FluidTokens boosted staking | Yield aggregation |
| `useSwap.ts` | DEX configuration | Best route PRO |
| `usePriceAlert.ts` | Price notifications | Unlimited alerts, depeg alerts |
| `usePrice.ts` | Price data | P&L calculations |
| `useAsset.ts` | Asset management | Portfolio tracking |
| `useMynth.ts` | Stablecoin bridge | Stablecoin optimization |
| `useDexHunter.ts` | DEX aggregation | Best route comparison |

### Views That Need Premium Gates

| View | Free Access | Pro Features |
|------|-------------|--------------|
| `views/invest/view-lend.tsx` | ✅ | Yield optimizer overlay |
| `views/swap/` | Basic swap | Best route comparison |
| `views/price-alert/` | 3 alerts | Unlimited |
| `views/wallet/` | 3 wallets | 10 wallets |
| `views/transactions/` | 30 days | Unlimited history |

---

## Conclusion

Begin Wallet has significant existing infrastructure to launch a premium subscription model quickly:

1. **Day 1 launch** is possible with feature gating on existing capabilities
2. **$9.99/mo price point** is competitive and sustainable
3. **Conservative Year 1 projection** of $50K-150K ARR is achievable
4. **Phase 1 (2 weeks)** delivers MVP subscription system
5. **Phase 2-3 (3 months)** builds full premium feature set

**Recommended immediate actions:**
1. Finalize tier features and pricing
2. Begin Stripe integration
3. Design upgrade prompts and subscription UI
4. Create feature gate infrastructure
5. Plan launch marketing

---

*Document prepared for Begin Wallet strategic planning*
*Contact: Research Team*
*Last updated: 2026-02-03*
