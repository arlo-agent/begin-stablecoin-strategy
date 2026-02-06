# Begin Wallet: AI Mode Integration Research

## Executive Summary

This research document explores how Begin Wallet can integrate an AI assistant alongside its traditional mobile UI. The goal is to balance power users who want conversational AI capabilities with regular users who need familiar button-based interfaces for core wallet actions.

Based on analysis of industry patterns, fintech examples, and crypto-specific considerations, we recommend a **phased hybrid approach**: starting with an AI Overlay/Sheet (Option 1) combined with Hybrid Cards (Option 4), then evolving toward Contextual AI (Option 3) as user patterns emerge.

---

## Table of Contents

1. [Current Landscape](#current-landscape)
2. [Option 1: AI Overlay/Sheet](#option-1-ai-overlaysheet)
3. [Option 2: AI Mode Toggle](#option-2-ai-mode-toggle)
4. [Option 3: Contextual AI](#option-3-contextual-ai)
5. [Option 4: Hybrid Cards](#option-4-hybrid-cards)
6. [Option 5: Voice-First](#option-5-voice-first)
7. [Comparative Analysis](#comparative-analysis)
8. [Recommended Approach](#recommended-approach)
9. [Implementation Roadmap](#implementation-roadmap)
10. [Sources & References](#sources--references)

---

## Current Landscape

### The AI Wallet Evolution

The crypto wallet industry is undergoing a fundamental shift from "dumb boxes" (key storage + signing) to "smart co-pilots" that actively assist users. According to industry analysis:

> "AI wallets act as smart co-pilots, proactively spotting threats and translating complex transactions into simple, human-readable terms." — PixelPlex, 2025

Key capabilities emerging in AI-enhanced wallets:
- **Transaction simulation** — Preview outcomes before signing
- **Scam detection** — Real-time contract analysis
- **Intent-based actions** — "Buy $100 of token X" vs. manual swap flows
- **Portfolio insights** — Personalized recommendations
- **Natural language queries** — "What did I spend on gas last month?"

### Industry Benchmarks

| App | AI Integration Style | Key Features |
|-----|---------------------|--------------|
| Bank of America (Erica) | Floating button → Search bar | 700+ intents, 50-60% proactive suggestions, 48-sec avg interaction |
| Klarna | Embedded assistant + ChatGPT plugin | Shopping recommendations, conversational search |
| Coinbase | Learning Hub + limited AI | Educational content, basic insights |
| Phantom | No AI yet | Best-in-class traditional UX benchmark |

### Key Insight: Erica's Success Pattern

Bank of America's Erica virtual assistant offers the most instructive model:
- **3 billion interactions** since 2018 launch
- Started with 200-250 intents → now 700+
- **50-60% of interactions are now proactive** (AI initiates)
- Average interaction: **48 seconds**
- 98% of users find the information they need
- Critical design change: Evolved from floating button to **search bar paradigm** — dramatically increased adoption among older users

---

## Option 1: AI Overlay/Sheet

### Pattern Description

A floating action button (FAB) or persistent entry point that opens an AI chat interface as a bottom sheet or overlay. The user remains visually aware of their current context while interacting with AI.

### Visual Concept

```
┌─────────────────────────────┐
│  [Balance Card]             │
│  $12,450.32                 │
│                             │
│  [Token List]               │  ← Main UI remains visible
│  • BTC  0.15                │     (dimmed/blurred)
│  • ADA  5,000               │
│                             │
├─────────────────────────────┤
│  ╭─────────────────────────╮│
│  │   🤖 Ask Begin AI       ││  ← Bottom sheet overlay
│  │                         ││
│  │   "Send 100 ADA to..."  ││
│  │   "What's my BTC value?"││
│  │   "Explain this tx"     ││
│  │                         ││
│  │   [💬 Type a message... ]│
│  ╰─────────────────────────╯│
└─────────────────────────────┘
       [🤖]  ← FAB entry point
    (before sheet opens)
```

### Examples Doing This Well

**1. Bank of America's Erica**
- Started as floating button, evolved to search bar entry
- Non-modal: doesn't block primary interface
- Seamlessly hands off to human when needed
- **Key learning**: Search bar paradigm increased adoption across age groups

**2. Zendesk/Intercom Widgets**
- Bottom-right floating chatbot pattern
- Non-intrusive but always accessible
- Context-aware based on current page

**3. Google Maps Bottom Sheet**
- Non-modal sheet showing destination details
- User can still interact with map while sheet is visible
- Expandable to full screen

### UX Best Practices (from NN/Group Research)

1. **Support dismissal via Back button/gesture** — Users expect standard navigation
2. **Include visible Close button** — Don't rely solely on swipe gestures
3. **Don't stack multiple sheets** — Causes disorientation
4. **Keep interactions short** — Bottom sheets are for quick tasks, not long workflows
5. **Maintain spatial context** — Keep background partially visible

### Pros for Begin Wallet

| Advantage | Why It Matters |
|-----------|---------------|
| ✅ Low learning curve | Users understand chat interfaces |
| ✅ Non-disruptive | Doesn't replace familiar UI |
| ✅ Quick access | One tap to AI from any screen |
| ✅ Graceful degradation | Traditional UI always available |
| ✅ Mobile-optimized | Bottom sheets are thumb-friendly |

### Cons & Risks

| Risk | Mitigation |
|------|------------|
| ⚠️ Limited screen real estate | Expandable sheet for complex queries |
| ⚠️ Discoverability for new users | Onboarding tutorial + subtle animations |
| ⚠️ Users may ignore it | Proactive suggestions (like Erica's 50-60% proactive rate) |
| ⚠️ Interrupts flow if modal | Use non-modal sheet by default |

### Implementation Complexity

**Difficulty: Medium**
- Bottom sheet component (standard mobile pattern)
- Chat interface with streaming responses
- Context injection (current screen, selected asset, etc.)
- Backend: LLM integration with wallet action APIs

**Estimated effort**: 4-6 weeks for MVP

---

## Option 2: AI Mode Toggle

### Pattern Description

A switch or toggle that transforms the entire UI between "Traditional Mode" (buttons, tabs, standard navigation) and "AI Mode" (conversational interface as primary interaction).

### Visual Concept

```
Traditional Mode:              AI Mode:
┌─────────────────────┐       ┌─────────────────────┐
│  [🔘 AI Mode]       │       │  [🔘 Classic]       │
│                     │       │                     │
│  $12,450.32         │       │  🤖 Begin AI        │
│                     │       │                     │
│  [Send] [Receive]   │       │  "What would you    │
│  [Swap] [Stake]     │       │   like to do?"      │
│                     │       │                     │
│  ┌─────────────────┐│       │  Suggestions:       │
│  │ BTC    0.15     ││       │  • Check balance    │
│  │ ADA    5,000    ││       │  • Send crypto      │
│  │ SOL    25.5     ││       │  • View portfolio   │
│  └─────────────────┘│       │                     │
│                     │       │  [💬 Ask anything...]│
│  [Home][NFT][Set]   │       │                     │
└─────────────────────┘       └─────────────────────┘
```

### Examples Doing This Well

**1. ChatGPT Canvas (Left Panel Pattern)**
- Chat on left, output/workspace on right
- Clear separation of input/output paradigms
- Supports iterative refinement

**2. You.com AI Modes**
- Multiple "modes" (Genius, Research, Create)
- Each mode optimizes for different tasks
- Clear mode indicator

**3. Lovable.dev**
- Full AI-first interface for app building
- Traditional UI elements available but secondary
- Chat drives primary actions

### UX Best Practices

1. **Clear mode indicator** — User must always know which mode they're in
2. **Instant switching** — No loading state when toggling
3. **State preservation** — Context survives mode switches
4. **Consistent entry points** — Same actions available in both modes

### Pros for Begin Wallet

| Advantage | Why It Matters |
|-----------|---------------|
| ✅ Full-screen AI experience | Maximum context for complex queries |
| ✅ Clean separation | No UI competition |
| ✅ Power user appeal | Feels like "pro mode" |
| ✅ Distinct identity | "Begin AI Mode" as feature |

### Cons & Risks

| Risk | Mitigation |
|------|------------|
| ⛔ Mode confusion | Persistent mode indicator |
| ⛔ Higher learning curve | Both UIs must be learned |
| ⛔ Feature parity challenge | AI mode must match button capabilities |
| ⛔ Loss of familiar anchors | Users may feel lost |
| ⛔ Mobile screen constraints | Chat-only UI needs careful design |

### Implementation Complexity

**Difficulty: High**
- Full alternative UI layer
- All actions must work via both paradigms
- State synchronization between modes
- Significant QA overhead (2x surface area)

**Estimated effort**: 8-12 weeks

### Verdict

**Not recommended as initial approach** — High complexity with risk of fragmenting the user experience. Better suited for desktop-first apps or power-user tools.

---

## Option 3: Contextual AI

### Pattern Description

AI assistance appears only in specific contexts where it adds clear value:
- Complex multi-step transactions
- DeFi protocol interactions
- NFT research and purchasing
- Transaction explanation/simulation
- Error states and recovery

### Visual Concept

```
Standard Send Screen:          With Contextual AI:
┌─────────────────────┐       ┌─────────────────────┐
│  Send ADA           │       │  Send ADA           │
│                     │       │                     │
│  To: addr1q...      │       │  To: addr1q...      │
│  Amount: 500        │       │  Amount: 500        │
│                     │       │                     │
│  [Review]           │       │  ┌─────────────────┐│
│                     │       │  │ 💡 AI Insight    ││
│                     │       │  │                 ││
│                     │       │  │ This address has││
│                     │       │  │ received 12 txs ││
│                     │       │  │ in the last 30  ││
│                     │       │  │ days. Looks like││
│                     │       │  │ an active wallet││
│                     │       │  │ [Learn more]    ││
│                     │       │  └─────────────────┘│
│                     │       │                     │
│                     │       │  [Review]           │
└─────────────────────┘       └─────────────────────┘
```

### Where Context Triggers AI

| Context | AI Value |
|---------|----------|
| **Unknown contract interaction** | Simulation + risk analysis |
| **Large transaction (>threshold)** | Double-check prompts |
| **DeFi protocol connection** | Explain permissions requested |
| **NFT purchase** | Price history, authenticity check |
| **Swap with high slippage** | Alternative route suggestions |
| **Error/failed transaction** | Explain what went wrong |
| **New token received** | "What is this token?" |

### Examples Doing This Well

**1. Notion AI / Grammarly (Inline Overlays)**
- AI appears when user selects text or triggers specific action
- Context-aware suggestions
- Non-disruptive: user controls when to engage

**2. GitHub Copilot (Right Panel Expert)**
- AI available but not intrusive
- Deep context awareness of current file/project
- On-demand activation

**3. Transaction Simulation (Crypto-specific)**
- Blowfish, Pocket Universe, Wallet Guard
- AI analyzes transactions before signing
- Critical for scam prevention

### UX Best Practices

1. **Clear trigger conditions** — User should understand why AI appeared
2. **Dismissable** — One tap to hide suggestion
3. **Non-blocking** — Suggestions don't prevent action
4. **Progressive disclosure** — Summary first, details on demand
5. **Learn from dismissals** — Reduce suggestions user consistently ignores

### Pros for Begin Wallet

| Advantage | Why It Matters |
|-----------|---------------|
| ✅ Minimal disruption | AI only when helpful |
| ✅ Educational | Teaches users gradually |
| ✅ Safety-focused | Perfect for scam prevention |
| ✅ Builds trust incrementally | Earns AI credibility |
| ✅ Lower AI costs | Not processing every interaction |

### Cons & Risks

| Risk | Mitigation |
|------|------------|
| ⚠️ Discoverability | Users may not know AI exists |
| ⚠️ Complexity in defining triggers | Start with high-value contexts |
| ⚠️ May feel inconsistent | Clear AI branding when present |
| ⚠️ Can't ask arbitrary questions | Combine with Option 1 (overlay) |

### Implementation Complexity

**Difficulty: Medium-High**
- Context detection logic
- Transaction simulation infrastructure
- Risk scoring models
- Contract analysis capabilities

**Estimated effort**: 6-10 weeks (depending on scope)

---

## Option 4: Hybrid Cards

### Pattern Description

AI-generated insight cards integrated directly into existing screens (home, portfolio, transaction history). Cards appear alongside standard UI elements, offering suggestions, insights, and actions.

### Visual Concept

```
┌─────────────────────────────┐
│  Good morning! ☀️            │
│                             │
│  Total Balance              │
│  $12,450.32  ↑ 3.2%         │
│                             │
│  ┌─────────────────────────┐│
│  │ 💡 AI Insight            ││  ← AI Card
│  │                         ││
│  │ Your ADA stake rewards  ││
│  │ are ready to claim!     ││
│  │ Estimated: 45.2 ADA     ││
│  │                         ││
│  │ [Claim Now] [Dismiss]   ││
│  └─────────────────────────┘│
│                             │
│  ┌───────┐ ┌───────┐       │
│  │ BTC   │ │ ADA   │        │  ← Standard cards
│  │ 0.15  │ │ 5,000 │        │
│  └───────┘ └───────┘       │
│                             │
│  ┌─────────────────────────┐│
│  │ 🔔 Price Alert           ││  ← AI Card
│  │ BTC is up 5% today.     ││
│  │ Your target of $100k is ││
│  │ within reach!           ││
│  │ [Set Alert] [Learn More]││
│  └─────────────────────────┘│
│                             │
│  [🏠 Home] [💱] [⚙️]         │
└─────────────────────────────┘
```

### Card Types for Begin

| Card Type | Content | Action |
|-----------|---------|--------|
| **Stake Rewards** | Claimable rewards amount | One-tap claim |
| **Price Insights** | Notable price movements | Set alerts |
| **Transaction Summary** | "You spent 500 ADA on NFTs this week" | View details |
| **Security Alert** | "We detected a phishing site" | Learn more |
| **Optimization Tips** | "Consolidate UTXOs to save on fees" | Execute |
| **DeFi Opportunities** | Yield comparisons | Compare options |
| **New Features** | "Try our new swap feature" | Try now |

### Examples Doing This Well

**1. Apple Wallet (Proactive Suggestions)**
- Cards for boarding passes appear at right time
- Location and time-aware
- Actionable with one tap

**2. Google Feed / Discover**
- Personalized content cards
- Mix of interests and utility
- Feedback loop (not interested, show more)

**3. Bank of America Insights**
- Spending pattern analysis
- Bill reminders
- "You spent more on dining this month"

### UX Best Practices

1. **Limited quantity** — Max 2-3 AI cards visible at once
2. **Dismissable** — User can remove cards they don't want
3. **Actionable** — Each card has clear CTA
4. **Relevance decay** — Cards disappear if not acted on
5. **Learn preferences** — Track which card types user engages with

### Pros for Begin Wallet

| Advantage | Why It Matters |
|-----------|---------------|
| ✅ Native feel | Blends into existing UI |
| ✅ Passive discovery | Users find AI naturally |
| ✅ Highly actionable | One-tap from insight to action |
| ✅ Personalization | Cards adapt to user behavior |
| ✅ Gradual introduction | Can start with few card types |

### Cons & Risks

| Risk | Mitigation |
|------|------------|
| ⚠️ Screen clutter | Strict limits on card count |
| ⚠️ "Ad" perception | Clear AI branding, not promotional |
| ⚠️ Notification fatigue | Respect user dismissals |
| ⚠️ Limited depth | Cards are summaries, need detail views |

### Implementation Complexity

**Difficulty: Medium**
- Card component system
- Personalization engine (user behavior tracking)
- Card prioritization logic
- Backend insights generation

**Estimated effort**: 4-6 weeks

---

## Option 5: Voice-First

### Pattern Description

Voice assistant integration for hands-free wallet operations. User can speak commands like "Send 100 ADA to Nick" or "What's my Bitcoin balance?"

### Visual Concept

```
┌─────────────────────────────┐
│                             │
│                             │
│         🎤                   │
│    "Listening..."           │
│                             │
│   ┌─────────────────────┐   │
│   │ 🔊 "Your Bitcoin    │   │
│   │ balance is 0.15 BTC,│   │
│   │ worth $15,234"      │   │
│   └─────────────────────┘   │
│                             │
│   Suggested commands:       │
│   • "Send crypto"           │
│   • "Check my rewards"      │
│   • "Show recent activity"  │
│                             │
│         [🎤 Tap to speak]   │
│                             │
└─────────────────────────────┘
```

### Examples Doing This Well

**1. Apple Siri + Apple Pay**
- "Send $50 to Sarah with Apple Pay"
- Biometric confirmation required
- Visual confirmation in Wallet app

**2. Capital One + Alexa**
- "Alexa, ask Capital One for my balance"
- Natural language understanding
- Context-aware responses

**3. ICICI Bank iPal**
- Available on Alexa and Google Assistant
- Account balance, credit card details
- Security via registered mobile + PIN

### UX Best Practices (from Fintech Voice UX Research)

1. **Biometric + Voice** — Never voice-only for transactions
2. **Visual confirmation** — Always show what's happening on screen
3. **Natural language** — Accept multiple phrasings for same intent
4. **Context awareness** — "Pay my bill" → most recent or recurring
5. **Private mode** — Display sensitive info instead of speaking aloud
6. **Quick undo** — Allow immediate reversal of voice-initiated actions

### Crypto-Specific Considerations

| Challenge | Solution |
|-----------|----------|
| **Address pronunciation** | "Send to Nick" (address book) or show address for confirmation |
| **Amount precision** | Visual confirmation of exact amount |
| **Network selection** | Default network with verbal override |
| **Security** | Voice + Face ID/fingerprint for transactions |
| **Public settings** | Private mode by default |

### Pros for Begin Wallet

| Advantage | Why It Matters |
|-----------|---------------|
| ✅ Accessibility | Helps users with motor impairments |
| ✅ Convenience | Hands-free operation |
| ✅ Speed | Faster than navigating UI |
| ✅ Differentiation | Few crypto wallets offer this |

### Cons & Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| ⛔ Security concerns | High | Always require biometric confirmation |
| ⛔ Privacy in public | High | Private mode default |
| ⛔ Accuracy for amounts | High | Visual confirmation required |
| ⛔ Limited use cases | Medium | Focus on read operations first |
| ⛔ Development complexity | High | Consider third-party integration |

### Implementation Complexity

**Difficulty: High**
- Speech-to-text integration (Apple/Google APIs)
- Natural language understanding for crypto intents
- Secure voice-to-action pipeline
- Multimodal feedback system

**Estimated effort**: 10-14 weeks

### Verdict

**Not recommended for Phase 1** — High complexity and security considerations. Better suited as a later enhancement once core AI features are established. Start with voice for read-only operations (balance checks, portfolio summary).

---

## Comparative Analysis

### Quick Comparison Matrix

| Criteria | Overlay/Sheet | Mode Toggle | Contextual | Hybrid Cards | Voice |
|----------|--------------|-------------|------------|--------------|-------|
| **Implementation Effort** | Medium | High | Medium-High | Medium | High |
| **User Learning Curve** | Low | High | Low | Low | Medium |
| **Discoverability** | Good | Excellent | Limited | Excellent | Good |
| **Mobile UX Fit** | Excellent | Fair | Good | Excellent | Good |
| **Traditional UI Compatibility** | Excellent | Poor | Excellent | Excellent | Good |
| **Power User Appeal** | Good | Excellent | Good | Fair | Good |
| **Casual User Appeal** | Good | Poor | Excellent | Excellent | Fair |
| **Security Integration** | Easy | Easy | Natural | Easy | Complex |
| **Crypto-Specific Value** | Good | Good | Excellent | Excellent | Fair |

### User Persona Fit

| Persona | Best Options | Avoid |
|---------|--------------|-------|
| **Crypto Newbie** | Hybrid Cards, Contextual | Mode Toggle |
| **Power Trader** | Overlay/Sheet, Mode Toggle | — |
| **HODLer** | Hybrid Cards, Contextual | Voice (security) |
| **DeFi Explorer** | Contextual, Overlay/Sheet | — |
| **Mobile-First User** | Hybrid Cards, Overlay/Sheet | Mode Toggle |

---

## Recommended Approach

### Phase 1: Foundation (Weeks 1-6)

**Implement: AI Overlay/Sheet + Basic Hybrid Cards**

```
Priority Features:
1. "Ask Begin AI" FAB/button → Bottom sheet chat
2. 2-3 AI insight card types on home screen:
   - Stake rewards notification
   - Price movement alerts
   - Security warnings
```

**Why this combination:**
- Covers both active queries (sheet) and passive discovery (cards)
- Low disruption to existing UX
- Manageable scope
- Gathers user behavior data for Phase 2

**Success Metrics:**
- AI sheet open rate
- Card engagement rate
- Query completion rate
- User retention with AI features

### Phase 2: Intelligence (Weeks 7-14)

**Add: Contextual AI for High-Value Moments**

```
Priority Contexts:
1. Transaction simulation before signing
2. Unknown contract warnings
3. Large transaction double-checks
4. Error state explanations
```

**Why now:**
- Phase 1 data shows where users need help most
- Transaction simulation is highest-value crypto AI feature
- Builds trust through safety features

### Phase 3: Enhancement (Weeks 15+)

**Evaluate and add based on data:**
- More sophisticated hybrid cards (personalized recommendations)
- Voice for read-only operations
- Proactive suggestions (Erica-style 50%+ proactive rate)

### Anti-Recommendations

**What NOT to do:**

1. ❌ **Don't launch with Mode Toggle** — Too complex, fragments UX
2. ❌ **Don't make AI blocking** — All AI should be optional/dismissable
3. ❌ **Don't use generative AI for transaction execution** — Deterministic for actions, generative for explanations only
4. ❌ **Don't hide traditional buttons** — AI augments, doesn't replace
5. ❌ **Don't over-notify** — Respect dismissals, limit card frequency

---

## Implementation Roadmap

### Week 1-2: Architecture
- Design AI service layer architecture
- Define context injection points
- Create component library for AI UI elements

### Week 3-4: Core Chat
- Implement bottom sheet component
- Build chat interface with streaming
- Connect to LLM backend
- Add basic wallet context (balance, assets)

### Week 5-6: Insight Cards
- Build card component system
- Implement 2-3 card types
- Add personalization logic
- Deploy A/B testing framework

### Week 7-10: Contextual AI
- Transaction simulation integration
- Contract analysis service
- Risk scoring system
- Inline warning components

### Week 11-14: Refinement
- Iterate based on user data
- Expand card types
- Improve NLU for chat
- Performance optimization

### Week 15+: Advanced Features
- Voice (read-only first)
- Proactive suggestions
- Cross-chain AI context
- Advanced personalization

---

## Appendix: Mockup Descriptions

### Home Screen with AI Elements

```
┌─────────────────────────────────┐
│ ☀️ Good morning, Nick            │
│                                 │
│ ┌─────────────────────────────┐ │
│ │ Total Balance               │ │
│ │ $12,450.32        ↑ 3.2%    │ │
│ │                             │ │
│ │ [Send] [Receive] [Swap]     │ │
│ └─────────────────────────────┘ │
│                                 │
│ 💡 AI Insight                    │
│ ┌─────────────────────────────┐ │
│ │ You have 45.2 ADA in stake  │ │
│ │ rewards ready to claim!     │ │
│ │ [Claim Now]     [Dismiss]   │ │
│ └─────────────────────────────┘ │
│                                 │
│ Your Assets                     │
│ ┌───────┐ ┌───────┐ ┌───────┐  │
│ │₿ BTC  │ │◇ ADA  │ │◎ SOL  │  │
│ │ 0.15  │ │ 5,000 │ │ 25.5  │  │
│ └───────┘ └───────┘ └───────┘  │
│                                 │
│ 🔔 BTC is up 5% today!           │
│    [Set Price Alert →]          │
│                                 │
│ [🏠] [💎 NFT] [📊] [⚙️]  [🤖]    │
│                          ↑      │
│                    AI Button    │
└─────────────────────────────────┘
```

### AI Sheet (Expanded)

```
┌─────────────────────────────────┐
│ ┌───────────────────────────┐   │
│ │  🤖 Begin AI              ✕│  │
│ │                           │   │
│ │  How can I help you today?│   │
│ │                           │   │
│ │  ┌─────────────────────┐  │   │
│ │  │ "How do I stake ADA?"│  │   │
│ │  └─────────────────────┘  │   │
│ │  ┌─────────────────────┐  │   │
│ │  │ "Send 100 ADA to..." │  │   │
│ │  └─────────────────────┘  │   │
│ │  ┌─────────────────────┐  │   │
│ │  │ "Explain this tx"    │  │   │
│ │  └─────────────────────┘  │   │
│ │                           │   │
│ │  Recent: "What is my...   │   │
│ │                           │   │
│ │  ┌─────────────────────┐  │   │
│ │  │ Type a message...  🎤│  │   │
│ │  └─────────────────────┘  │   │
│ └───────────────────────────┘   │
└─────────────────────────────────┘
```

### Contextual AI Warning (Transaction)

```
┌─────────────────────────────────┐
│ Review Transaction              │
│                                 │
│ Sending: 500 ADA                │
│ To: addr1qxy7...                │
│ Fee: 0.17 ADA                   │
│                                 │
│ ⚠️ AI Security Check             │
│ ┌─────────────────────────────┐ │
│ │ This address was created    │ │
│ │ today and has no history.   │ │
│ │                             │ │
│ │ Common in phishing scams.   │ │
│ │ Please verify the recipient.│ │
│ │                             │ │
│ │ [I've Verified] [Cancel]    │ │
│ └─────────────────────────────┘ │
│                                 │
│       [Confirm Transaction]     │
│                                 │
└─────────────────────────────────┘
```

---

## Sources & References

### Research Sources

1. **NN/Group** — "Bottom Sheets: Definition and UX Guidelines" (2024)
2. **PixelPlex** — "AI Crypto Wallet Development Overview" (2025)
3. **UX Collective** — "Where should AI sit in your UI?" (2025)
4. **Customer Experience Dive** — "How Bank of America's Erica raised the stakes for virtual assistants" (2025)
5. **Klarna** — AI Assistant and ChatGPT Integration announcements (2023-2024)
6. **The Block** — "'The smartest wallet wins': Industry leaders on AI and UX" (2025)
7. **UXDA** — "Fintech App Design: 10 Latest Mobile Banking Trends" (2025)
8. **Medium** — "Voice UX & Conversation Design in Fintech" (2025)

### Industry Benchmarks

- Bank of America Erica: 3 billion interactions, 700+ intents, 48-sec average interaction
- Klarna AI: First fintech ChatGPT integration, shopping assistant
- Phantom Wallet: Best-in-class traditional crypto wallet UX benchmark

### Key Statistics

- 2/3 of customers want full banking from mobile (Forrester)
- <30% of consumers trust AI chatbots for financial info (J.D. Power)
- 50-60% of Erica interactions are AI-initiated (proactive)
- $2.17B in crypto stolen in 2025 — AI scam detection critical

---

*Document prepared for Begin Wallet by Arlo, February 2026*
*Version: 1.0*
