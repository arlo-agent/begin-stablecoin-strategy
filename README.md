# Begin Wallet — Stablecoin Strategy

> **"Begin — Your Stablecoin Home Onchain"**  
> *The onchain complement to your bank.*

This repository contains strategic research, implementation plans, and UX specifications for transforming Begin Wallet into the premier stablecoin home for multi-chain users.

---

## 📋 Document Index

### Strategy & Vision

| Document | Description |
|----------|-------------|
| [**Executive Summary**](begin-executive-summary.md) | TL;DR of the entire strategy — start here |
| [**Market Research**](stablecoin-financial-services-begin-strategy.md) | $84B+ stablecoin opportunity, competitive analysis, market positioning |

### Implementation Plans

| Document | Description |
|----------|-------------|
| [**CLI Implementation Plan**](begin-stablecoin-implementation-plan.md) | 12-week roadmap for begin-cli stablecoin features |
| [**Extension Implementation Plan**](begin-extension-stablecoin-implementation-plan.md) | UI/UX implementation for b58-extension with wireframes |
| [**Solana SPL Summary**](solana-spl-implementation-summary.md) | SPL token support implementation (USDC, USDT) |

### Technical Decisions

| Document | Description |
|----------|-------------|
| [**Native vs Extension Evaluation**](begin-native-vs-extension-evaluation.md) | Should we go native? Verdict: Stay on extension + Capacitor |

### Product & UX

| Document | Description |
|----------|-------------|
| [**Premium Strategy**](begin-premium-strategy.md) | Subscription tiers ($9.99/mo Pro), feature matrix, revenue projections |
| [**Home Screen Wireframe**](begin-ux-homescreen-wireframe.md) | UX specification for "Stablecoin Home" interface |
| [**Feature Reorganization**](begin-feature-reorganization-options.md) | Options for organizing existing features (recommended: Home + Power Menu) |

---

## 🎯 Key Decisions

| Decision | Outcome |
|----------|---------|
| **Positioning** | "Your Stablecoin Home Onchain" — complement to banks, not competitor |
| **Platform** | Stay on browser extension + Capacitor mobile (no native rewrite) |
| **Initial Protocol** | Liqwid Finance only (FluidTokens later) |
| **Premium Price** | $9.99/month, $199 lifetime |
| **Multi-chain** | Cardano + Bitcoin + Solana — chains invisible to user |

---

## 🚀 Quick Wins (Ship This Week)

1. Add stablecoin aliases to CLI swap — 2 hours
2. `begin stablecoins list` command — 4 hours
3. Enhanced wallet "Cash" section — 1 week
4. Rename "Invest" → "Earn" — 1 day

---

## 📊 Revenue Projections (Year 1)

| Scenario | Subscribers | ARR |
|----------|-------------|-----|
| Conservative | 500 | $51K |
| Moderate | 1,750 | $178K |
| Optimistic | 5,000 | $510K |

---

## 🗂 Related Repositories

| Repo | Description |
|------|-------------|
| [b58-extension](https://github.com/ADABase/b58-extension) | Begin Wallet browser extension |
| [begin-cli](https://github.com/ADABase/begin-cli) | Command-line interface for Begin |

---

## 📅 Timeline

### Phase 1: Foundation (Weeks 1-2)
- Stablecoin swap improvements
- Real staking transactions
- Cash section redesign

### Phase 2: Yield Intelligence (Weeks 3-4)
- Yield aggregator hook
- Liqwid adapter
- Yield dashboard

### Phase 3: Savings Vault (Weeks 5-8)
- Vault strategies (conservative/balanced/aggressive)
- Auto-compound
- Deposit/withdraw flows

### Phase 4: DCA & Polish (Weeks 9-12)
- DCA automation
- Premium feature gates
- Launch subscription

### Phase 5: Expansion (Future)
- FluidTokens integration
- Additional yield sources
- Cross-chain features

---

## 🏠 Brand Guidelines

**Tagline:** "Begin — Your Stablecoin Home Onchain"

**Subheadline:** "The onchain complement to your bank."

**Key Messages:**
- Home for your stablecoins
- Earn yield, save smarter, stay in control
- Premium UX meets DeFi yields
- Self-custody with easy access

**Avoid:**
- ❌ "Bank" (regulatory issues)
- ❌ "Cardano-specific" (we're multi-chain)
- ❌ "Just a wallet" (we're financial services)

---

*Last updated: 2026-02-04*
*Maintained by: Arlo @ NeuralSpark*
