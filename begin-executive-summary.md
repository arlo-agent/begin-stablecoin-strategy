# Begin Wallet: Executive Summary

*Strategic Analysis — 2026-02-03*

---

## The Opportunity

**$84B+ stablecoin financial services market by 2026.** Regulatory clarity (GENIUS Act, MiCA) has opened a window. The infrastructure layer exists; the services layer remains unbuilt. Begin can become "The Stablecoin Bank for Cardano."

---

## What We Found

### Current State: Better Than Expected

| Area | Status | Notes |
|------|--------|-------|
| **begin-cli** | 60% built | Minswap swap routing complete, wallet management solid |
| **b58-extension** | 70% built | DexHunter multi-DEX, Liqwid lending, stablecoin registry exists |
| **Mobile** | Already exists | Capacitor iOS/Android with biometrics — not widely known |

### Key Gaps

| Gap | CLI | Extension | Priority |
|-----|-----|-----------|----------|
| Yield aggregation | ❌ | ❌ | HIGH |
| Savings vaults (Liqwid) | ❌ | ❌ | HIGH |
| Stablecoin-optimized swap | 🟡 | 🟡 | HIGH |
| DCA automation | ❌ | ❌ | MEDIUM |
| FluidTokens integration | ❌ | ❌ | LOW (future) |
| Multi-DEX routing | 🟡 | ✅ | LOW (extension has it) |

---

## Strategic Decision: Native vs Extension

### Recommendation: Stay on Extension + Capacitor

**Don't rewrite. Improve.**

| Factor | Verdict |
|--------|---------|
| **Time to market** | Native = 5-6 months. Extension features = ship now. |
| **Performance** | Adequate. Can improve with offscreen docs + web workers. |
| **Mobile** | Already have Capacitor apps with biometrics. Optimize these. |
| **Code reuse** | Only 40-60% reusable in React Native anyway. |

### The Passkey Question

**Critical finding:** Browser extensions CANNOT do cross-site passkeys. Current biometrics ≠ passkeys.

- If passkeys = nice-to-have → Stay on extension
- If passkeys = hard requirement → Native migration necessary

**Current auth (Capacitor biometrics) is industry standard.** Passkeys for seed phrase replacement is future-looking.

### If Native Becomes Necessary

React Native + Expo. Budget 6 months. Existing migration plan is solid.

---

## Quick Wins (Ship This Week)

| Task | Time | Impact |
|------|------|--------|
| Add stablecoin aliases to CLI swap | 2 hours | Enables `begin swap --from USDA --to USDM` |
| `begin stablecoins list` command | 4 hours | Shows supported stables with balances |
| Fix mocked staking transactions | 2-3 days | Completes wallet functionality |
| Enhanced wallet Cash section (extension) | 1 week | High visibility CTA for vault |

---

## 12-Week Roadmap

### Phase 1: Foundation (Weeks 1-2)
- Stablecoin swap improvements (CLI + extension)
- Real staking transactions
- `useStablePeg` hook for depeg monitoring

### Phase 2: Yield Intelligence (Weeks 3-4)
- `useYieldAggregator` hook
- Liqwid adapter (FluidTokens deferred to Phase 5)
- Yield dashboard view
- `begin yield list/best/compare` commands

### Phase 3: Savings Vault (Weeks 5-8)
- Vault strategy system (conservative/balanced/aggressive)
- Deposit/withdraw flows
- Auto-compound service
- Full CLI + extension UI

### Phase 4: DCA & Polish (Weeks 9-12)
- DCA scheduling and execution
- Background service for automation
- Performance optimizations (offscreen docs, web workers)
- Testing and launch prep

### Phase 5: Protocol Expansion (Future)
- FluidTokens integration (second lending protocol)
- Additional yield sources
- Multi-protocol vault strategies

---

## Revenue Model

| Source | Model |
|--------|-------|
| Vault management fee | 0.1-0.5% annual AUM |
| Swap routing fee | 0.05-0.1% per tx |
| **Premium subscription** | **$9.99/mo Pro tier** (see detailed strategy below) |

**Conservative projection:** 5% of Cardano stablecoin flow (~$50M) = $50-100K ARR

### Premium Subscription Strategy

See **[Begin Premium Strategy](begin-premium-strategy.md)** for comprehensive subscription monetization plan:

| Tier | Price | Key Features |
|------|-------|--------------|
| **Free** | $0 | Basic wallet, 3 wallets, basic DeFi |
| **Pro** | $9.99/mo | P&L tracking, CSV export, Best Route PRO, unlimited alerts, yield optimizer |
| **Enterprise** | $49.99/mo | API access, unlimited wallets, team management |

**Year 1 Revenue Projection (Moderate):** $178,500 ARR with 1,750 Pro subscribers

**Launch-Ready Features (Already Built):**
- Multi-DEX "Best Route PRO" (gate existing DexHunter integration)
- Unlimited price alerts (existing `usePriceAlert.ts`)
- Extended wallet limits (configuration change)
- Extended trade history (query filtering)

**Near-Term Premium Features (1-3 months):**
- P&L tracking with cost basis
- CSV tax export
- Yield optimization recommendations
- Auto-compound for Liqwid positions
- DCA automation

---

## Architecture Actions (Non-Native Improvements)

### Immediate

1. **Implement offscreen documents** — WASM persistence across service worker restarts
2. **Add web workers** — Heavy crypto operations off main thread
3. **Optimize Capacitor mobile** — It exists, make it better

### Near-Term

4. **FluidTokens integration** — Second lending protocol for diversification
5. **Background price fetching** — Keep stablecoin peg data fresh
6. **DCA executor service** — Chrome alarms API for scheduling

---

## Success Metrics

| Metric | 3-Month Target | 6-Month Target |
|--------|----------------|----------------|
| Vault AUM | $100K | $500K |
| Stablecoin swaps/month | 500 | 2,000 |
| Active DCA schedules | 100 | 500 |
| Yield dashboard DAU | 10% of users | 20% of users |

---

## Bottom Line

**Begin has more infrastructure than expected.** The path forward is:

1. **Don't rebuild** — Optimize what exists
2. **Ship stablecoin features** — Vault + yield aggregation = differentiation
3. **Focus mobile on Capacitor** — Already have apps, make them better
4. **Monitor passkey adoption** — Only trigger for native if it becomes critical
5. **Move fast** — Window is open, competitors are moving

The research, implementation plans, and roadmap are ready. Time to build.

---

## Appendix: Document Locations

| Document | Path |
|----------|------|
| Original Research | `research/stablecoin-financial-services-begin-strategy.md` |
| CLI Implementation Plan | `research/begin-stablecoin-implementation-plan.md` |
| Extension Implementation Plan | `research/begin-extension-stablecoin-implementation-plan.md` |
| Native vs Extension Evaluation | `research/begin-native-vs-extension-evaluation.md` |
| **Premium Strategy** | `research/begin-premium-strategy.md` |
| **This Summary** | `research/begin-executive-summary.md` |

---

*Prepared for Francis | NeuralSpark*
