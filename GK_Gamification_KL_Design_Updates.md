# KL-Grounded Gamification — Design Update Spec

> Concise reference for implementing the Flutter demo prototype.  
> BM-English bilingual. All data grounded in Kuala Lumpur geography and Malaysian culture.

---

## 1. KL League System

| League | Icon | Housing | KL Examples | Baseline |
|--------|------|---------|-------------|----------|
| Condo Cup | 🏢 | High-rise condos | Mont Kiara, Bangsar South, KLCC, Sri Hartamas, Desa ParkCity | Standard rolling |
| Taman League | 🏘️ | Landed terrace/semi-D | TTDI, Bangsar Park, Damansara Heights, Cheras, Kepong, Setapak | Standard + 15% |
| Rumah Pangsa | 🏠 | Low-cost PPR flats | PPR Lembah Subang, PPR Kerinchi, PPR Seri Alam, PPR Pantai Dalam | TNB lifeline band (200 kWh) |

**Rivalry examples:** Bangsar vs Damansara Heights · Cheras vs Ampang · Arcoris MK vs Verve Suites vs Kiara 163 · Desa ParkCity vs TTDI · PPR Kerinchi vs PPR Pantai Dalam

**Rules:** Postcode auto-assignment. 15+ homes to trigger rivalry. Monthly reset. Winner gets "Jaguh Bulan Ini" badge + 20% donation multiplier. Cross-league PPR donations earn 2x badge XP.

---

## 2. Level & XP System

| Lv | XP | EN Title | BM Title | Perk |
|----|----|----------|----------|------|
| 1 | 0 | New Plug | **Budak Baru** | Basic dashboard + 1 schedule |
| 2 | 500 | Aware Consumer | **Celik Tenaga** | Custom schedules unlocked |
| 3 | 1,500 | Smart Saver | **Jimat Cermat** | Anomaly alerts activated |
| 4 | 3,000 | Eco Champion | **Wira Hijau** | Community Grid + badge display |
| 5 | 6,000 | Power Guardian | **Pendekar Tenaga** | 2x credit earning rate |
| 6 | 12,000 | Grid Hero | **Jaguh Komuniti** | Custom badge display name |
| 7 | 25,000 | Energy Legend | **Dato' Jimat** | Priority donation + 3x multiplier |

**XP Rules:** 10 XP/kWh saved · 50 XP/credit donated (100 XP to PPR) · 200 XP/7-day streak · 500 XP/referral · 100 XP/anomaly resolved · 1,000 XP/festival challenge

---

## 3. Badge System

### Earned (4 in demo)

| Badge | Emoji | Unlock | KL Grounding |
|-------|-------|--------|-------------|
| **7-Hari Rantaian** | 🔥 | 7 days below baseline | Streak counter, always visible |
| **Jiran Terbaik** | 🛡️ | 15+ credits, 5+ to PPR | Jiran = cultural neighbor bond |
| **Anak Bulan Hero** | 🌙 | 30 days below baseline during Ramadan | Ramadan sahur/iftar savings |
| **Celik Tenaga** | 🍃 | Reach Level 3 | Energy-literate milestone |

### Locked (8 in demo)

| Badge | Emoji | Unlock | KL Grounding |
|-------|-------|--------|-------------|
| **Taugeh Champion** | 🌱 | Save RM 50+/month | Taugeh = cheapest food; slang for frugal |
| **Merdeka Saver** | 🇲🇾 | Save 55 kWh in August | "55 for Malaysia" challenge |
| **Kopitiam Regular** | ☕ | 30 days off-peak (12PM-4PM) | Escape to kopitiam during peak heat |
| **Mamak Squad** | 🫓 | 10 nights saving 8PM-12AM | Mamak = KL nightlife |
| **Balik Kampung** | 🚗 | Auto-shutdown during festive exodus | Universal Malaysian tradition |
| **LRT Warrior** | 🚆 | 30 days LRT commute | KL public transit = home savings |
| **PPR Champion** | 🏅 | Top 3 Rumah Pangsa league | B40 community recognition |
| **Dato' Jimat** | 👑 | Reach Level 7 (25,000 XP) | Ultimate badge |

---

## 4. Festival Challenge Calendar

| Month | Challenge | Theme |
|-------|----------|-------|
| January | **CNY Spring Clean** | Pre-CNY energy audit |
| February | **Chap Goh Meh Savings** | Post-CNY return to baseline |
| March | **Ramadan Nur** | Sahur/iftar optimized scheduling |
| April | **Raya Balik Kampung** | Full auto-shutdown during exodus |
| May | **Cuti-Cuti Sekolah** | School holiday family challenge |
| June | **Gawai-Kaamatan** | East Malaysia festival savings |
| July | **Haze Shield** | Jerebu season AC management |
| August | **Merdeka 55** | "55 kWh for Malaysia" |
| September | **Hari Malaysia Unity** | Cross-league PPR donation drive |
| October | **Deepavali Lights** | Efficient festive lighting |
| November | **Monsoon Watch** | Rainy season savings |
| December | **Tutup Tahun** | Year-end review + awards |

---

## 5. Demo Persona Data

| # | Name | Kawasan | League | Vibe |
|---|------|---------|--------|------|
| User | **Aisyah** Binti Rahman | TTDI | Taman | Working mom, 2 kids |
| #1 | **Kumar** A/L Muthu | Bangsar | Taman | Young professional, competitive |
| #2 | **Mei Ling** Wong | Cheras | Taman | Retiree, steady saver |
| #3 | **Raj** A/L Selvam | Damansara Hts | Taman | Family man, 3 kids |
| #4 | **Fatimah** Binti Hassan | PPR Kerinchi | Rumah Pangsa | Single mom, recipient→donor |

---

## 6. UI Widget Specifications

### 6A. StreakBadgeRow

**Location:** Top of `/community`, below App Bar, above Credits Card. **Height:** 44dp.

```
🔥 7-Hari  │ 🛡️ Jiran Terbaik │ 🌙 A.Bulan │ 🍃 Celik Tenaga │ ›
```

- Horizontal scrollable row of compact chips
- Streak chip always leftmost if streak > 0 (fire 🔥 + count + "Hari"). Pulsing opacity (1.2s loop) when active
- 4 most recent badge chips (emoji + BM name, max 12 chars)
- Chevron `›` → `/badges`
- **Streak broken:** Chip greyed, subtle shake animation on first view
- **No badges:** Single grey chip "🔒 4 lencana untuk dibuka" (40% opacity)
- **Background:** `--color-surface` (#1E1E1E) · Spacing: 8dp between chips

### 6B. NeighborhoodChallengeCard

**Location:** Between leaderboard and impact card. **Height:** 120dp (collapsed), 200dp (expanded). Hidden when no challenge.

**Collapsed:**
```
⚡ CABARAN: Haze Shield (Julai)

🏘️ TTDI (38)    ▓▓▓▓▓▓▓▓░░  72%
🏘️ Damansara (29) ▓▓▓▓▓░░░░░  58%

4 hari lagi · TTDI mendahului!
```

- Expand/collapse: 300ms easeInOut
- Progress bars: filled `--color-primary`, unfilled `--color-divider`, 8dp height
- Left border accent: 3dp `--color-primary`

**States:** Won → "🏆 TTDI menang! 20% multiplier untuk Ogos 🔥" · Lost → "Damansara menang Julai. Sertai Ogos: Merdeka 55 🇲🇾"

### 6C. Level/XP Bar (Settings Profile)

**Location:** `/settings` Profile card, between address and Edit button.

```
🏅 Celik Tenaga (Tahap 3)
▓▓▓▓▓▓▓▓▓▓▓░░░  1,840 / 3,000 XP
560 XP lagi ke Wira Hijau  ›
```
- Progress bar: 6dp height, filled `--color-primary`, unfilled `--color-background-alt`
- 600ms easeOut fill animation on load
- Entire row tappable → `/badges`

### 6D. Badge Collection Screen (`/badges`)

- App bar: "← Koleksi Lencana"
- Level/XP card at top (same as Settings but taller, 48dp)
- **Earned:** "Diperolehi (4)" · 3-column grid · Full color · 80dp × 100dp cards · Emoji (28sp) + name + unlock date
- **Locked:** "Terkunci (8)" · Same grid · 40% opacity · Unlock condition text instead of date
- Grid spacing: 12dp between cards, 16dp row gap
- **Empty:** "Belum ada lencana. Mulakan perjalanan jimat tenaga anda!"
- **New badge toast:** "🎉 Lencana baru: {name}! Ketik untuk lihat." · Slides up, 3s, auto-dismiss

---

## 7. KL Impact Card Content

```
Bulan ini, kawasan KL kita jimat:
          1,240 kWh

Setara dengan:
🏠 62 unit PPR sehari
❄️ 11 AC selama 8 jam
🚦 58 lampu Jln Bukit Bintang

84 kredit dikongsi ke 4 kawasan
                    [📤 Kongsi]
```

---

## 8. Key BM Copy Strings

| Context | BM |
|---------|----|
| Community tab | "Komuniti" |
| Credits card | "Kredit Tenaga Anda · 14 kredit tersedia" |
| Donate button | "Derma Kredit" |
| Leaderboard | "🏆 Penderma Teratas — Julai" |
| Empty leaderboard | "Tiada derma bulan ini. Jadilah yang pertama!" |
| Impact header | "🌏 Impak Komuniti" |
| Challenge header | "⚡ CABARAN: Haze Shield (Julai)" |
| Challenge status | "4 hari lagi · TTDI mendahului!" |
| Donation confirm | "Sahkan Derma" |
| Donation success | "5 kredit didermakan! Anda bantu ringankan ~RM 1.09 untuk PPR Kerinchi." |
| Donation pool | "🏠 Tabung Komuniti — Diagih ke PPR sekitar KL secara automatik" |
| PPR waiting | "3 keluarga sedang tunggu" |
| Badge hint | "🛡️ 3 lagi untuk lencana Jiran Terbaik!" |
| PPR multiplier | "Derma ke PPR = 2x XP lencana" |
| Streak label | "🔥 7-Hari" |
| Streak nudge | "Jangan patah rantaian! 7 hari berturut-turut ✓" |
| No credits | "Jimat tenaga untuk dapatkan kredit. Baseline anda sedang dikira — semak semula dalam 7 hari." |
| Badge screen | "Koleksi Lencana" |
| Earned header | "Diperolehi (4)" |
| Locked header | "Terkunci (8)" |
| Settings title | "Profil" |
| Plug section | "🔌 Plugs Saya" |
| Always-on | "📌 Sentiasa-On" |
| Notifications | "🔔 Notifikasi" |

---

## 9. Leaderboard Mock Data

```
🏢 CONDO CUP — Mont Kiara
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#1  Arcoris MK (47)    1,240 kWh  🔥 18
#2  Verve Suites (62)    980 kWh  🔥 12
#3  Kiara 163 (31)       730 kWh  🔥 9
Top donor: Aisyah (15 cr → PPR Kerinchi)

🏘️ TAMAN LEAGUE — TTDI vs Damansara
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TTDI (38)          840 kWh  🔥 14d
Damansara (29)     720 kWh  🔥 8d
                 🏆 TTDI leads!
Top TTDI: Raj (12 cr) | Top DPC: Mei Ling (9 cr)

🏠 RUMAH PANGSA — PPR Lembah Subang
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Blok A (14)  320 kWh
Blok C (11)  280 kWh
Blok B (16)  190 kWh
Credits received from Condo Cup: 48
Families helped: 8
```

---

## 10. Demo Build Scope

| Priority | Screen/Widget | Elements |
|----------|--------------|----------|
| **High** | Community Grid | StreakBadgeRow, leaderboard w/ KL names, NeighborhoodChallengeCard, KL impact card, Donation Modal w/ BM copy |
| **Medium** | Badge Collection | 4 earned + 8 locked badges in 3-column grid, Level/XP card, back nav |
| **Low** | Settings Profile | Level/XP progress bar row, tappable → `/badges` |

**New route:** `/badges` — push from Community Grid (tap StreakBadgeRow) or Settings (tap Level/XP row).
