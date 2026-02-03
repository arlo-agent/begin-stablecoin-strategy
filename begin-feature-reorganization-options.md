# Begin Feature Reorganization Options

*"The Stablecoin Bank Onchain" — What do we do with existing features?*

---

## Current Feature Inventory

### Core Wallet
| Feature | Status | Fits "Bank" Theme? |
|---------|--------|-------------------|
| Wallet Home | ✅ Built | 🔄 Needs redesign |
| Send | ✅ Built | ✅ Yes ("Transfer") |
| Receive | ✅ Built | ✅ Yes ("Deposit") |
| Swap | ✅ Built | ✅ Yes ("Exchange") |
| Transaction History | ✅ Built | ✅ Yes ("Statements") |

### Invest Section
| Feature | Status | Fits "Bank" Theme? |
|---------|--------|-------------------|
| Lend/Borrow (Liqwid) | ✅ Built | ✅ Core — "Savings Vault" |
| ADA Staking | ✅ Built | ⚠️ Partial — "Earn on ADA" |
| Boosted Staking | ✅ Built | ⚠️ Partial |
| Precious Metals | ✅ Built | ❌ Off-brand |
| BTC Earn | ✅ Built | ⚠️ Partial |
| Mynth Savings | ✅ Built | ✅ Yes — stablecoin bridge |

### Hub Section
| Feature | Status | Fits "Bank" Theme? |
|---------|--------|-------------------|
| Midnight Claim | ✅ Built | ❌ Temporary promo |
| eSIM Shop | ✅ Built | ⚠️ "Spend" category |
| Travel Shop | ✅ Built | ⚠️ "Spend" category |
| Governance | ✅ Built | ❌ Off-brand |

### Other Features
| Feature | Status | Fits "Bank" Theme? |
|---------|--------|-------------------|
| NFTs/Collectibles | ✅ Built | ❌ Off-brand |
| dApps Browser | ✅ Built | ⚠️ Power user |
| Hardware Wallets | ✅ Built | ✅ Security feature |
| BeginPay | ✅ Built | ✅ Yes — "Payments" |
| BeginID | ✅ Built | ⚠️ Identity feature |
| Price Alerts | ✅ Built | ✅ Pro feature |
| Charts/Performance | ✅ Built | ✅ Pro feature |
| Contacts | ✅ Built | ✅ Yes — "Payees" |

---

## Option 1: "Clean Slate" — Full Rebrand

**Philosophy:** Rebuild home screen entirely around bank metaphor. Hide non-bank features.

### New Structure
```
┌─────────────────────────────────────┐
│  HOME                               │
│  - Balance (Cash + Investments)     │
│  - Savings Vault                    │
│  - Auto-Invest (DCA)                │
│  - Recent Activity                  │
├─────────────────────────────────────┤
│  EARN                               │
│  - Savings Vault (Liqwid)           │
│  - Staking (ADA, SOL)               │
│  - Yield Explorer                   │
├─────────────────────────────────────┤
│  SPEND (Future)                     │
│  - Cards                            │
│  - eSIM                             │
│  - Travel                           │
├─────────────────────────────────────┤
│  MORE ⋯                             │
│  - NFTs                             │
│  - dApps                            │
│  - Governance                       │
│  - Settings                         │
└─────────────────────────────────────┘
```

### What Gets Hidden
- NFTs → Buried in "More"
- Governance → Buried in "More"
- dApps Browser → Buried in "More"
- Precious Metals → Remove or hide
- Prediction Markets → Remove

### Pros
- ✅ Clean, focused experience
- ✅ Strong brand differentiation
- ✅ Easier to market

### Cons
- ❌ Existing users may be confused
- ❌ Loses some power-user features
- ❌ More development work

---

## Option 2: "Progressive Disclosure" — Layers

**Philosophy:** Bank features prominent, everything else accessible but secondary.

### New Structure
```
┌─────────────────────────────────────┐
│  HOME (Bank Mode - Default)         │
│  - Cash Balance                     │
│  - Savings Vault                    │
│  - Quick Actions: Add/Send/Earn     │
├─────────────────────────────────────┤
│  Toggle: [Bank] [Wallet]            │
├─────────────────────────────────────┤
│  WALLET MODE (Power User)           │
│  - All tokens                       │
│  - NFTs                             │
│  - dApps                            │
│  - Governance                       │
│  - Full control                     │
└─────────────────────────────────────┘
```

### How It Works
- Default view = "Bank Mode" (stablecoins, savings, simple)
- Toggle to "Wallet Mode" for full crypto features
- Power users get everything, normies get simplicity

### Pros
- ✅ Keeps all features
- ✅ Appeals to both audiences
- ✅ Less development work

### Cons
- ❌ Two modes = complexity
- ❌ Brand message diluted
- ❌ Users may be confused by toggle

---

## Option 3: "Tabbed Sections" — Clear Categories

**Philosophy:** Everything stays, but organized into clear tabs.

### New Structure
```
Bottom Navigation:
[🏠 Home] [💰 Earn] [🛒 Spend] [🎨 Explore] [⚙️ More]

HOME
├── Balance Overview
├── Cash (Stablecoins)
├── Investments (Volatile)
└── Recent Activity

EARN
├── Savings Vault (Liqwid)
├── Staking (ADA/SOL)
├── Auto-Invest (DCA)
└── Yield Explorer

SPEND
├── Send Money
├── BeginPay
├── eSIM
├── Travel
└── (Future: Cards)

EXPLORE
├── NFTs
├── dApps
├── Governance
└── Prediction Markets

MORE
├── Settings
├── Hardware Wallets
├── BeginID
├── Price Alerts
└── Support
```

### Pros
- ✅ Everything accessible
- ✅ Clear mental model
- ✅ Bank features prominent in Home/Earn

### Cons
- ❌ 5 tabs may be too many
- ❌ "Explore" still feels crypto-native
- ❌ Less focused than Option 1

---

## Option 4: "Bank + Power Menu" — Hybrid (RECOMMENDED)

**Philosophy:** Bank-first home, power features in slide-out menu.

### New Structure
```
┌─────────────────────────────────────┐
│  [≡]  Begin                  [Pro]  │
├─────────────────────────────────────┤
│                                     │
│  HOME (Always Bank-Focused)         │
│  - Total Balance                    │
│  - Cash Section                     │
│  - Savings Vault                    │
│  - Auto-Invest                      │
│  - Investments (collapsible)        │
│                                     │
├─────────────────────────────────────┤
│  [🏠 Home] [💰 Earn] [📊 Activity]  │
└─────────────────────────────────────┘

HAMBURGER MENU [≡]
├── 🏠 Home
├── 💰 Earn
│   ├── Savings Vault
│   ├── Staking
│   └── Yield Explorer
├── 📤 Send & Pay
│   ├── Send
│   ├── BeginPay
│   └── Contacts
├── 📊 Portfolio
│   ├── All Assets
│   ├── Performance
│   └── Price Alerts
├── 🛒 Shop
│   ├── eSIM
│   └── Travel
├── 🔧 Advanced
│   ├── NFTs
│   ├── dApps
│   ├── Governance
│   └── Swap
├── ⚙️ Settings
│   ├── Security
│   ├── Hardware Wallets
│   └── Subscription
└── 💬 Support
```

### How It Works
1. **Home screen** = Pure bank experience
2. **Bottom nav** = 3 tabs (simple)
3. **Hamburger** = Everything else organized
4. **"Advanced"** = Crypto-native features for power users

### Pros
- ✅ Bank-first without losing features
- ✅ Clean home screen
- ✅ Power users can find everything
- ✅ Easy to add/remove features
- ✅ 3 bottom tabs = simple

### Cons
- ❌ Hamburger menus are less discoverable
- ❌ Some features require extra taps

---

## Feature Mapping: Old → New

### Option 4 Mapping

| Old Location | New Location | Notes |
|--------------|--------------|-------|
| Wallet Home | **Home** (redesigned) | Bank-style balance |
| Invest > Lend | **Earn > Savings Vault** | Core feature |
| Invest > Staking | **Earn > Staking** | Keep |
| Invest > Metals | **Remove** or Shop | Off-brand |
| Hub > eSIM | **Shop > eSIM** | "Spend your crypto" |
| Hub > Travel | **Shop > Travel** | Keep |
| Hub > Governance | **Advanced > Governance** | Power users |
| Hub > Midnight | **Remove** after promo | Temporary |
| Collectibles | **Advanced > NFTs** | Power users |
| dApps | **Advanced > dApps** | Power users |
| Swap | **Advanced > Swap** or Home action | Debatable |
| Send | **Send & Pay** | Keep prominent |
| Charts | **Portfolio > Performance** | Pro feature |
| Price Alerts | **Portfolio > Alerts** | Pro feature |

---

## Recommended Approach

### Phase 1: Quick Wins (1-2 weeks)
1. **Redesign Home Screen** per wireframe
2. **Rename "Invest"** → "Earn"
3. **Add "Cash" section** at top of wallet
4. **Add Savings Vault card** to home

### Phase 2: Reorganize (2-4 weeks)
1. **Implement hamburger menu** with new structure
2. **Move NFTs/dApps** to "Advanced"
3. **Create "Shop" section** for eSIM/Travel
4. **Simplify bottom nav** to 3 tabs

### Phase 3: Polish (4-6 weeks)
1. **Remove off-brand features** (Metals, etc.)
2. **Add premium gates** to Pro features
3. **A/B test** bank vs wallet terminology
4. **User research** on new IA

---

## My Recommendation

**Go with Option 4 (Bank + Power Menu)** because:

1. **Doesn't alienate existing users** — Everything still accessible
2. **Strong bank identity** — Home screen is 100% bank-focused
3. **Scalable** — Easy to add/remove features from menu
4. **Low risk** — Can implement incrementally
5. **Premium-ready** — Pro features naturally live in menu sections

The key insight: **Home screen = marketing.** That's what people see first. Make it scream "Stablecoin Bank." Everything else can live in the menu for those who need it.

---

*Next: Implement wireframe for Option 4 home screen + hamburger menu structure*
