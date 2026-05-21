# KL-Grounded Gamification Engine — Comprehensive Design Document

> **Project:** Smart Plug Power Management Ecosystem — UM Technothon 2026  
> **Scope:** Complete gamification layer design for the Community Grid  
> **Language:** BM-English bilingual, grounded in Kuala Lumpur geography and Malaysian culture  
> **Target:** Flutter mobile app demo prototype

---

## Table of Contents

1. [Philosophy & Design Principles](#1-philosophy--design-principles)
2. [KL Cultural Grounding — Why Localization Matters](#2-kl-cultural-grounding--why-localization-matters)
3. [The Core Game Loop](#3-the-core-game-loop)
4. [League System — Three Housing Tiers](#4-league-system--three-housing-tiers)
5. [Level & XP System — Malaysianized Progression](#5-level--xp-system--malaysianized-progression)
6. [Badge System — 12 Achievements Rooted in KL Life](#6-badge-system--12-achievements-rooted-in-kl-life)
7. [Festival Challenge Calendar — The Malaysian Year](#7-festival-challenge-calendar--the-malaysian-year)
8. [Leaderboard Design — Competition with Dignity](#8-leaderboard-design--competition-with-dignity)
9. [Rewards & Incentives — Tangible KL Value](#9-rewards--incentives--tangible-kl-value)
10. [Social Features — Sharing & Virality](#10-social-features--sharing--virality)
11. [UX/UI Integration — Widgets & Screens](#11-uxui-integration--widgets--screens)
12. [Behavioral Psychology — Why This Works](#12-behavioral-psychology--why-this-works)
13. [Demo Implementation Guide](#13-demo-implementation-guide)
14. [Future Expansions](#14-future-expansions)

---

## 1. Philosophy & Design Principles

### Core Thesis

Most energy-saving apps fail because they treat energy conservation as a data problem. They show kilowatt-hour graphs, usage trends, and percentage breakdowns — then wonder why 74% of users abandon them within 30 days.

Our gamification engine treats energy saving as a **human behavior problem**. It applies the same psychological loops that make fitness apps, language-learning apps, and social games addictive — but rebuilt entirely around Malaysian cultural identity, KL geography, and the reality of how urban Malaysians actually live.

### Five Design Principles

| # | Principle | What It Means | Counter-Example |
|---|-----------|---------------|-----------------|
| **1** | **Identity over Data** | Every metric (kWh, RM, CO₂) is translated into identity markers: levels, badges, titles. The user is not "a household consuming 420W" — they are a "Wira Hijau in TTDI with a 7-day streak." | A dashboard showing a line chart of daily wattage |
| **2** | **Cultural Resonance over Generic Tropes** | No "Vampire Hunter" or "Night Owl" badges. Every mechanic references something a KL resident immediately recognizes: Ramadan, kopitiam, mamak, haze season, PPR, Balik Kampung, taugeh, Dato' | Western gamification tropes (dragons, vampires, castles) applied to energy |
| **3** | **Community over Competition** | The primary loop is "save energy → help a neighbor." Competition exists (leaderboards, challenges) but always in service of collective good. Shame-based mechanics (worst-performer leaderboards) are deliberately excluded | Step challenges that only reward the top performer |
| **4** | **Inclusivity through Separation** | A Mont Kiara penthouse should not compete with a PPR Lembah Subang flat. Three leagues ensure you compete with economic peers. Cross-league donation carries multiplier rewards for the donor | One-size-fits-all leaderboards that privilege the wealthy |
| **5** | **Opt-In, Not Opt-Out** | Every gamification feature is opt-in with a single toggle. The app works perfectly without any gamification. No dark patterns, no nag notifications, no FOMO manipulation. The streak nudge ("Don't break your 7-day streak!") is the only gentle prompt — and it can be disabled | Apps that guilt-trip users for not engaging |

### The Emotional Arc

The user's journey through the gamification system follows a deliberate emotional arc:

```
Discovery        →   "Oh, I earned a badge just for saving during Ramadan?"
Competence       →   "I'm Level 3 now — Celik Tenaga. I actually understand my energy."
Belonging        →   "TTDI vs Damansara this month. My kawasan needs me to save."
Generosity       →   "I donated 5 credits to PPR Kerinchi. That's RM 1.09 off someone's bill."
Identity         →   "I'm Aisyah, Wira Hijau of TTDI. 14-day streak. Jiran Terbaik."
```

Each stage unlocks the next. A user cannot reach Generosity without first passing through Competence.

---

## 2. KL Cultural Grounding — Why Localization Matters

### The Generic Problem

Every existing smart plug app (TP-Link Kasa, Wemo, Eve Energy, stock Sonoff) uses identical gamification:
- "You saved 12 kWh this week!" 
- "Energy Saver Badge earned!"
- "Top 10% of users!"

These mean nothing to a KL resident because:
1. **kWh is abstract.** Most Malaysians understand Ringgit, not kilowatt-hours. 12 kWh = RM 2.62 at TNB lifeline rate. Saying "You saved RM 2.62" would be better — but still misses the social layer.
2. **"Top 10%" is faceless.** Who are the other 90%? Are they my neighbors in TTDI? Are they in Mont Kiara? Are they in Penang? Without geography, competition has no meaning.
3. **"Energy Saver Badge" is soulless.** It could be awarded by any app on Earth. It has no cultural memory. Compare: "Anak Bulan Hero" — earned by saving for 30 consecutive days during Ramadan. That badge carries the weight of a shared cultural experience.

### What KL Grounding Unlocks

| Generic Element | KL-Grounded Replacement | Why It Resonates |
|----------------|------------------------|-----------------|
| "Energy Saver" badge | **Anak Bulan Hero** 🌙 | Every Malaysian Muslim knows the altered sahur/iftar rhythm of Ramadan. Saving energy during this month is naturally achievable — the badge validates what they already do. |
| "Top Donor" title | **Jiran Terbaik** 🛡️ | "Jiran" (neighbor) carries cultural weight in Malaysia. Gotong-royong (community self-help) is a national value. Being called "Best Neighbor" means something. |
| "Monthly Challenge: Save 20 kWh" | **Haze Shield (Julai)** | KL residents dread the annual jerebu. When the API hits 200, you stay home running AC and air purifiers. Saving energy during haze is genuinely hard — the badge acknowledges that difficulty. |
| "Level 7: Energy Legend" | **Dato' Jimat** 👑 | "Dato'" is the most recognized honorific in Malaysia. Pairing it with "Jimat" (save/frugal) creates a title that is simultaneously prestigious and self-deprecating — very Malaysian humor. |
| Generic neighborhood names | **Kawasan sebenar: TTDI, Bangsar, Cheras** | These are real places with real identities. TTDI residents feel different from Bangsar residents. A challenge between them taps into existing social dynamics. |
| "You saved 5% this month" | **"Anda jimat RM 18.30 — setara 84 kWh pada kadar lifeline TNB"** | References the real TNB tariff structure that every Malaysian bill-payer knows (200 kWh lifeline band, RM 0.218/kWh). |

### The Multi-Ethnic Dimension

Malaysia is multi-ethnic (Malay, Chinese, Indian, Indigenous). The gamification system deliberately:

- **Uses BM as the primary interface language** (national language, understood by all) with English subtitles on key elements
- **Includes festivals from all major communities:** Ramadan/Raya (Malay-Muslim), Chinese New Year/Chap Goh Meh (Chinese), Deepavali (Indian), Gawai-Kaamatan (East Malaysia Indigenous)
- **Names personas across ethnicities:** Aisyah (Malay), Kumar (Indian), Mei Ling (Chinese), Raj (Indian), Fatimah (Malay)
- **References shared KL experiences that transcend ethnicity:** haze season, mamak culture, kopitiam, LRT commutes, Pasar Malam — these are KL experiences, not ethnic ones

---

## 3. The Core Game Loop

### Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      THE CORE GAME LOOP                          │
│                                                                 │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────────┐    │
│  │  ACTION   │ ──→ │   REWARD     │ ──→ │   PROGRESSION    │    │
│  │           │     │              │     │                  │    │
│  │ Plug in   │     │ XP + Credits │     │ Level up         │    │
│  │ Save kWh  │     │ Badge unlock │     │ Title change     │    │
│  │ Donate    │     │ Streak count │     │ Unlock features  │    │
│  │ Refer     │     │ RM equivalent│     │ Badge collection │    │
│  └──────────┘     └──────────────┘     └────────┬─────────┘    │
│                                                  │              │
│                                                  ▼              │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────────┐    │
│  │  SOCIAL   │ ←── │  RECOGNITION │ ←── │   COMPETITION    │    │
│  │           │     │              │     │                  │    │
│  │ Share card│     │ Leaderboard  │     │ Kawasan vs       │    │
│  │ WhatsApp │      │ position     │     │ kawasan          │    │
│  │ Recruit   │     │ Donor status │     │ Monthly challenge│    │
│  └──────────┘     └──────────────┘     └──────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Loop Breakdown

**Phase 1: Action (Automatic + Deliberate)**

The system captures energy-saving actions seamlessly. Most are passive — the user just lives their life:

| Action | Trigger | User Effort |
|--------|---------|-------------|
| Save 1 kWh below baseline | Automatic (plug reports to MQTT) | Zero — happens in background |
| Complete a day below baseline | Automatic at midnight | Zero |
| Auto-cutoff triggers (vampire power) | System detects standby draw > 30 min | Zero |
| Donate credits | User taps "Derma Kredit" | 3 taps (modal → slider → confirm) |
| Refer a neighbor | User taps "Jemput Jiran" | Share sheet → select contact |
| Resolve an anomaly | User taps anomaly alert → views detail | 2 taps |

**Phase 2: Reward (Immediate + Delayed)**

Rewards are designed with variable-ratio scheduling — the most addictive reinforcement pattern:

| Reward Type | Delivery | Example |
|-------------|----------|---------|
| **Immediate micro-reward** | Instant | XP counter increments on dashboard after every kWh saved |
| **Daily reward** | Midnight | "Anda jimat 1.2 kWh hari ini. 🔥 7-Hari Rantaian berterusan!" |
| **Threshold reward** | On level-up | Confetti animation + "Tahniah! Anda kini Wira Hijau 🌿" |
| **Surprise reward** | Random interval | "🎉 Lencana istimewa: Kopitiam Regular — 30 hari guna luar waktu puncak!" |
| **Social reward** | Monthly reset | "🏆 TTDI menang! Anda #3 penderma teratas." |

**Phase 3: Progression (Linear + Surprise)**

The user always knows what's next (linear XP bar) but also discovers unexpected achievements (badge unlocks they didn't know about):

- **Linear:** XP bar fills toward next level. Always visible on Settings profile. "560 XP lagi ke Wira Hijau."
- **Surprise:** Badge unlocks appear when conditions are met. The user doesn't know about "Mamak Squad" until they've saved energy on 10 nights between 8PM-12AM — then the badge suddenly unlocks, creating a delight moment.

**Phase 4: Competition (Team-Based, Not Individual)**

Competition is always kawasan vs. kawasan, never individual vs. individual within the same league. This prevents:

- Wealthy users from dominating (the league system handles that)
- Shame dynamics (no one sees that you personally are underperforming)
- Toxic competition (you compete FOR your community, not AGAINST your neighbors)

**Phase 5: Recognition (Public + Private)**

- **Public:** Leaderboard position, kawasan challenge result, donation count
- **Private:** Personal badge collection, level title, streak counter
- **Semi-public:** Impact card (user chooses to share)

**Phase 6: Social (Organic + Designed)**

- **Organic:** Users share impact cards on WhatsApp/Instagram because they're genuinely proud
- **Designed:** "Jemput Jiran" referral flow, thank-you cards from recipients, weekly community digest

### Feedback Loops

| Loop Type | Description | Reinforcement Schedule |
|-----------|-------------|----------------------|
| **Immediate** | Wattage display updates every 1s on dashboard. User turns off AC → sees wattage drop instantly | Continuous |
| **Daily** | "Good morning, Aisyah. You saved RM 1.80 yesterday. 🔥 7-day streak!" | Fixed interval (8 AM push) |
| **Weekly** | Weekly community digest: "TTDI saved 1,200 kWh this week" | Fixed interval (Monday 9 AM) |
| **Monthly** | Challenge result + leaderboard reset | Fixed interval (1st of month) |
| **Variable** | Badge unlocks, level-ups, anomaly resolved | Variable ratio (most addictive) |

---

## 4. League System — Three Housing Tiers

### Design Rationale

Kuala Lumpur has extreme housing inequality. A family in a Mont Kiara penthouse (RM 2M+) has fundamentally different energy realities than a family in PPR Lembah Subang (low-cost flat, RM 124/month rent). Putting them on the same leaderboard is:

1. **Demotivating for PPR residents:** They can never "win" against households with 5x the baseline consumption
2. **Meaningless for affluent residents:** Saving 40 kWh when your baseline is 800 kWh feels trivial. Saving 8 kWh when your baseline is 160 kWh is proportionally massive
3. **Culturally tone-deaf:** It recreates income inequality in a gamification system that should transcend it

The three-league system solves this by matching households to economic peers, while using **cross-league donation** to create upward generosity flow from Condo Cup/Taman League → Rumah Pangsa.

### League Specifications

#### Condo Cup 🏢

| Attribute | Detail |
|-----------|--------|
| **Housing type** | High-rise condominiums and serviced apartments |
| **KL examples** | Mont Kiara, Bangsar South, KLCC, Sri Hartamas, Desa ParkCity, Ampang Hilir, Bukit Jalil |
| **Typical user** | Young professionals (25–40), small families, expats |
| **Unit size** | 800–1,500 sq ft (1–3 bedrooms) |
| **Typical appliance mix** | 2–3 AC split units, fridge, TV, water heater, washer/dryer, gaming console, multiple chargers |
| **Baseline** | Standard rolling 14-day baseline |
| **Monthly baseline range** | 300–800 kWh |
| **Typical bill** | RM 65–220/month |
| **Community formation** | Self-forming by condominium building. Once 15+ units in the same building join, rivalry mode activates with other buildings in the same postcode |

**Demo leaderboard example:**
```
🏢 CONDO CUP — Mont Kiara Division
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🥇 Arcoris MK (47 units)    1,240 kWh saved  🔥 18-day streak
🥈 Verve Suites (62 units)    980 kWh saved  🔥 12-day streak
🥉 Kiara 163 (31 units)       730 kWh saved  🔥 9-day streak
```

#### Taman League 🏘️

| Attribute | Detail |
|-----------|--------|
| **Housing type** | Landed terrace houses (link houses), semi-detached, bungalows |
| **KL examples** | TTDI, Bangsar Park, Damansara Heights, Cheras, Kepong, Setapak, Wangsa Maju, Taman Desa |
| **Typical user** | Families (3–6 members), multi-generational households, retirees |
| **Unit size** | 1,500–3,500 sq ft (3–5 bedrooms) |
| **Typical appliance mix** | 3–5 AC split units (larger capacity), 2 fridges, multiple TVs, water heater, washing machine, dryer, kitchen appliances, pond pump, gate motor |
| **Baseline** | Standard rolling baseline + 15% home-size adjustment |
| **Monthly baseline range** | 500–1,500 kWh |
| **Typical bill** | RM 120–450/month |
| **Community formation** | Self-forming by Taman/postcode. TTDI vs Desa ParkCity = natural rivalry (adjacent communities, similar demographics) |

**Demo rivalry pairings:**
| Pairing | Why It Works |
|---------|-------------|
| **Bangsar vs Damansara Heights** | Adjacent affluent areas, decades-long friendly rivalry among residents |
| **TTDI vs Desa ParkCity** | Adjacent family communities, regularly compared on local Facebook groups |
| **Cheras vs Ampang** | KL's two largest residential districts, distinct identities |
| **Setapak vs Wangsa Maju** | Neighboring mid-range areas with large student populations |

#### Rumah Pangsa League 🏠

| Attribute | Detail |
|-----------|--------|
| **Housing type** | Low-cost flats (PPR — Program Perumahan Rakyat), public housing |
| **KL examples** | PPR Lembah Subang, PPR Kerinchi, PPR Seri Alam, PPR Pantai Dalam, PPR Kg Muhibbah |
| **Typical user** | B40 households (bottom 40% income), single parents, elderly, large families in small units |
| **Unit size** | 500–700 sq ft (2–3 bedrooms) |
| **Typical appliance mix** | 1–2 fans (AC rare or absent), 1 fridge, TV, basic kitchen appliances, phone chargers |
| **Baseline** | TNB lifeline band (first 200 kWh at RM 0.218/kWh). Baseline calibrated specifically for low-consumption households |
| **Monthly baseline range** | 80–200 kWh |
| **Typical bill** | RM 18–50/month (often subsidized) |
| **Community formation** | Self-forming by PPR complex. Blok-level competition within the same PPR |

**Special Rumah Pangsa mechanics:**
- **Donation weight:** Credits donated TO a Rumah Pangsa household carry 2x badge XP for the donor
- **Cross-league visibility:** Condo Cup → Rumah Pangsa donations appear on both league boards simultaneously
- **Recipient dignity:** Donors never see the recipient's full name — only their league and the impact amount. Recipients see the donor's chosen display name in their thank-you card
- **Pathway to donor:** Rumah Pangsa households that accumulate surplus beyond their baseline can donate within their own league — turning recipients into contributors

### League Fairness Mechanics

| Mechanic | Purpose |
|----------|---------|
| **Per-league baseline calculation** | A household's baseline is only compared to itself over time, never to other households in the league. This prevents high-consumption homes from hoarding credits just by owning more appliances |
| **Rolling 14-day window** | Prevents gaming: run high for a week, then cut back to earn credits. The baseline resets downward within 2 weeks |
| **Credit earning cap (15/month)** | Prevents solar-rich homes or extreme savers from dominating cross-league donations |
| **90-day credit expiry** | Encourages circulation. Hoarding credits helps no one |
| **Separate leaderboards per league** | No cross-league comparison. TTDI never sees PPR Kerinchi's absolute numbers — only their own league and the total donations flowing to Rumah Pangsa |

---

## 5. Level & XP System — Malaysianized Progression

### Design Rationale

Traditional level systems use fantasy tropes: "Level 1 Novice → Level 50 Grandmaster." These are culturally neutral — which means they belong to no culture. Our level system uses titles that a KL resident would feel proud to display on their profile.

The progression from **Budak Baru** (new kid) to **Dato' Jimat** (Sir Saves-a-Lot) mirrors the Malaysian social hierarchy but subverts it: you don't earn "Dato'" through wealth or politics — you earn it through consistent energy savings and community donations.

### Full Level Table

| Level | XP Required | Cumulative XP | EN Title | BM Title | Perk Unlocked | Cultural Note |
|-------|------------|--------------|----------|----------|---------------|---------------|
| **1** | 0 | 0 | New Plug | **Budak Baru** | Basic dashboard + 1 manual schedule | Everyone starts as "the new kid on the block" |
| **2** | 500 | 500 | Aware Consumer | **Celik Tenaga** | Custom schedules (unlimited) | "Energy-literate" — you now understand your usage patterns |
| **3** | 1,500 | 2,000 | Smart Saver | **Jimat Cermat** | Anomaly alerts activated | Classic Malaysian frugality phrase; used in schools, households, everywhere |
| **4** | 3,000 | 5,000 | Eco Champion | **Wira Hijau** | Community Grid access + public badge display | "Green Hero" — you're now visible to your community |
| **5** | 6,000 | 11,000 | Power Guardian | **Pendekar Tenaga** | 2x credit earning rate | "Pendekar" = silat warrior — you've mastered energy |
| **6** | 12,000 | 23,000 | Grid Hero | **Jaguh Komuniti** | Custom badge display name + profile highlight | "Community Champion" — your kawasan recognizes you |
| **7** | 25,000 | 48,000 | Energy Legend | **Dato' Jimat** | Priority donation matching + 3x donation multiplier | Honorific "Dato'" + "Save" — the highest honor. Self-deprecating humor: you're a "Dato'" of saving money |

### XP Earning Rules — Detailed

| Action | Base XP | Cooldown | Notes |
|--------|---------|----------|-------|
| 1 kWh saved below baseline | **10 XP** | None (per-kWh) | Primary XP source. Accumulates passively |
| 1 credit donated (any league) | **50 XP** | None | Encourages circulation |
| 1 credit donated to Rumah Pangsa | **100 XP** (2x) | None | Explicitly rewards generosity toward underserved |
| 7-day saving streak | **200 XP** | Once per 7 days | Streak bonus; adds pressure not to break |
| 30-day streak | **1,000 XP** | Once per 30 days | Major milestone |
| 1 jiran referred | **500 XP** | None (per referral) | Both referrer and referred get bonus |
| First anomaly resolved | **100 XP** | Per anomaly (first resolution only) | "Doktor Fridge" — you diagnosed an appliance issue |
| Festival challenge completed | **1,000 XP** | Per challenge | Big bonus for participating in cultural challenges |
| Donation milestone: 50 total credits | **2,500 XP** | Once | Lifetime achievement |
| First month entirely below baseline | **500 XP** | Once | "Zero Hero" path |

### XP Curve Design

The XP curve is deliberately front-loaded to create early wins (dopamine), then stretches to make Level 7 feel genuinely prestigious:

```
Level 1 → 2:  500 XP   (~2 weeks at average saving rate)
Level 2 → 3:  1,500 XP  (~4 weeks)
Level 3 → 4:  3,000 XP  (~8 weeks)
Level 4 → 5:  6,000 XP  (~4 months)
Level 5 → 6:  12,000 XP (~8 months)
Level 6 → 7:  25,000 XP (~18 months)
```

**Why the stretch?** If everyone reaches Level 7 in 3 months, the title "Dato' Jimat" means nothing. It should represent ~2 years of consistent energy consciousness — a genuine lifestyle change, not a sprint.

### XP Visibility

| Location | What's Shown |
|----------|-------------|
| **Dashboard** | Compact: current level badge icon + XP ring (small, bottom of Summary Card) |
| **Settings Profile** | Full: "🏅 Celik Tenaga (Tahap 3) ▓▓▓▓▓▓▓▓▓▓▓░░░ 1,840 / 3,000 XP · 560 XP ke Wira Hijau" |
| **Badge Collection** | Top of screen: level title + full progress bar + next level preview |
| **Community Grid** | StreakBadgeRow: level badge chip included among other badges |

---

## 6. Badge System — 12 Achievements Rooted in KL Life

### Design Philosophy

Each badge tells a story about Kuala Lumpur. The badge name, unlock condition, and visual design all reference something a KL resident immediately recognizes. A badge should make the user think: "Oh, I did that? That's so Malaysian."

Badges are deliberately discoverable but not explicitly listed upfront. The locked badge grid shows greyed-out icons with unlock conditions — users can see what's possible and set goals. But the surprise of unexpected unlocks (e.g., user doesn't realize they've been to the mamak 10 nights until the badge pops) creates delight.

### Earned Badges (Shown in Demo)

#### 1. 7-Hari Rantaian 🔥
| Attribute | Detail |
|-----------|--------|
| **BM Name** | 7-Hari Rantaian |
| **EN Name** | 7-Day Streak |
| **Unlock** | 7 consecutive days below baseline |
| **Visual** | Fire emoji + streak counter number |
| **KL Grounding** | "Rantaian" (chain) is a universal concept — breaking a chain has negative weight across all Malaysian cultures. This badge is always displayed as the leftmost chip in the StreakBadgeRow |
| **Behavioral purpose** | The streak mechanic is the strongest retention tool in gamification. Once a user has a 6-day streak, they will actively manage energy on day 7 to avoid breaking it |
| **Edge Cases** | Streak broken → chip goes grey, subtle shake animation on first view of broken streak. User can restart immediately (the next day counts as day 1 again) |

#### 2. Jiran Terbaik 🛡️
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Jiran Terbaik |
| **EN Name** | Best Neighbor |
| **Unlock** | Donate 15+ total credits, with at least 5 to a Rumah Pangsa household |
| **Visual** | Shield icon with two hands forming a heart |
| **KL Grounding** | "Jiran" (neighbor) is a deeply resonant concept in Malaysian culture. During Raya/CNY/Deepavali, visiting neighbors is expected. Gotong-royong (community self-help) is embedded in the national identity. Being called "Jiran Terbaik" by an app that tracks real impact is deeply meaningful |
| **Behavioral purpose** | Converts abstract "donate credits" into a pursuit: "I'm 3 donations away from Jiran Terbaik." The PPR requirement ensures the badge is earned through generosity toward those who need it, not trading with equally affluent neighbors |
| **Progress visibility** | After each PPR donation, the success modal shows: "🛡️ 3 lagi untuk lencana Jiran Terbaik!" |

#### 3. Anak Bulan Hero 🌙
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Anak Bulan Hero |
| **EN Name** | Ramadan Champion |
| **Unlock** | 30 consecutive days below baseline during Ramadan |
| **Visual** | Crescent moon + green ketupat (diamond-shaped rice cake, iconic Raya food) |
| **KL Grounding** | Ramadan transforms daily rhythms. Sahur (pre-dawn meal) before 5:30 AM means altered energy patterns. Iftar (breaking fast) at ~7:20 PM means cooking peaks at specific times. Many families spend evenings at the mosque or Ramadan bazaars — out of the house, AC off. These natural behavioral shifts make saving during Ramadan genuinely achievable. The badge celebrates what the community already does |
| **Cultural sensitivity** | Available to all users regardless of religion. The badge celebrates participation in a shared cultural rhythm, not religious observance. Non-Muslim users who save during Ramadan month (perhaps coinciding with school holidays or altered work schedules) can earn it too |
| **Timing** | Ramadan 2026 expected: ~Feb 18 – March 19. Challenge activates automatically during the Ramadan month |

#### 4. Celik Tenaga 🍃
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Celik Tenaga |
| **EN Name** | Energy Literate |
| **Unlock** | Reach Level 3 (Jimat Cermat) |
| **Visual** | Green leaf + open eye |
| **KL Grounding** | "Celik" means "aware/opened eyes" — used in "celik IT" (tech literate), "celik wang" (financially literate). "Celik Tenaga" positions energy literacy as a modern Malaysian skill |
| **Behavioral purpose** | This is the first "level badge" — it tells the user that badges can represent progression milestones, not just actions. It seeds curiosity about what other badges exist |

### Locked Badges (Shown as Grey in Demo)

#### 5. Taugeh Champion 🌱
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Taugeh Champion |
| **EN Name** | Bean Sprout Champion |
| **Unlock** | Save RM 50+ on a single monthly bill compared to baseline |
| **Visual** | Bean sprout + Ringgit Malaysia symbol |
| **KL Grounding** | "Taugeh" (bean sprouts) are the cheapest vegetable in any Malaysian market — RM 0.50 for a bag. "Makan taugeh je" (just eating bean sprouts) is Malaysian slang for extreme frugality. The badge embraces this self-deprecating humor: "You're so good at saving, you're a Taugeh Champion." It reframes frugality as a skill, not a deprivation |
| **Humor note** | This badge is intentionally funny. Malaysian culture values humor as a coping mechanism. A badge that makes the user chuckle is more memorable than a serious "Frugality Master" badge |

#### 6. Merdeka Saver 🇲🇾
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Merdeka Saver |
| **EN Name** | Independence Saver |
| **Unlock** | Save 55 kWh during August (Merdeka month) |
| **Visual** | Jalur Gemilang (Malaysian flag) + kWh counter |
| **KL Grounding** | August is Merdeka month (Independence Day: August 31). The number 55 references Malaysia's age — in 2026, Malaysia turns 69, so the target becomes 69 kWh, creating an annual tradition where each year's target increments by 1. The challenge is: "Save 69 kWh for Malaysia's 69th year." This ties personal energy savings to national pride |
| **Deployment** | Challenge launches August 1, closes August 31. All leagues compete simultaneously. National leaderboard shows total kWh saved across all participating kawasan |

#### 7. Kopitiam Regular ☕
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Kopitiam Regular |
| **EN Name** | Coffee Shop Regular |
| **Unlock** | 30-day streak of off-peak usage (below baseline during 12PM–4PM peak hours) |
| **Visual** | Coffee cup (kopi tarik style) + clock showing 2 PM |
| **KL Grounding** | During KL's hottest hours (12PM–4PM), staying home means running AC at full blast. But KL residents have a beautiful alternative: the air-conditioned kopitiam. Whether it's a PappaRich, Old Town White Coffee, or a traditional kedai kopi, escaping to a shared air-conditioned space is both social and energy-efficient. The badge recognizes this KL lifestyle as an energy strategy |
| **Behavioral purpose** | Reframes going out (which costs money at the kopitiam) as an energy-saving action. The math: "Kopi tarik = RM 3.50. Home AC for 4 hours = RM 2.40. Net cost: RM 1.10 for a social experience and a cooler home when you return." The badge makes this trade-off visible |

#### 8. Mamak Squad 🫓
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Mamak Squad |
| **EN Name** | Mamak Squad |
| **Unlock** | Save energy on 10 nights between 8PM–12AM (nighttime savings) |
| **Visual** | Roti canai on a plate + crescent moon |
| **KL Grounding** | The mamak stall is KL's quintessential nightlife. Open until 3 AM, serving roti canai, teh tarik, and maggi goreng. When KL residents are at the mamak at 10 PM, their home AC is off. This badge recognizes that KL's social culture IS an energy-saving culture — you just never framed it that way before |
| **Behavioral purpose** | Same as Kopitiam Regular but for nightlife. Makes the invisible visible: "Your 10 mamak nights saved ~28 kWh = RM 6.10. Teh tarik costs ~RM 1.80. You're basically breaking even on your social life." |

#### 9. Balik Kampung Shutdown 🚗
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Balik Kampung Shutdown |
| **EN Name** | Hometown Shutdown |
| **Unlock** | All plugs auto-off during a festive exodus week (Raya, CNY, or Deepavali) |
| **Visual** | House with arrow pointing away + luggage |
| **KL Grounding** | "Balik kampung" (returning to hometown) is the great Malaysian migration. During Hari Raya, Chinese New Year, and Deepavali, KL empties. Millions leave the city. Homes sit empty for 3–7 days. This badge triggers when the system detects all plugs off for 3+ consecutive days during a recognized festival period — validating the user's exodus as an energy achievement |
| **Technical trigger** | System detects zero consumption for 72+ consecutive hours during a pre-configured festival window. "Leaving Home" mode auto-activates, turning off all non-whitelisted plugs. Badge awarded on return when first plug comes back online |
| **Multi-festival** | Triggers for Raya (April), CNY (January–February), and Deepavali (October–November). Badge shows the festival icon specific to which exodus triggered it |

#### 10. LRT Warrior 🚆
| Attribute | Detail |
|-----------|--------|
| **BM Name** | LRT Warrior |
| **EN Name** | Transit Warrior |
| **Unlock** | 30 days where home energy drops on days the user commutes via LRT/MRT (detected by phone location + energy pattern) |
| **Visual** | Train + downward arrow |
| **KL Grounding** | KL's LRT, MRT, and Monorail network has expanded dramatically (MRT Putrajaya Line opened 2023). Taking public transit means your home AC is off all day — massive energy savings. The badge gamifies the already-growing transit culture |
| **Detection** | Phone GPS detects presence at LRT/MRT stations during commuting hours. Cross-referenced with home energy data showing "away" pattern (>4 hours of near-zero consumption during commute hours). Privacy: station data stays on-device; only the badge trigger (yes/no) syncs |

#### 11. PPR Champion 🏅
| Attribute | Detail |
|-----------|--------|
| **BM Name** | PPR Champion |
| **EN Name** | Community Housing Champion |
| **Unlock** | Place in Top 3 of any Rumah Pangsa league for a given month |
| **Visual** | PPR building silhouette + gold star |
| **KL Grounding** | This badge is exclusively available to Rumah Pangsa league members. It provides recognition within the B40 community, where energy savings are proportionally more impactful. A resident saving 15 kWh at PPR Lembah Subang (15% of their monthly bill) is doing more, proportionally, than a resident saving 40 kWh at Mont Kiara (5% of their bill). The badge acknowledges this reality |
| **Inclusivity** | This badge CANNOT be earned by Condo Cup or Taman League members. It is exclusive to Rumah Pangsa. This exclusivity creates pride within the community rather than comparison across communities |

#### 12. Dato' Jimat 👑
| Attribute | Detail |
|-----------|--------|
| **BM Name** | Dato' Jimat |
| **EN Name** | Sir Saves-a-Lot |
| **Unlock** | Reach Level 7 (25,000 XP) |
| **Visual** | Songkok (traditional Malay cap, associated with Dato' investiture) + green diamond |
| **KL Grounding** | "Dato'" is Malaysia's most recognized honorific title, awarded by state rulers (Sultans/Yang di-Pertua) for significant contributions. Pairing it with "Jimat" (frugal/save) is simultaneously prestigious and humorous. It says: "You have achieved the highest honor... in saving electricity." The self-deprecating humor makes it deeply Malaysian |
| **Rarity** | Projected: ~18 months of consistent engagement to reach. This badge should be rare. When a user in TTDI reaches Dato' Jimat, it should be a community event — their name shows in the weekly digest with a special callout |
| **Visual distinction** | The Dato' Jimat badge icon appears as a small crown/songkok symbol next to the user's name on the leaderboard, profile, and community digest. It is the only badge that gives persistent visual distinction across the entire app |

---

## 7. Festival Challenge Calendar — The Malaysian Year

### Design Rationale

Generic challenge calendars use Western months ("New Year Resolution January", "Summer Savings July"). These mean nothing in Malaysia, where Christmas is not the defining December event, where "summer" doesn't exist (it's always summer), and where the calendar is shaped by festivals, monsoons, and haze season.

### The Annual Cycle

```
        🌧️  MONSOON           🌤️  DRY               ☀️  HOT/HAZE           🌧️  MONSOON
    Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  Dec
     │    │    │    │    │    │    │    │    │    │    │    │
    CNY  CNY  Ram  Ray  Cut  Gaw  Haz  Mer  HMy  Dee  Mon  Tut
    Spr  GM   dan  a    i    ai   e    dek  ari  pav  soo  up
    Cln       Nur  Bal  Sek  Kaa  Shi  a    Uni  ali  n    Tah
                    ik   ola  mat  eld  55   ty   Lig  Wat  un
                    Kam  h    an                           ch
                    pun                                     
                    g                                       
```

### Challenge Details

#### January: CNY Spring Clean 🧹
| Attribute | Detail |
|-----------|--------|
| **Theme** | Pre-Chinese New Year energy audit |
| **Duration** | January 1–20 (before reunion dinner week) |
| **Mechanic** | Users earn double XP for each always-on device they identify and whitelist. "Spring cleaning" your energy setup |
| **Cultural tie** | CNY preparation involves deep cleaning the house. Extending this ritual to energy — identifying vampire power, optimizing schedules — feels natural |
| **Reward** | "CNY Spring Clean" completion badge |

#### February: Chap Goh Meh Savings 🏮
| Attribute | Detail |
|-----------|--------|
| **Theme** | Return to normal after CNY celebrations |
| **Duration** | February 1–15 (15th day of CNY) |
| **Mechanic** | Return to baseline after the CNY spike. Users who keep post-CNY consumption at or below pre-CNY baseline earn 50% bonus XP |
| **Cultural tie** | Chap Goh Meh (15th day of CNY) marks the end of celebrations. Returning to normal rhythms after the festive spike |

#### March–April: Ramadan Nur 🌙
| Attribute | Detail |
|-----------|--------|
| **Theme** | Optimized sahur/iftar scheduling |
| **Duration** | Entire Ramadan month (29–30 days, lunar calendar) |
| **Mechanic** | System auto-adjusts schedules for sahur (pre-dawn) and iftar (sunset) timings. Off-peak crediting during altered hours. Users who maintain 30-day streak during Ramadan earn "Anak Bulan Hero" badge |
| **Cultural tie** | Ramadan transforms daily energy patterns. The system adapts automatically — users don't need to reconfigure schedules |

#### April–May: Raya Balik Kampung 🚗
| Attribute | Detail |
|-----------|--------|
| **Theme** | Full auto-shutdown during Hari Raya exodus |
| **Duration** | Raya week (varies by year) |
| **Mechanic** | "Balik Kampung Mode" — one-tap activation. All non-whitelisted plugs auto-off. System monitors zero-consumption days. Badge awarded on return |
| **Cultural tie** | The KL exodus is the largest annual energy-saving event in Malaysia — millions of empty homes. Making the savings visible and rewarding feels natural |

#### May: Cuti-Cuti Sekolah 🎒
| Attribute | Detail |
|-----------|--------|
| **Theme** | School holiday family energy challenge |
| **Duration** | May–June school holidays |
| **Mechanic** | Family challenge: households with kids home from school must manage increased daytime energy use. Kid-friendly UI: "Tolong mak ayah jimat tenaga!" (Help mom and dad save energy!) with simple child-accessible toggles |
| **Cultural tie** | School holidays mean kids at home, AC running all day. Turning this into a family game reduces stress and consumption |

#### June: Gawai-Kaamatan 🌾
| Attribute | Detail |
|-----------|--------|
| **Theme** | East Malaysia harvest festival savings |
| **Duration** | June 1–30 |
| **Mechanic** | Special recognition for Sabah/Sarawak users. Harvest-themed saving goals: "Tuai hasil jimat anda" (Harvest your savings) |
| **Inclusivity** | Ensures East Malaysian festivals are represented. The app is not KL-centric to the exclusion of Sabah and Sarawak |

#### July: Haze Shield 🛡️
| Attribute | Detail |
|-----------|--------|
| **Theme** | Energy stability during jerebu season |
| **Duration** | July 1–31 (can extend into August/September if haze persists) |
| **Mechanic** | Keep energy within 5% of baseline despite staying home more (haze forces people indoors, increasing AC use). The badge rewards stability, not reduction |
| **KL Grounding** | KL's annual haze crisis (API 150–200+) is a shared ordeal. Everyone stays home, everyone runs AC and air purifiers. Saving energy during haze is genuinely difficult — the badge acknowledges effort over outcome |
| **Behavioral twist** | This is the only challenge where "saving less than usual" is still rewarded. It recognizes that external factors (haze) override user control. This builds trust: the system doesn't punish you for circumstances |

#### August: Merdeka 55 🇲🇾
| Attribute | Detail |
|-----------|--------|
| **Theme** | National energy independence challenge |
| **Duration** | August 1–31 |
| **Mechanic** | Save X kWh for Malaysia's Xth year (55 kWh in 2026, increments each year). National leaderboard aggregating all kawasan |
| **Cultural tie** | Ties energy saving to national pride. "Jimat tenaga untuk Malaysia" (Save energy for Malaysia) |

#### September: Hari Malaysia Unity 🤝
| Attribute | Detail |
|-----------|--------|
| **Theme** | Cross-league donation drive |
| **Duration** | September 1–16 (leading up to Malaysia Day, September 16) |
| **Mechanic** | Condo Cup and Taman League are encouraged to donate to Rumah Pangsa. Double badge XP for all cross-league donations during this period. Unity leaderboard shows total donations flowing to Rumah Pangsa nationwide |
| **Cultural tie** | Hari Malaysia (September 16, 1963) marks the formation of Malaysia — Sabah, Sarawak, and Malaya uniting. Cross-league donations mirror this unity |

#### October–November: Deepavali Lights 🪔
| Attribute | Detail |
|-----------|--------|
| **Theme** | Efficient festive lighting |
| **Duration** | Deepavali week |
| **Mechanic** | Track festive lighting energy separately. Compare oil lamp (traditional diya) vs. electric decorative lights. Bonus XP for using timers on festive lights |
| **Cultural tie** | Deepavali is the festival of lights. Balancing celebration with efficient lighting |

#### November–December: Monsoon Watch 🌧️
| Attribute | Detail |
|-----------|--------|
| **Theme** | Rainy season energy patterns |
| **Duration** | November–December (northeast monsoon) |
| **Mechanic** | Natural savings challenge: cooler weather means less AC needed. System detects lower AC demand and credits the difference as "Monsoon Bonus" |
| **KL Grounding** | KL's monsoon season (November–December) brings daily afternoon downpours. Temperatures drop 3–5°C. AC naturally runs less. This challenge celebrates the natural savings rather than treating it as baseline drift |

#### December: Tutup Tahun 📊
| Attribute | Detail |
|-----------|--------|
| **Theme** | Year-end review + awards |
| **Duration** | December 15–31 |
| **Mechanic** | Year-in-Review infographic generated: total kWh saved, RM saved, CO₂ avoided, badges earned, kawasan rank, donations made, families helped. "Energy Personality" generated based on saving patterns |
| **Cultural tie** | End-of-year reflection is universal. The shareable infographic drives organic user acquisition during the holiday season |

---

## 8. Leaderboard Design — Competition with Dignity

### Core Philosophy

Leaderboards are dangerous. Done wrong, they:
- Shame low performers (who then quit)
- Reward the already-privileged (who don't need more status)
- Create toxic competition (undermining the community spirit)

Our design principles:
1. **Team-based, never individual.** You represent your kawasan, not yourself alone.
2. **Two views, no shame view.** Community (collective savings) and Generosity (most donated). Never "Least Saved."
3. **Per-league separation.** Condo Cup never sees Taman League numbers and vice versa — only their own league.
4. **Donation visibility.** When a Condo Cup household donates to Rumah Pangsa, the donation shows on BOTH league boards — the donor gets recognition, and the recipient community sees that others are contributing.

### Leaderboard Views

#### View 1: Community (Collective Savings)

Shows total kWh saved by each participating kawasan within the user's league. NOT individual rankings.

```
🏢 CONDO CUP — Mont Kiara Division (Julai 2026)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🥇 Arcoris MK (47 unit)    1,240 kWh  🔥 18 hari
🥈 Verve Suites (62 unit)    980 kWh  🔥 12 hari
🥉 Kiara 163 (31 unit)       730 kWh  🔥 9 hari
   Penderma teratas: Aisyah (Arcoris) — 15 kredit
                     ke PPR Kerinchi
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
4. Marc Residences (28 unit)  620 kWh
5. Kiaramas (19 unit)         480 kWh

Kawasan anda (Arcoris MK) di kedudukan #1 🏆
```

#### View 2: Generosity (Most Donated Credits)

Shows total credits donated, NOT saved. This rewards giving behavior over consumption patterns.

```
🏘️ TAMAN LEAGUE — Penderma Teratas (Julai 2026)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#1  Kumar, Bangsar        22 kredit  (12 ke PPR)
#2  Mei Ling, Cheras      16 kredit  (9 ke PPR)
#3  Anda (Aisyah), TTDI   14 kredit  (6 ke PPR)
#4  Raj, Damansara Hts    9 kredit   (5 ke PPR)
#5  Fatimah, PPR Kerinchi 6 kredit   (semua ke PPR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Why Fatimah appears here despite being in Rumah Pangsa:** She has earned surplus credits beyond her baseline and is donating within her own league. This shows the system working: a PPR resident who becomes a donor is the ultimate success metric.

### States

| State | Display |
|-------|---------|
| **No donations yet** | "Tiada derma bulan ini. Jadilah yang pertama dan tuntut tempat #1!" |
| **User is #1** | Green highlighted row with subtle glow pulse |
| **User is unranked** | "Anda di kedudukan #12. Derma 3 kredit untuk naik ke Top 10." |
| **Kawasan won challenge** | Banner at top: "🏆 TTDI menang Julai! 20% multiplier untuk Ogos." |
| **Kawasan lost challenge** | "Damansara menang Julai. Sertai Ogos: Merdeka 55 🇲🇾" |

---

## 9. Rewards & Incentives — Tangible KL Value

### Reward Hierarchy

```
Level 1: Intangible (Badges, Titles, Streaks) → Identity
Level 2: Social (Leaderboard Rank, Impact Cards) → Status
Level 3: Functional (2x Credits, Priority Matching) → Utility
Level 4: Tangible (Touch 'n Go, Grab, TNB rebates) → Real Value
```

Levels 1–3 are built into the app. Level 4 represents partnership opportunities (conceptual for the demo).

### Intangible Rewards (Built-In)

| Reward | Earned How | Psychological Function |
|--------|-----------|----------------------|
| Badge unlock | Meet specific condition | Dopamine hit from unexpected achievement. Collection instinct |
| Level-up animation | Reach XP threshold | Milestone celebration. Confetti triggers positive association |
| Streak counter | Consecutive days below baseline | Loss aversion. Users will actively save to avoid breaking a streak |
| Title upgrade | Level progression | Identity reinforcement. "I am a Wira Hijau" vs. "I use an energy app" |

### Social Rewards (Built-In)

| Reward | Earned How | Social Function |
|--------|-----------|-----------------|
| Leaderboard rank | Donate credits, save consistently | Public recognition within kawasan |
| Impact card | Monthly summary | Shareable social proof. "My neighborhood saved 1,200 kWh" |
| Thank-you card | Receive donation | Emotional reward. Seeing the donor's name creates social bond |
| Weekly digest mention | Be top donor for the week | Community recognition. Your name in the digest |

### Functional Rewards (Built-In)

| Reward | Level Required | Function |
|--------|---------------|----------|
| Custom schedules | Level 2 (Celik Tenaga) | Unrestricted scheduling |
| Anomaly alerts | Level 3 (Jimat Cermat) | Access to predictive maintenance |
| Community Grid access | Level 4 (Wira Hijau) | Participation in the marketplace |
| 2x credit earning | Level 5 (Pendekar Tenaga) | Doubled credits per kWh saved |
| Custom badge display | Level 6 (Jaguh Komuniti) | Choose which badges show on profile |
| Priority donation matching | Level 7 (Dato' Jimat) | Your donations are matched to recipients first |

### Tangible Rewards (Conceptual, For Demo)

These require external partnerships and are shown as mockups in the prototype:

| Reward | Earned By | Partner | Demo Implementation |
|--------|----------|---------|-------------------|
| Touch 'n Go eWallet RM 5 | 30-day streak | TNG Digital | Show "eWallet credit added" toast notification |
| GrabFood RM 10 voucher | Donate 20 credits | Grab Malaysia | Show voucher code in notification card |
| TNB bill reduction display | Always visible | TNB (data partnership) | Live RM comparison to TNB lifeline band: "Anda jimat RM 18.30 — setara 84 kWh" |
| Setel RM 5 fuel | Monthly challenge winner | Petronas Dagangan | Show QR redemption screen |
| Pasar Malam credit | 14-day streak + walk to market | Local council (conceptual) | "Anda ke Pasar Malam TTDI — jimat RM 3 Grab + RM 0.80 tenaga rumah" |

---

## 10. Social Features — Sharing & Virality

### Impact Card Generator

The primary organic growth mechanism. One-tap export of personal energy impact as a shareable infographic designed for Instagram Stories (9:16) and WhatsApp Status.

**Card content template:**
```
┌─────────────────────────┐
│  IMPAK TENAGA SAYA       │
│                          │
│  👤 Aisyah, TTDI        │
│  🏅 Celik Tenaga (Lv.3) │
│                          │
│  📊 Julai 2026           │
│  ━━━━━━━━━━━━━━━━━━━━━  │
│  ⚡ 84 kWh dijimatkan    │
│  💰 RM 18.30 bil elektrik│
│  🛡️ 6 kredit didermakan │
│  🌳 47 kg CO₂ dielakkan │
│                          │
│  🏆 #3 Penderma TTDI    │
│  🔥 Rantaian 7-Hari     │
│                          │
│  Lencana: 🛡️ 🍃 🌙      │
│                          │
│  Kawasan TTDI jimat      │
│  1,240 kWh bulan ini.    │
│                          │
│  Jiran saya dah jimat.   │
│  Anda bila lagi?         │
│  🇲🇾 @[appname]          │
└─────────────────────────┘
```

**Sharing copy (pre-filled):**
- **WhatsApp:** "🏘️ Saya dah jimat 84 kWh bulan ni dengan [app name]! Kawasan TTDI jimat 1,240 kWh — setara 62 unit PPR sehari. Jom join — makin ramai, makin banyak kita boleh bantu PPR sekitar KL. 🇲🇾"
- **Instagram:** Same + hashtags: #JimatTenaga #KomunitiKL #TTDI

### Donation Thank-You Card

When a Rumah Pangsa household receives donations, they receive a thank-you card showing the donors' display names and impact amounts. This encourages:
- **Reciprocity:** "3 jiran bantu saya bulan ni. Bila saya ada lebihan, saya akan derma balik."
- **Visibility:** Donations are not anonymous — they carry social weight
- **Dignity:** The card uses respectful language: "Terima kasih kepada Aisyah (Arcoris MK) — 5 kredit didermakan." Not "Aisyah paid your bill."

### Weekly Community Digest

Sent every Monday at 9 AM (push notification + in-app):

```
📊 Ringkasan Mingguan — TTDI

━━━━━━━━━━━━━━━━━━━━━━━━━
Minggu ini, TTDI jimat:
⚡ 1,200 kWh

Setara:
🏠 58 unit PPR untuk sehari
🚦 40 lampu Jln Bukit Bintang

━━━━━━━━━━━━━━━━━━━━━━━━━
🏆 Penderma Teratas Minggu Ini:
#1 Raj — 12 kredit (8 ke PPR Kerinchi)
#2 Anda (Aisyah) — 8 kredit (5 ke PPR Lembah Subang)

━━━━━━━━━━━━━━━━━━━━━━━━━
⚡ Cabaran Haze Shield:
TTDI 72% | Damansara 58%
4 hari lagi!

🔥 Rantaian anda: 7 hari. Teruskan!
```

---

## 11. UX/UI Integration — Widgets & Screens

### Screen Map

```
/dashboard           ← Streak indicator (compact, bottom of Summary Card)
    │
/community           ← StreakBadgeRow (top)
    │                  NeighborhoodChallengeCard (middle)
    │                  Leaderboard (middle)
    │                  Impact Card (bottom)
    │
    ├─ DonationModal   ← Slide-up sheet, BM copy
    │
    └─ /badges         ← Push, back arrow to /community
         │               Full badge grid + Level/XP card
         │
/settings             ← Profile: Level/XP progress bar
    │
    └─ /badges         ← Push (same screen as above)
```

### Widget Specifications

#### StreakBadgeRow
| Attribute | Spec |
|-----------|------|
| **Component name** | `StreakBadgeRow` |
| **Location** | `/community`, below App Bar, above Credits Card |
| **Height** | 44dp |
| **Layout** | Horizontal scrollable row of compact chips |
| **Chip width** | Variable: streak chip 48dp, badge chips 56–72dp depending on name length |
| **Spacing** | 8dp between chips |
| **Background** | `--color-surface` (#1E1E1E) with 1dp `--color-divider` (#2C2C2C) border |
| **Streak chip** | Always leftmost. Fire 🔥 emoji (16sp) + count number (14sp, `--color-text-primary`) + "Hari" label (12sp, `--color-text-secondary`). Pulsing opacity animation (1.2s loop, 100% → 70% → 100%) when streak is active |
| **Badge chip** | Emoji (16sp) + Malay name (12sp, `--color-text-secondary`, truncated to 12 chars). Earned badges: `--color-text-primary` text. Locked badges not shown in this row |
| **Chevron** | `›` character, 20sp, `--color-text-tertiary`. Taps to `/badges` |
| **Empty state** | Single grey chip: "🔒 4 lencana untuk dibuka" (40% opacity, 12sp `--color-text-tertiary`) |
| **Streak broken** | Chip greyed (40% opacity). Subtle horizontal shake animation (200ms, 2 oscillations, 4dp amplitude) on first view of broken streak |

#### NeighborhoodChallengeCard
| Attribute | Spec |
|-----------|------|
| **Component name** | `NeighborhoodChallengeCard` |
| **Location** | `/community`, between leaderboard and impact card |
| **Height** | Collapsed: 120dp. Expanded: 200dp |
| **Animation** | Expand/collapse: 300ms `easeInOut` |
| **Background** | `--color-surface` (#1E1E1E) with `--color-primary` (#00C853) left border (3dp) |
| **Header** | "⚡ CABARAN: {name} ({month})" — 14sp, `--color-text-primary` |
| **Progress bar** | 2 horizontal bars, 8dp height. Filled: `--color-primary`. Unfilled: `--color-divider`. Bar width proportional to percentage. Kawasan name (14sp) + home count (12sp, `--color-text-secondary`) + percentage (16sp bold, `--color-primary`) |
| **Status text** | "{count} hari lagi · {leadingKawasan} mendahului!" — 12sp, `--color-text-secondary` |
| **Expanded extras** | kWh numbers (e.g., "864/1200 kWh"), top donor per kawasan, streak days, countdown clock, share button |
| **Won state** | "🏆 {kawasan} MENANG! 20% multiplier untuk {nextMonth}. Kongsi kejayaan anda →" Green background tint |
| **Lost state** | "{winningKawasan} menang {month}. Sertai cabaran {nextMonth}: {nextChallenge} 🇲🇾" Neutral |
| **Hidden state** | Widget hidden when no active challenge exists |
| **Data model** | `challengeName`, `month`, `neighborhoodA` (name + homeCount + progress + topDonor + streak), `neighborhoodB` (same), `daysRemaining`, `leadingNeighborhood` |

#### Level/XP Progress Bar (Settings Profile)
| Attribute | Spec |
|-----------|------|
| **Location** | `/settings`, inside Profile card, between address and Edit button |
| **Layout** | Emoji icon + level title + progress bar + XP numbers + next level hint |
| **Format** | "🏅 Celik Tenaga (Tahap 3)" (14sp, `--color-text-primary`) → progress bar → "1,840 / 3,000 XP" (12sp, `--color-text-secondary`) → "560 XP lagi ke Wira Hijau ›" (11sp, `--color-text-tertiary`) |
| **Progress bar** | 6dp height, full width. Filled: `--color-primary`. Unfilled: `--color-background-alt` (#181818). Rounded corners (3dp radius) |
| **Tap target** | Entire row tappable → push `/badges` |
| **Animation** | Progress bar fills with 600ms `easeOut` animation when screen loads |

#### Badge Collection Screen (`/badges`)
| Attribute | Spec |
|-----------|------|
| **Route** | `/badges` |
| **App bar** | "← Koleksi Lencana" — 18sp, `--color-text-primary`. Back arrow returns to previous screen |
| **Level card** | Top of screen. Same as Settings Profile progress bar, but larger (48dp height) |
| **Earned section** | "Diperolehi ({count})" header (14sp, `--color-text-secondary`). 3-column grid. Each badge card: 80dp × 100dp. Emoji (28sp, centered). Name below (12sp, `--color-text-primary`, centered, max 2 lines). Unlock date below (10sp, `--color-text-tertiary`) |
| **Locked section** | "Terkunci ({count})" header. Same grid layout but 40% opacity overall. Unlock condition text (10sp, `--color-text-tertiary`) below name instead of date |
| **Grid spacing** | 12dp between cards, 16dp row gap |
| **Empty earned** | Centered text: "Belum ada lencana. Mulakan perjalanan jimat tenaga anda!" (14sp, `--color-text-secondary`) |
| **New badge toast** | Bottom toast: "🎉 Lencana baru: {name}! Ketik untuk lihat." Slides up from bottom, 3s display, auto-dismiss. Tapping navigates to `/badges` with new badge pulsing glow animation (2s loop, opacity 100% → 70% → 100%) |

---

## 12. Behavioral Psychology — Why This Works

### The Hook Model Applied

Nir Eyal's Hook Model (Trigger → Action → Variable Reward → Investment) maps directly to our gamification:

| Hook Phase | Our Implementation |
|------------|-------------------|
| **External Trigger** | Push notification: "Jangan patah rantaian 7-hari anda! 🔥" at 8 PM. Community digest on Monday 9 AM |
| **Internal Trigger** | Boredom at kopitiam → check leaderboard. Guilt after high-bill month → set goal |
| **Action** | Save energy (automatic). Donate credits (3 taps). Check leaderboard (1 tap). Share impact card (2 taps) |
| **Variable Reward** | Badge unlock (unpredictable which and when). Leaderboard rank change (unpredictable). Donation thank-you card (unpredictable sender) |
| **Investment** | Streak to protect. Badge collection to complete. Level progress to maintain. Community reputation to build |

### Loss Aversion (Streak Mechanic)

The #1 retention mechanic in the system is the streak counter. Behavioral economics shows that losses loom larger than gains (Kahneman & Tversky, 1979). A user with a 6-day streak will actively manage energy on day 7 because **losing the streak feels worse** than the effort of saving.

**Implementation nuance:** The nudge at 8 PM ("Jangan patah rantaian 7-hari anda!") is the ONLY push notification that cannot be fully disabled (it can be snoozed for 24 hours). All other notifications are opt-in.

### Social Proof (Leaderboard + Impact Cards)

Robert Cialdini's principle of social proof: people do what they see others doing. When a user sees that Kumar in Bangsar donated 22 credits, and Mei Ling in Cheras donated 16, they think: "These are my neighbors. They're donating. I should too."

The impact card extends this to non-users: "My neighborhood saved 1,200 kWh" is social proof to friends on WhatsApp that this behavior is normal and valued.

### Endowed Progress (XP Bar)

The "endowed progress effect" (Nunes & Dreze, 2006) shows that people are more likely to complete a goal if they feel they've already made progress. The XP bar starts at 0 but fills quickly in the early levels (Level 1 → 2 requires only 500 XP, achievable in ~2 weeks). This creates a sense of momentum that carries users into the longer mid-game.

### Self-Determination Theory

Deci & Ryan's three psychological needs:

| Need | How We Address It |
|------|-------------------|
| **Autonomy** | Everything is opt-in. The user chooses their sharing tier, their donation amount, their challenge participation. No mandatory gamification |
| **Competence** | Level system provides clear progression. Badges provide concrete evidence of skill. "Celik Tenaga" = you have mastered energy literacy |
| **Relatedness** | Community Grid is built entirely on social connection. Kawasan vs. kawasan challenges. Donation thank-you cards. Impact cards shared to WhatsApp. The user is never saving alone |

### The IKEA Effect

People value things they've invested effort in more highly. A badge earned over 30 days of Ramadan saving has more psychological value than a badge given for installing the app. A Level 7 "Dato' Jimat" earned over 18 months becomes part of the user's identity.

---

## 13. Demo Implementation Guide

### Scope

The demo prototype shows a fully functional Community Grid tab with all gamification widgets, using mock data. No backend required — all data is hardcoded Dart models.

### What to Build (Priority Order)

#### Priority 1 — Community Grid Tab (Must Have)

| Element | Description | Hardcoded Data |
|---------|-------------|----------------|
| **StreakBadgeRow** | Horizontal scrollable chips at top | User has 7-day streak + 3 badges (Jiran Terbaik, Anak Bulan Hero, Celik Tenaga) |
| **Credits Card** | Shows "14 kredit tersedia" with "Derma Kredit" button | 14 credits |
| **Leaderboard** | 5 entries with KL persona names | Kumar/Bangsar (22 cr), Mei Ling/Cheras (16 cr), Aisyah/TTDI (14 cr, highlighted), Raj/Damansara (9 cr), Fatimah/PPR Kerinchi (6 cr) |
| **NeighborhoodChallengeCard** | Collapsed Haze Shield challenge | TTDI 72% vs Damansara 58%, 4 days left |
| **Impact Card** | KL-grounded stats | 1,240 kWh = 62 PPR units, 11 AC, 58 Jln Bukit Bintang lamps, 84 credits shared |
| **Donation Modal** | Slide-up sheet with PPR recipients | PPR Kerinchi (3 families waiting), PPR Lembah Subang (2 families) |

#### Priority 2 — Badge Collection (Should Have)

| Element | Description | Hardcoded Data |
|---------|-------------|----------------|
| **Level/XP Card** | Full progress bar | "Celik Tenaga (Tahap 3) · 1,840/3,000 XP · 560 XP ke Wira Hijau" |
| **Earned badges grid** | 3-column, 4 badges | 7-Hari Rantaian, Jiran Terbaik, Anak Bulan Hero, Celik Tenaga |
| **Locked badges grid** | 3-column, greyed, 8 badges | All 8 with unlock conditions |
| **Back navigation** | From /badges to /community or /settings | |

#### Priority 3 — Settings Profile (Nice to Have)

| Element | Description | Hardcoded Data |
|---------|-------------|----------------|
| **Level/XP row** | Inside Profile card | Same data as Badge Collection Level/XP card but compact |
| **Tap target** | Opens /badges | |

### Mock Data Classes (Dart)

```dart
// Badge model
class Badge {
  final String id;
  final String nameBM;
  final String nameEN;
  final String emoji;
  final String unlockCondition;
  final bool isUnlocked;
  final DateTime? unlockedDate;
  
  const Badge({
    required this.id,
    required this.nameBM,
    required this.nameEN,
    required this.emoji,
    required this.unlockCondition,
    required this.isUnlocked,
    this.unlockedDate,
  });
}

// Level model
class UserLevel {
  final int level;
  final String titleBM;
  final String titleEN;
  final int currentXP;
  final int requiredXP;
  final int xpToNext;
  final String nextTitleBM;
  final String nextTitleEN;
  
  double get progress => currentXP / requiredXP;
}

// Leaderboard entry model
class LeaderboardEntry {
  final int rank;
  final String name;
  final String kawasan;
  final String league; // 'condo', 'taman', 'ppr'
  final int creditsDonated;
  final int creditsToPPR;
  final bool isCurrentUser;
}

// Challenge model
class NeighborhoodChallenge {
  final String name;
  final String month;
  final ChallengeKawasan kawasanA;
  final ChallengeKawasan kawasanB;
  final int daysRemaining;
  final String leadingKawasan;
}

class ChallengeKawasan {
  final String name;
  final int homes;
  final double progress; // 0.0 - 1.0
  final String topDonor;
  final int streak;
}

// Pre-built mock data for demo
final mockBadges = {
  'earned': [/* 4 badges */],
  'locked': [/* 8 badges */],
};

final mockUserLevel = UserLevel(
  level: 3,
  titleBM: 'Celik Tenaga',
  titleEN: 'Energy Literate',
  currentXP: 1840,
  requiredXP: 3000,
  xpToNext: 560,
  nextTitleBM: 'Wira Hijau',
  nextTitleEN: 'Eco Champion',
);

final mockLeaderboard = [/* 5 entries from persona data */];

final mockChallenge = NeighborhoodChallenge(/* Haze Shield, TTDI vs Damansara */);
```

### UI Strings Used in Demo (BM Primary)

| String | Usage |
|--------|-------|
| "Komuniti" | Tab label, App Bar title |
| "Kredit Tenaga Anda" | Credits Card header |
| "14 kredit tersedia" | Credits Card body |
| "Derma Kredit" | Button text |
| "🏆 Penderma Teratas — Julai" | Leaderboard header |
| "kpd {count} jiran — {ppr}" | Leaderboard donation detail |
| "Anda" | Current user label on leaderboard |
| "⚡ CABARAN: Haze Shield (Julai)" | Challenge header |
| "4 hari lagi · {kawasan} mendahului!" | Challenge status |
| "🌏 Impak Komuniti" | Impact Card header |
| "Bulan ini, kawasan KL kita jimat:" | Impact Card intro |
| "84 kredit dikongsi ke 4 kawasan" | Impact Card footer |
| "📤 Kongsi" | Share button |
| "Pilih penerima" | Donation modal label |
| "🏠 Tabung Komuniti" | Community Pool option |
| "Diagih ke PPR sekitar KL secara automatik" | Pool description |
| "3 keluarga sedang tunggu" | PPR waiting count |
| "Jumlah" | Amount label |
| "~RM 1.09 bil elektrik" | Impact preview |
| "Derma ke PPR = 2x XP lencana" | Multiplier hint |
| "Sahkan Derma" | Confirm button |
| "5 kredit didermakan!" | Success title |
| "Anda bantu ringankan ~RM 1.09 untuk PPR Kerinchi." | Success body |
| "🛡️ 3 lagi untuk lencana Jiran Terbaik!" | Success badge hint |
| "Koleksi Lencana" | Badge screen title |
| "Diperolehi (4)" | Earned section header |
| "Terkunci (8)" | Locked section header |
| "Belum ada lencana." | Empty state (not shown in demo) |
| "Profil" | Settings title |
| "Plugs Saya" | Plug section header |
| "Sentiasa-On" | Always-on section header |
| "Notifikasi" | Notifications header |

---

## 14. Future Expansions

### Phase 2: Gamification (Post-Demo)

| Feature | Description | Timeline |
|---------|-------------|----------|
| **Live Streak Animation** | Real-time streak counter updates on dashboard with micro-celebrations (confetti burst at 7, 30, 100 days) | Sprint 3 |
| **Challenge Notifications** | Push when kawasan falls behind in monthly challenge: "TTDI ketinggalan 120 kWh! Buka app untuk lihat." | Sprint 4 |
| **Badge Animation Polish** | Unique animation per badge unlock (Anak Bulan Hero = moon rise, Jiran Terbaik = hands shake, Merdeka Saver = flag wave) | Sprint 5 |
| **Friend Challenges** | Direct challenge between two households: "Aisyah cabar Kumar: siapa jimat lebih banyak minggu ni?" | Sprint 6 |

### Phase 3: Community (Year 2)

| Feature | Description |
|---------|-------------|
| **Kawasan Chat** | Optional chat for league members (moderated, energy-topic only) |
| **Community Events** | "TTDI Energy Day" — real-world meetup at local park, app-organized |
| **PPR Ambassador Program** | Rumah Pangsa residents become community ambassadors, earning credits for onboarding neighbors |
| **School Integration** | "Sekolah Hijau" program — schools compete in energy challenges, students bring app home to families |

### Phase 4: National Scale (Year 3)

| Feature | Description |
|---------|-------------|
| **State Leagues** | Selangor vs KL vs Penang vs Johor annual energy challenge |
| **TNB Integration** | Direct TNB bill data import for hybrid metering (smart plug + utility meter combined) |
| **Carbon Credit Marketplace** | Verified CO₂ savings bundled and sold to corporate offset buyers. Revenue returned to users as bill credits |
| **Malaysia Energy Day** | Annual national event: "1 juta rakyat Malaysia jimat 1 kWh hari ini" |

---

*Document prepared for UM Technothon 2026 — Smart Energy Management for a Sustainable Future*  
*Gamification design grounded in Kuala Lumpur, Malaysia*
