# Presentation Deck Plan — 7 Minutes

---

## Slide 1 — Cover (5 sec)

**Layout:**
- Full dark background (#121212)
- Large top: "Smart Energy. Smarter Homes."
- Subtitle: "Smart Plug Power Management Ecosystem — UM Technothon 2026"
- Bottom-left: Team name
- Bottom-right: UM Technothon 2026 badge

**Speaker notes:**
> *"Good morning. We're here to fix the way Malaysians use energy at home."*

---

## Slide 2 — The Problem (40 sec)

**Header:** Malaysian households waste 22% of electricity every year.

**3 pain points (icon + text):**

| Icon | Text |
|------|------|
| 🏠 | **Appliances run unattended** — ACs, fans, TVs left on when nobody's home |
| 🔌 | **Vampire power drains 5-10%** — Standby consumption from chargers, microwaves, game consoles |
| 💸 | **Bill shock at month-end** — Users see the damage only when it's too late |

**Footer:** Source citation (TNB / Malaysia Energy Information Hub) in small text.

**Speaker notes:**
> *"Massive amounts of electricity are wasted daily. Lights, AC, appliances running unnecessarily. Existing approaches aren't working — because they weren't designed for real people. They were designed for engineers."*

---

## Slide 3 — What's Missing (25 sec)

**Two-column layout:**

| ❌ Current Solutions | ✅ What We Need |
|---------------------|----------------|
| Smart meters — limited pilots, no user action | **Zero-thought automation** — the system acts, not just shows |
| Basic smart plugs — manual toggle only | **AI that anticipates** — learns patterns, predicts failures |
| Energy apps — graphs you ignore after day 1 | **Community incentive** — saving helps others, not just yourself |

**Bottom bold statement:**
> "The problem isn't technology. It's that nobody designed these tools for real people."

**Speaker notes:**
> *"Existing tools show you the problem but expect you to fix it yourself. We think technology should do the work — silently, invisibly, effectively."*

---

## Slide 4 — Our Solution (30 sec)

**3 pillars, each with icon + 1-liner:**

| Pillar | Description |
|--------|-------------|
| 🔌 **Smart Plugs** | Pre-flashed Sonoff S31 · Zero-config BLE tap-to-pair in 3 seconds · Real-time MQTT energy metering |
| 🧠 **AI Intelligence** | Auto-detects plugged appliances (NILM) · Predicts failures before they happen · Learns each user's notification tolerance |
| 👥 **Community Grid** | Turn saved kWh into energy credits · Donate to low-income neighbors · Leaderboard that rewards generosity, not wealth |

**Bottom anchor statement:**
> "We don't show you your bill. We fix it."

**Speaker notes:**
> *"Three layers. Hardware that just works. AI that learns when to act and when to stay silent. And a community model that turns personal savings into collective good."*

---

## Slide 5 — [APP PROTOTYPE] Dashboard Demo (60 sec)

**PLACEHOLDER:** Insert recorded screen capture of the Flutter app or high-fidelity Figma prototype.

**Show 3 moments in sequence:**

| Moment | Duration | Key Visual |
|--------|----------|------------|
| **Dashboard overview** | 20 sec | 6+ DeviceCards across 3 rooms (Living Room, Kitchen, Bedroom). Live wattage on every card. Summary card showing "420 W · RM 1.37 today" |
| **Leaving Home** | 20 sec | Tap "🚪 Leaving Home" chip → all non-essential plugs turn off simultaneously → Toast: "Turned off 4 devices. Fridge still running." |
| **Live update** | 20 sec | Pull-to-refresh → one plug comes back online → DeviceCard animates from grey (offline) to green (online) → wattage updates |

**Caption on slide:**
> "Setup: 3 seconds. Control: 1 tap. Zero learning curve."

**Speaker notes:**
> *"This is what a home actually looks like — multiple rooms, multiple devices, all running simultaneously. One tap turns everything off except the essentials. No hunting for switches. No thinking about what to leave on."*

---

## Slide 6 — [APP PROTOTYPE] AI Features Demo (40 sec)

**PLACEHOLDER:** Insert app recording showing AI Insights tab.

**2-part demo:**

| Feature | Duration | Visual |
|---------|----------|--------|
| **NILM — Appliance Fingerprinting** | 20 sec | User plugs in a fridge. App auto-detects "🧊 Fridge" without any manual labeling. Caption: "The AI reads the electrical signature. No setup required." |
| **Anomaly Detection** | 20 sec | Push notification appears: "Your fridge consumed 35% more than usual. Estimated impact: RM 18/month." User taps → Plug Detail screen opens with red PowerGauge and a spike visible on the trend chart. |

**Caption on slide:**
> "The AI spots problems before you feel them."

**Speaker notes:**
> *"Two AI features that work invisibly. The system auto-identifies every appliance — no manual labeling. And when something's wrong, it tells you before the repair bill arrives. Not after."*

---

## Slide 7 — [APP PROTOTYPE] AI Chat Demo (30 sec)

**PLACEHOLDER:** Insert recorded screen capture showing the chat interface.

**Show 3 interactions in quick succession:**

| Interaction | Duration | Visual |
|-------------|----------|--------|
| **Natural language control** | 10 sec | User types "Turn off the living room AC" → Chat responds: "Done ✓ — Living Room AC turned off. Saving ~RM 0.26/hour." Relay toggles on the dashboard in real-time. |
| **Diagnostic query** | 10 sec | User taps quick-chip: "What's using the most power?" → Chat: "Your AC Bedroom at 1.2 kW (78% of total). Next: Rice Cooker at 160W." |
| **Proactive summary** | 10 sec | Chat auto-sends morning summary: "Good morning. You saved RM 1.80 yesterday. No anomalies detected. Today's forecast: hot — AC pre-cooling scheduled at 5 AM (off-peak). [Adjust Schedule]" |

**UI notes for the prototype:**
- Floating action button (chat bubble icon) visible on the Dashboard — tap to open
- Chat sheet slides up from bottom (60% screen height)
- User bubbles right-aligned in `--color-surface`, assistant bubbles left-aligned with `--color-primary` tint
- Quick-action chips row at bottom: "Turn everything off", "What's using the most?", "Show anomalies", "Bill forecast"
- Typing indicator (animated dots) when assistant is "thinking" — 1 second simulated delay

**Caption on slide:**
> "Talk to your home like you talk to a person."

**Speaker notes:**
> *"Dashboards are great — when you want to stare at graphs. But sometimes you just want to ask a question. Turn off the AC. What's wrong with my fridge? How much did I save today? The chat understands your energy and acts on it. This is the easiest interface we've ever designed — because you already know how to use it."*

---

## Slide 8 — Community Grid Concept (40 sec)

**Visual:** KL neighborhood map — TTDI, Damansara Heights, PPR Kerinchi connected by dotted energy lines. Arrows flow from greener Condo Cup/Taman League homes to Rumah Pangsa households.

**3 steps:**

1. **Every home gets a dynamic baseline** — Rolling 14-day prediction. Weather-adjusted. Changes with your habits.
2. **Save below baseline → earn credits** — 1 credit = 1 kWh saved. Max 15 credits/month (prevents solar-rich homes from dominating).
3. **Donate to neighbors** — One tap sends credits to low-income PPR households or the community pool.

**3 league tiers (KL-ground trinity):**
| League | Housing | Example | Inclusivity |
|--------|---------|---------|-------------|
| 🏢 Condo Cup | High-rise | Mont Kiara, Bangsar South | Standard baseline |
| 🏘️ Taman League | Landed homes | TTDI, Damansara Heights | +15% home size adjustment |
| 🏠 Rumah Pangsa | PPR flats | PPR Kerinchi, PPR Lembah Subang | TNB lifeline band baseline |

**Bottom line:**
> "TTDI saves more → PPR Kerinchi pays less. It's gotong-royong for energy."

**Speaker notes:**
> *"Most energy projects only help people who can already afford solar panels. Community Grid is different — it creates a tangible incentive to save, and routes that benefit directly to the households that need it most. We separate the leagues so a Mont Kiara penthouse doesn't compete with a PPR flat. But cross-league donation carries double the impact."*

---

## Slide 9 — [APP PROTOTYPE] Community Grid Demo (30 sec)

**PLACEHOLDER:** Insert app recording of Community Grid tab.

**Sequence:**

| Step | Visual |
|------|--------|
| **Komuniti tab** | Streak + Badge row: "🔥 7-Hari 🔥 🛡️ Jiran Terbaik 🍃 Celik Tenaga" · Credits card: "14 kredit tersedia" · "Derma Kredit" button |
| **Donation modal** | Pick recipient → "PPR Kerinchi (3 keluarga sedang tunggu)" pre-selected. Amount slider → drag to 5 kredit. Impact preview: "~RM 1.09 bil elektrik · Derma ke PPR = 2x XP lencana" |
| **Confirmation** | Tap "Sahkan Derma" → confetti → "5 kredit didermakan! Anda bantu ringankan ~RM 1.09 untuk PPR Kerinchi. 🛡️ 3 lagi untuk lencana Jiran Terbaik!" |
| **Impact card** | KL-grounded stats: "Bulan ini, kawasan KL kita jimat 1,240 kWh = 62 unit PPR sehari. 84 kredit dikongsi ke 4 kawasan." Kongsi button visible. |
| **Neighborhood Challenge** | Live Haze Shield challenge: TTDI (72%) vs Damansara (58%). "4 hari lagi · TTDI mendahului!" |

**Caption on slide:**
> "Jiran tolong jiran. Energy credits flow from surplus homes to PPR families."

**Speaker notes:**
> *"Everything is in BM-English mix. The leaderboard shows real KL kawasan names — Kumar in Bangsar, Mei Ling in Cheras. When you donate, the app tells you exactly which PPR your credits go to. That transparency builds trust."*

---

## Slide 9A — Gamification: Badges, Levels & Streaks (25 sec)

**Visual:** 3-column grid showing 6 most visually distinct badges with Malay names.

| 🌙 Anak Bulan | 🛡️ Jiran Terbaik | 🇲🇾 Merdeka Saver |
|--------------|-----------------|-----------------|
| Ramadan 30-day streak | 15 credits to PPR | 55 kWh in August |
| 🍃 Celik Tenaga | 🌱 Taugeh Champion | 👑 Dato' Jimat |
| Reach Level 3 | Save RM 50+/month | Level 7 — 25,000 XP |

**Level progression bar below badges:**
```
Budak Baru → Celik Tenaga → Jimat Cermat → Wira Hijau → Pendekar Tenaga → Jaguh Komuniti → Dato' Jimat
    1             2              3             4              5                6              7
```

**Bullet points:**
- 12 localized badges tied to KL culture (Ramadan, Balik Kampung, Kopitiam, Mamak, Haze)
- 7-level XP system with BM titles ("Jimat Cermat" = classic Malaysian frugality)
- Festival-linked challenges follow the Malaysian calendar (not generic months)
- Cross-league donations to Rumah Pangsa earn 2x badge XP

**Bottom line:**
> "Like a fitness app — but for your electric bill. And every badge is grounded in KL life."

**Speaker notes:**
> *"No 'Vampire Hunter' or 'Night Owl' badges here. Our badges are Anak Bulan Hero for Ramadan savers, Kopitiam Regular for people who escape to air-conditioned kopitiams during peak heat, and Taugeh Champion — taugeh means bean sprout, the cheapest food. Malaysian slang for frugal living. Every badge tells a KL story."*

---

## Slide 10 — Architecture + Privacy (35 sec)

**Clean architecture diagram:**

```
┌───────────────────────────────────────────────────┐
│  User Side (encrypted, on-device)                 │
│  ┌──────────┐     ┌───────────────────────────┐   │
│  │ 3× Sonoff│ MQTT│ Flutter App + TimescaleDB │   │
│  │ S31 Plugs│ ───→│ (raw data stays here)     │   │
│  └──────────┘     └──────────┬────────────────┘   │
│                              │                    │
│                 ┌────────────▼────────────┐       │
│                 │ Differential Privacy    │       │
│                 │ (noise added before     │       │
│                 │  upload)                │       │
│                 └────────────┬────────────┘       │
├──────────────────────────────┼────────────────────┤
│  Host Side (aggregated only) │                    │
│                 ┌────────────▼────────────┐       │
│                 │ BigQuery — Aggregated   │       │
│                 │ Data (cohorts of 100+   │       │
│                 │ households, anonymous)  │       │
│                 └─────────────────────────┘       │
└───────────────────────────────────────────────────┘
```

**Two callouts:**
- Left: "Raw energy data never leaves your phone. AI models train on-device."
- Right: "You choose what to share. 3 tiers: Default — Eco — Grid."

**Bottom ethics statement:**
> "Informed consent, not dark patterns."

**Speaker notes:**
> *"We built privacy into the architecture from day one. The raw data — every watt, every second — stays on your phone. What goes to the cloud is anonymized, aggregated, and useless for identifying you. You control what tier you share."*

---

## Slide 11 — Business Model + Impact (30 sec)

**Two-column layout:**

| 💰 Revenue | 🌱 Impact |
|------------|-----------|
| **Hardware:** RM 59/plug (40% margin). RM 169 starter kit (3-pack) | SDG 7 — Affordable & Clean Energy |
| **Freemium App:** RM 9.90–19.90/month. Free tier always available | SDG 11 — Sustainable Communities |
| **B2B Data Licensing:** Regional reports to TNB (RM 15k–60k/year) | SDG 13 — Climate Action |

**Center emphasis line:**
> "Revenue funds mission. Mission creates revenue."

**Speaker notes:**
> *"The business model funds the impact, not the other way around. Hardware gets us in the door. The free tier builds trust. Data licensing pays for the infrastructure. And every revenue stream is built on opt-in consent."*

---

## Slide 12 — Roadmap (25 sec)

**Horizontal timeline, 3 phases:**

| Phase | Timeline | Milestone |
|-------|----------|-----------|
| 🔵 **Now** | Hackathon | 3 Sonoff plugs + Flutter app · NILM trained on UK-DALE public dataset · Pairing + dashboard + AI demo running |
| 🟢 **6 Months** | Pilot | 50 beta users in Klang Valley · Nudge engine collecting persona data · Community Grid pilot in one neighborhood |
| 🟡 **12 Months** | Scale | TNB partnership discussions · 5,000+ household data → statistically meaningful analytics |

**Bottom statement:**
> "Scaling is a software problem. The hardware is already solved."

**Speaker notes:**
> *"We're not building hardware from scratch. The Sonoff S31 is a proven platform. Every month of the roadmap adds users, not new engineering. Scaling is a software problem — and that's where our team's strength is."*

---

## Slide 13 — Thank You (5 sec)

- **Title:** "Smart Energy. Smarter Homes."
- **Team members:** Names listed
- **Links:**
  - ✉️ Email: team@email.com
  - 🐙 GitHub: github.com/team/project
  - 📱 Prototype: link to Figma or app demo
- **QR code** (optional — if you have a project page)

---

## Appendix: Timing Summary

| Slide | Topic | Duration |
|-------|-------|----------|
| 1 | Cover | 5 sec |
| 2 | The Problem | 40 sec |
| 3 | What's Missing | 25 sec |
| 4 | Our Solution | 30 sec |
| 5 | 🔴 APP PROTOTYPE — Dashboard | 60 sec |
| 6 | 🔴 APP PROTOTYPE — AI Features | 40 sec |
| 7 | 🔴 APP PROTOTYPE — AI Chat | 30 sec |
| 8 | Community Grid Concept (KL-Grounded) | 40 sec |
| 9 | 🔴 APP PROTOTYPE — Community Grid Demo | 30 sec |
| 9A | Gamification: Badges, Levels & Streaks | 25 sec |
| 10 | Architecture + Privacy | 35 sec |
| 11 | Business + Impact | 30 sec |
| 12 | Roadmap | 25 sec |
| 13 | Thank You | 5 sec |
| | **Total** | **~7 min 20 sec** |

## Appendix: Rubric Coverage

| Criterion | Weight | Slides That Hit It |
|-----------|--------|---------------------|
| Applicability / Relevance | 10% | 2, 3, 8 |
| Innovation in Implementation | 10% | 4, 6, 7, 9A |
| Sustainability | 10% | 8, 9, 11 |
| Feasibility & Practicability | 10% | 10, 11, 12 |
| Technical Aspects of Functionalities | 20% | 5, 6, 7, 10 |
| Stakeholder Understanding & Ethics | 10% | 7, 8, 10 |
| Urgency & Real-World Relevance | 10% | 2, 3, 8 |
| Clarity of Concept | 10% | 5, 7, 9, 9A |
| Formatting & Presentation Creativity | 10% | 9, 9A — KL-grounded visual treatment throughout |

## Appendix: Prototype Requirements

| Slide | What Needs to Be Recorded | Priority |
|-------|--------------------------|----------|
| 5 | Dashboard with 6+ DeviceCards, live wattage, "Leaving Home" chip + tap + toast | High |
| 6 | AI Insights tab — anomaly card, NILM auto-detection | High |
| 7 | Chat interface — typing an NL command, response, quick-action chips | Medium |
| 9 | Komuniti tab — credits card, StreakBadgeRow (4 badges), leaderboard with KL names, donation modal with confetti, Neighborhood Challenge widget, KL impact card with PPR equivalencies | High |
| 9A | Badge collection screen (`/badges`) — 4 earned + 8 locked badges in 3-column grid, Level/XP progress bar at top | Medium |

If one of these can't be ready in time, prioritize Slides 5 (Dashboard) and 9 (Komuniti) — they establish the core user experience and the KL-grounded gamification.

---

*Plan prepared for UM Technothon 2026 — 13 slides, ~7 minutes*
