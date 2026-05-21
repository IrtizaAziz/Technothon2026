# Claude Prompt — Build the Smart Plug Backend

*Copy everything from this line onward and give it to Claude. No additional context needed — this prompt is self-contained.*

---

## YOUR TASK

You are building the complete production backend for a smart plug energy management mobile app created for **UM Technothon 2026**, a national hackathon in Malaysia. The Flutter frontend design is already complete — your job is to build the entire Node.js + Express + TypeScript backend that replaces all mock data with real data flowing from smart plugs.

**Build every file listed in the directory structure below. Write complete, working TypeScript code. No pseudo-code, no placeholders.**

---

## PART 1 — WHAT THIS SYSTEM IS

### Project Purpose

Malaysian households waste 18–22% of residential electricity. Smart plug penetration is under 3%. Existing energy apps show kilowatt-hour graphs to users who don't understand them — 74% are abandoned within 30 days.

**Our solution:** Sonoff S31 smart plugs (flashed with Tasmota firmware) connect to an MQTT broker. A Flutter mobile app serves as the dashboard. Your Express backend is the bridge between plugs and app — it ingests telemetry, stores it, runs gamification, and serves the API that the Flutter app calls.

The system has 6 screens in the Flutter app:
1. **Dashboard** — real-time power overview with DeviceCards grouped by room
2. **Plug Detail** — live power gauge, consumption charts, schedules, budget tracking
3. **AI Insights** — anomaly detection alerts, behavioral nudges, savings summary
4. **Community Grid** — leaderboard, neighborhood challenges, credit donations, impact stats
5. **Badge Collection** — earned and locked badges in a 3-column grid, Level/XP bar
6. **Settings** — profile, plug management, notification toggles, privacy tiers

Plus two modals: AI Chat and Donation.

### Three Defining Features

| Feature | What It Does | What Backend Must Support |
|---------|-------------|--------------------------|
| **AI Chat** | Users type in natural language. "Why is my bill high?" → AI explains. No dashboard needed. | Chat endpoint with regex-based intent parser → MQTT commands / DB queries |
| **Community Grid** | Peer-to-peer energy credits. Save kWh → earn credits → donate to PPR neighbors. Leaderboards by KL kawasan. | Gamification engine, donation ledger, leaderboard MV, challenge tracking |
| **Privacy-by-Design** | Raw data stays on-device. Three-tier consent. Users control what they share. | Privacy tier field on profiles, tier check on community queries |

---

## PART 2 — SYSTEM ARCHITECTURE

### Data Flow (Plug to Screen)

```
Sonoff S31 (Tasmota firmware)
  │  tele/smartplug1/SENSOR   {"ENERGY":{"Power":160,"Voltage":240,"Current":0.67,...}}
  │  tele/smartplug2/SENSOR   (every 60 seconds per plug)
  ▼
Mosquitto MQTT Broker  ←  mqtt.js subscriber  ←  Express Backend (:3000)
  localhost:1883                                       │
                                                       │  parse JSON
                                                       │  INSERT → energy_readings hypertable
                                                       │  UPDATE → plugs cache
                                                       │  check anomaly (14d baseline, 35% threshold)
                                                       │  award XP + check badges + streak
                                                       │  REFRESH MATERIALIZED VIEW leaderboard
                                                       ▼
                                                Supabase PostgreSQL
                                                       │
                                                       │  REST API + Realtime subscriptions
                                                       ▼
                                                Flutter App
```

### Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Runtime | Node.js | 22 LTS |
| Framework | Express | 4.x |
| Language | TypeScript | 5.x |
| Database | Supabase (PostgreSQL) | 15 |
| Auth | Supabase Auth | Built-in |
| MQTT Client | mqtt.js | 6.x |
| MQTT Broker | Mosquitto | Latest |
| ORM | @supabase/supabase-js | 2.x |

### Environment Variables (`.env.example`)

```
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOi...       # PUBLIC — for Flutter SDK, includes in RLS queries
SUPABASE_SERVICE_KEY=eyJhbGciOi...    # SECRET — service_role, bypasses RLS, for backend ops

MQTT_BROKER_URL=mqtt://localhost
MQTT_BROKER_PORT=1883

PORT=3000
NODE_ENV=development

TNB_TARIFF_RATE=0.218         # Malaysian electricity rate: RM/kWh
```

---

## PART 3 — MQTT: THE HEARTBEAT OF THE BACKEND

### How the Smart Plugs Communicate

The Sonoff S31 plugs run Tasmota firmware. Each plug is configured with:
- Host: MQTT broker IP
- Port: 1883
- Topic: `smartplug1`, `smartplug2`, etc.
- FullTopic: `%topic%/%prefix%/`

This generates three MQTT topics per plug:

| Topic | Direction | Payload | When |
|-------|-----------|---------|------|
| `tele/smartplug1/SENSOR` | Plug → Broker | JSON (see below) | Every 60 seconds |
| `stat/smartplug1/POWER` | Plug → Broker | `{"POWER":"ON"}` | On relay state change |
| `cmnd/smartplug1/POWER` | Broker → Plug | `ON`, `OFF`, `TOGGLE` | When app sends command |

### What a Plug Sends (Sensor JSON)

```json
{
  "ENERGY": {
    "TotalStartTime": "2025-01-01T00:00:00",
    "Total": 12.345,
    "Yesterday": 0.420,
    "Today": 0.380,
    "Power": 160,
    "ApparentPower": 165,
    "ReactivePower": 15,
    "Factor": 0.97,
    "Voltage": 240,
    "Current": 0.667,
    "PowerFactor": 0.97,
    "Period": 60
  }
}
```

### What Your MQTT Service Must Do

Create `src/services/mqtt.service.ts`. This is the most important file in the backend — it runs on startup and never stops.

**On startup:**
1. Connect to `mqtt://localhost:1883`
2. Subscribe to `tele/+/SENSOR` (wildcard `+` catches all plug topics)
3. Subscribe to `stat/+/POWER`

**On every `tele/+/SENSOR` message — the 8-step pipeline:**

```
1. PARSE → Extract plugTopic from MQTT topic
   Example: "tele/smartplug1/SENSOR" → "smartplug1"

2. LOOKUP → Find plug in plugs table by mqtt_topic
   If not found: skip (unregistered plug)

3. STORE → INSERT into energy_readings:
   plug_id, timestamp, power_w = ENERGY.Power,
   voltage_v = ENERGY.Voltage, current_a = ENERGY.Current,
   energy_kwh = ENERGY.Total, energy_today_kwh = ENERGY.Today,
   energy_yesterday_kwh = ENERGY.Yesterday,
   power_factor = ENERGY.PowerFactor

4. CACHE → UPDATE plugs table:
   SET current_power_w, current_voltage_v, current_current_a,
   energy_today_kwh, is_online = true, updated_at = now()

5. ANOMALY CHECK → Call anomalyService.check(plugId, ENERGY.Power)
   - Get 14-day rolling average power for this plug
   - deviation = |current - baseline| / baseline × 100
   - If deviation ≥ 35%: severity = deviation ≥ 50% ? 'critical' : 'warning'
     Debounce: skip if unresolved anomaly <24h old
     impact_rm = (current - baseline) × 24 × 30 / 1000 × 0.218
     INSERT into anomalies table with title/description in BM and EN
   - If deviation < 35%: SET anomaly_status = NULL on plug

6. AWARD XP → Call gamificationService.awardEnergyXP(userId, plugId, ENERGY.Today)
   - Get user's daily baseline (7-day rolling avg kWh)
   - kWh_saved = baseline - today_kwh
   - If positive: xp = floor(kWh_saved × 10)
   - UPDATE user_levels SET current_xp += xp
   - Check level-up: if new level > old level, update level

7. UPDATE STREAK → Call gamificationService.updateStreak(userId)
   - Get today_kwh_today across ALL user's plugs (sum)
   - If sum < baseline: 
       If last_active_date was yesterday → streak_days++
       Else → streak_days = 1
   - Else → streak_days = 0
   - SET last_active_date = today

8. BADGE + LEADERBOARD (debounced)
   - Call gamificationService.checkAllBadges(userId) — evaluate all 12 badge conditions
   - If new unlocks: INSERT into user_badges
   - Refresh leaderboard (debounced, max once per 5 minutes):
     await supabaseAdmin.query('REFRESH MATERIALIZED VIEW public.leaderboard')
```

**On every `stat/+/POWER` message:**
```
Extract plugTopic → parse JSON → UPDATE plugs SET is_on = (POWER === "ON")
```

**Public methods used by routes:**
```
togglePlug(plugTopic: string)    → publish cmnd/{topic}/POWER TOGGLE (QoS 1)
setPower(plugTopic: string, 'ON'|'OFF') → publish cmnd/{topic}/POWER ON|OFF (QoS 1)
```

### Anomaly Detection Algorithm

```typescript
// For each plug reading:
async function checkAnomaly(plugId: string, currentWatts: number) {
  // Get 14-day rolling average from energy_readings
  const { data, error } = await supabaseAdmin
    .from('energy_readings')
    .select('power_w')
    .eq('plug_id', plugId)
    .gte('timestamp', new Date(Date.now() - 14 * 24 * 60 * 60 * 1000).toISOString())
    .order('timestamp', { ascending: false })
    .limit(336); // ~14 days × 24 readings/hour

  if (!data || data.length < 168) return; // Need at least 7 days

  const avg = data.reduce((sum, r) => sum + r.power_w, 0) / data.length;
  const deviation = Math.abs(currentWatts - avg) / avg * 100;

  if (deviation >= 35) {
    // Check 24-hour debounce
    const recent = await supabaseAdmin
      .from('anomalies')
      .select('id')
      .eq('plug_id', plugId)
      .gte('created_at', new Date(Date.now() - 86400000).toISOString())
      .limit(1);

    if (recent.data?.length) return; // Debounced

    const severity = deviation >= 50 ? 'critical' : 'warning';
    const plug = await getPlug(plugId);
    const impact = Math.round((currentWatts - avg) * 24 * 30 / 1000 * 0.218);

    await supabaseAdmin.from('anomalies').insert({
      plug_id: plugId, user_id: plug.user_id, severity,
      title_bm: `${plug.name} guna ${Math.round(deviation)}% lebih tenaga`,
      title_en: `${plug.name} consumed ${Math.round(deviation)}% more power`,
      description_bm: `${plug.name} anda guna ${Math.round(deviation)}% lebih tenaga semalam. Anggaran impak: RM ${impact}/bulan.`,
      description_en: `Your ${plug.name} consumed ${Math.round(deviation)}% more power yesterday. Estimated impact: RM ${impact}/month.`,
      impact_rm_per_month: impact,
    });

    await supabaseAdmin.from('plugs').update({ anomaly_status: severity }).eq('id', plugId);
  } else {
    await supabaseAdmin.from('plugs').update({ anomaly_status: null }).eq('id', plugId);
  }
}
```

---

## PART 4 — DATABASE SCHEMA

Create `supabase/schema.sql` with ALL 13 database objects. Execute this on a Supabase project to create the database.

### Entity Relationships

```
auth.users (Supabase built-in)
  └── profiles (1:1 extension)
        ├── plugs (1:N)
        │     ├── energy_readings (1:N, hypertable)
        │     ├── schedules (1:N)
        │     └── anomalies (1:N)
        ├── user_badges (1:N) → badges (N:1 reference)
        ├── user_levels (1:1)
        ├── donations (1:N as donor)
        └── notification_settings (1:1)

challenges (standalone)
  └── challenge_participants (1:N)

leaderboard (materialized view — aggregates profiles + donations)
```

### Complete DDL

```sql
-- ============================================================
-- 1. profiles
-- Used by: Settings screen profile card, all auth-dependent routes
-- ============================================================
CREATE TABLE public.profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name TEXT NOT NULL,
  email TEXT NOT NULL,
  address TEXT DEFAULT '',           -- "12, Jalan Bahagia, TTDI"
  kawasan TEXT NOT NULL,             -- "TTDI", "Bangsar", "PPR Kerinchi"
  league TEXT NOT NULL CHECK (league IN ('condo', 'taman', 'ppr')),
  privacy_tier SMALLINT DEFAULT 1 CHECK (privacy_tier BETWEEN 1 AND 3),
  avatar_url TEXT DEFAULT '',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================
-- 2. plugs
-- Used by: Dashboard (DeviceCards), Plug Detail, Settings (Plugs Saya)
-- mqtt_topic is UNIQUE — each physical plug maps to one Tasmota topic
-- ============================================================
CREATE TABLE public.plugs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  mqtt_topic TEXT NOT NULL UNIQUE,          -- "smartplug1", "smartplug2"
  name TEXT NOT NULL,                        -- "AC Bedroom", "TV"
  room TEXT NOT NULL,                        -- "Living Room", "Kitchen"
  appliance_icon TEXT DEFAULT '🔌',          -- emoji: ❄️ 🖥️ 🧊 🍚
  appliance_type TEXT DEFAULT '',            -- 'ac', 'tv', 'fridge', 'rice_cooker'
  is_on BOOLEAN DEFAULT FALSE,               -- current relay state
  is_online BOOLEAN DEFAULT FALSE,           -- MQTT connected?
  is_always_on BOOLEAN DEFAULT FALSE,        -- exclude from "Leaving Home"
  vampire_auto_off BOOLEAN DEFAULT TRUE,     -- auto-cutoff standby <5W
  monthly_budget_rm DECIMAL(10,2) DEFAULT NULL,
  current_power_w REAL DEFAULT 0,            -- cached from latest telemetry
  current_voltage_v REAL DEFAULT 0,
  current_current_a REAL DEFAULT 0,
  energy_today_kwh REAL DEFAULT 0,           -- from Tasmota "Today"
  energy_month_kwh REAL DEFAULT 0,
  anomaly_status TEXT DEFAULT NULL,          -- null | 'warning' | 'critical'
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_plugs_user ON public.plugs(user_id);
CREATE INDEX idx_plugs_topic ON public.plugs(mqtt_topic);

-- ============================================================
-- 3. energy_readings (TIMESCALE HYPERTABLE)
-- Tip: If TimescaleDB is not available in Supabase free tier,
-- still create the table normally, just skip create_hypertable().
-- Used by: Plug Detail (Consumption Chart 7d/30d, Stat Row)
-- 1440 rows/day/plug at 60s intervals
-- ============================================================
CREATE TABLE public.energy_readings (
  id BIGSERIAL PRIMARY KEY,
  plug_id UUID NOT NULL REFERENCES public.plugs(id) ON DELETE CASCADE,
  timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  power_w REAL NOT NULL,
  apparent_power_w REAL DEFAULT 0,
  reactive_power_w REAL DEFAULT 0,
  voltage_v REAL DEFAULT 0,
  current_a REAL DEFAULT 0,
  power_factor REAL DEFAULT 0,
  energy_kwh REAL DEFAULT 0,
  energy_today_kwh REAL DEFAULT 0,
  energy_yesterday_kwh REAL DEFAULT 0,
  relay_state BOOLEAN DEFAULT TRUE
);

-- Try to convert to hypertable (gracefully fail if extension missing)
DO $$
BEGIN
  PERFORM create_hypertable('energy_readings', 'timestamp',
    chunk_time_interval => INTERVAL '1 day');
EXCEPTION WHEN OTHERS THEN
  RAISE NOTICE 'TimescaleDB not available — using standard table for energy_readings';
END $$;

CREATE INDEX idx_readings_plug_time ON public.energy_readings(plug_id, timestamp DESC);

-- ============================================================
-- 4. badges (12 rows, static reference data)
-- Used by: Badge Collection screen (earned + locked grid)
-- ============================================================
CREATE TABLE public.badges (
  id TEXT PRIMARY KEY,                      -- 'streak', 'jiran', 'bulan', etc.
  name_bm TEXT NOT NULL,                   -- "7-Hari Rantaian"
  name_en TEXT NOT NULL,                   -- "7-Day Streak"
  emoji TEXT NOT NULL,                     -- 🔥 🛡️ 🌙 🍃
  unlock_condition_bm TEXT NOT NULL,
  unlock_condition_en TEXT NOT NULL,
  category TEXT DEFAULT 'general',         -- 'general' | 'festival' | 'league'
  sort_order INT DEFAULT 0
);

-- ============================================================
-- 5. user_badges
-- ============================================================
CREATE TABLE public.user_badges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  badge_id TEXT NOT NULL REFERENCES public.badges(id) ON DELETE CASCADE,
  unlocked_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, badge_id)
);
CREATE INDEX idx_user_badges ON public.user_badges(user_id);

-- ============================================================
-- 6. user_levels
-- Used by: Settings (Level/XP bar), Badge Collection (level card),
--          Community Grid (streak, credits),
--          gamification routes
-- ============================================================
CREATE TABLE public.user_levels (
  user_id UUID PRIMARY KEY REFERENCES public.profiles(id) ON DELETE CASCADE,
  level INT DEFAULT 1 CHECK (level BETWEEN 1 AND 7),
  current_xp INT DEFAULT 0,
  streak_days INT DEFAULT 0,
  last_active_date DATE DEFAULT NULL,      -- for streak continuity check
  monthly_kwh_saved REAL DEFAULT 0,
  total_credits_earned INT DEFAULT 0,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================
-- 7. donations — credit donation ledger
-- Used by: Community Grid (Leaderboard, Donation Modal, Impact)
-- ============================================================
CREATE TABLE public.donations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  to_user_id UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  to_pool TEXT DEFAULT NULL,              -- 'tabung_komuniti' | 'PPR Kerinchi' | 'PPR Lembah Subang'
  credits INT NOT NULL CHECK (credits > 0),
  amount_rm DECIMAL(10,2) NOT NULL,        -- credits × 0.218
  created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_donations_from ON public.donations(from_user_id);
CREATE INDEX idx_donations_date ON public.donations(created_at DESC);

-- ============================================================
-- 8. challenges — monthly neighborhood challenges
-- Used by: Community Grid (NeighborhoodChallengeCard)
-- ============================================================
CREATE TABLE public.challenges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,                       -- "Haze Shield", "Merdeka 55"
  month TEXT NOT NULL,                      -- "Julai", "Ogos"
  kawasan_a TEXT NOT NULL,                  -- "TTDI"
  kawasan_b TEXT NOT NULL,                  -- "Damansara"
  target_kwh REAL NOT NULL,                 -- 1200
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  winner_kawasan TEXT DEFAULT NULL,
  status TEXT DEFAULT 'active' CHECK (status IN ('active', 'ended')),
  reward_badge_id TEXT REFERENCES public.badges(id)
);

-- ============================================================
-- 9. challenge_participants — per-kawasan progress
-- ============================================================
CREATE TABLE public.challenge_participants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  challenge_id UUID NOT NULL REFERENCES public.challenges(id) ON DELETE CASCADE,
  kawasan TEXT NOT NULL,
  homes_count INT DEFAULT 0,
  kwh_saved REAL DEFAULT 0,
  top_donor_name TEXT DEFAULT '',
  top_donor_credits INT DEFAULT 0,
  streak_days INT DEFAULT 0,
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(challenge_id, kawasan)
);

-- ============================================================
-- 10. schedules — per-plug time schedules
-- Used by: Plug Detail (Schedule list)
-- days_of_week is a bitmask: 1=Mon, 2=Tue, 4=Wed, 8=Thu, 16=Fri, 32=Sat, 64=Sun
-- ============================================================
CREATE TABLE public.schedules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  plug_id UUID NOT NULL REFERENCES public.plugs(id) ON DELETE CASCADE,
  name TEXT DEFAULT '',                     -- "Weekdays", "Weekend"
  days_of_week SMALLINT NOT NULL,           -- bitmask
  time_on TIME NOT NULL,
  time_off TIME NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_schedules_plug ON public.schedules(plug_id);

-- ============================================================
-- 11. anomalies — detected appliance issues
-- Used by: AI Insights (AnomalyCards), Plug Detail (anomaly banner)
-- ============================================================
CREATE TABLE public.anomalies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  plug_id UUID NOT NULL REFERENCES public.plugs(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  severity TEXT NOT NULL CHECK (severity IN ('critical', 'warning')),
  title_bm TEXT NOT NULL,
  title_en TEXT NOT NULL,
  description_bm TEXT NOT NULL,
  description_en TEXT NOT NULL,
  impact_rm_per_month REAL DEFAULT 0,
  is_read BOOLEAN DEFAULT FALSE,
  is_dismissed BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_anomalies_user ON public.anomalies(user_id);
CREATE INDEX idx_anomalies_plug ON public.anomalies(plug_id);

-- ============================================================
-- 12. notification_settings — per-user toggle prefs
-- Used by: Settings screen (Notification toggles)
-- ============================================================
CREATE TABLE public.notification_settings (
  user_id UUID PRIMARY KEY REFERENCES public.profiles(id) ON DELETE CASCADE,
  all_enabled BOOLEAN DEFAULT TRUE,
  anomalies BOOLEAN DEFAULT TRUE,
  budget BOOLEAN DEFAULT TRUE,
  tips BOOLEAN DEFAULT TRUE,
  community BOOLEAN DEFAULT TRUE,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================
-- 13. leaderboard — MATERIALIZED VIEW
-- Partitioned by league, ranked by credits_donated this month
-- Refresh after each donation or on-demand
-- Used by: Community Grid (Leaderboard section)
-- ============================================================
CREATE MATERIALIZED VIEW public.leaderboard AS
SELECT
  ROW_NUMBER() OVER (
    PARTITION BY p.league
    ORDER BY COALESCE(SUM(d.credits), 0) DESC
  ) AS rank,
  p.id AS user_id,
  p.full_name,
  p.kawasan,
  p.league,
  COALESCE(SUM(d.credits), 0) AS credits_donated,
  COALESCE(SUM(d.credits) FILTER (WHERE d.to_pool LIKE 'PPR%'), 0) AS credits_to_ppr
FROM public.profiles p
LEFT JOIN public.donations d
  ON d.from_user_id = p.id
  AND d.created_at >= date_trunc('month', CURRENT_DATE)
GROUP BY p.id, p.full_name, p.kawasan, p.league
ORDER BY p.league, credits_donated DESC;
```

### Row-Level Security Policies

```sql
-- Profiles
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own profile select" ON public.profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "own profile update" ON public.profiles FOR UPDATE USING (auth.uid() = id);

-- Plugs
ALTER TABLE public.plugs ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own plugs" ON public.plugs FOR ALL USING (auth.uid() = user_id);

-- Energy readings → backend only (uses service_role key to bypass)
ALTER TABLE public.energy_readings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "service all" ON public.energy_readings FOR ALL USING (true);

-- Badges → public read
ALTER TABLE public.badges ENABLE ROW LEVEL SECURITY;
CREATE POLICY "public read" ON public.badges FOR SELECT USING (true);

-- User badges → own only
ALTER TABLE public.user_badges ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own badges" ON public.user_badges FOR SELECT USING (auth.uid() = user_id);

-- Donations → own + community pool is public
ALTER TABLE public.donations ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own donations"   ON public.donations FOR SELECT USING (auth.uid() = from_user_id);
CREATE POLICY "pool is public"  ON public.donations FOR SELECT USING (to_pool IS NOT NULL);

-- Notification settings → own only
ALTER TABLE public.notification_settings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own notifs" ON public.notification_settings FOR ALL USING (auth.uid() = user_id);

-- Note: leaderboard is a materialized view — it does NOT support RLS.
-- Use a wrapper function with SECURITY DEFINER or query it only via service_role.
```

### Seed Data — 12 Badges

Create `supabase/seed_badges.sql`:

```sql
INSERT INTO public.badges (id, name_bm, name_en, emoji, unlock_condition_bm, unlock_condition_en, category, sort_order) VALUES
('streak',   '7-Hari Rantaian',  '7-Day Streak',        '🔥', '7 hari berturut-turut bawah baseline', '7 consecutive days below baseline', 'general', 1),
('jiran',    'Jiran Terbaik',    'Best Neighbor',        '🛡️', 'Derma 15+ kredit, 5+ ke PPR', 'Donate 15+ credits, 5+ to PPR', 'general', 2),
('celik',    'Celik Tenaga',     'Energy Literate',      '🍃', 'Capai Tahap 3', 'Reach Level 3', 'general', 3),
('bulan',    'Anak Bulan Hero',  'Ramadan Champion',     '🌙', '30 hari bawah baseline semasa Ramadan', '30 days below baseline during Ramadan', 'festival', 4),
('taugeh',   'Taugeh Champion',  'Bean Sprout Champ',    '🌱', 'Jimat RM 50+ sebulan', 'Save RM 50+/month', 'general', 5),
('merdeka',  'Merdeka Saver',    'Merdeka Saver',        '🇲🇾', 'Jimat 55 kWh semasa Ogos', 'Save 55 kWh in August', 'festival', 6),
('kopitiam', 'Kopitiam Regular', 'Kopitiam Regular',     '☕', '30 hari jimat luar waktu puncak (12PM-4PM)', '30 days off-peak savings (12PM-4PM)', 'general', 7),
('mamak',    'Mamak Squad',      'Mamak Squad',          '🫓', '10 malam jimat 8PM-12AM', '10 nights saving 8PM-12AM', 'general', 8),
('balik',    'Balik Kampung',    'Balik Kampung',        '🚗', 'Auto-tutup semasa minggu cuti perayaan', 'Auto-shutdown during festive exodus week', 'general', 9),
('lrt',      'LRT Warrior',      'LRT Warrior',          '🚆', '30 hari guna LRT untuk ulang-alik', '30 days LRT commute savings', 'general', 10),
('ppr',      'PPR Champion',     'PPR Champion',         '🏅', 'Top 3 Liga Rumah Pangsa', 'Top 3 Rumah Pangsa league', 'league', 11),
('dato',     'Dato'' Jimat',      'Sir Saves-a-Lot',     '👑', 'Capai Tahap 7 (25,000 XP)', 'Reach Level 7 (25,000 XP)', 'general', 12);
```

---

## PART 5 — GAMIFICATION ENGINE

Create `src/services/gamification.service.ts` and `src/models/constants.ts`.

### Level Thresholds and Titles

```typescript
// src/models/constants.ts
export const LEVEL_THRESHOLDS: number[] = [0, 500, 1500, 3000, 6000, 12000, 25000];

export const LEVEL_TITLES_BM: Record<number, string> = {
  1: 'Budak Baru', 2: 'Celik Tenaga', 3: 'Jimat Cermat',
  4: 'Wira Hijau', 5: 'Pendekar Tenaga', 6: 'Jaguh Komuniti', 7: "Dato' Jimat",
};

export const LEVEL_TITLES_EN: Record<number, string> = {
  1: 'New Plug', 2: 'Energy Literate', 3: 'Smart Saver',
  4: 'Eco Champion', 5: 'Power Guardian', 6: 'Community Hero', 7: 'Sir Saves-a-Lot',
};

export function getLevel(xp: number): number {
  for (let i = LEVEL_THRESHOLDS.length - 1; i >= 0; i--) {
    if (xp >= LEVEL_THRESHOLDS[i]) return i + 1;
  }
  return 1;
}
```

### XP Rules

| Action | XP | Implementation |
|--------|-----|----------------|
| 1 kWh saved below baseline | 10 XP | Awarded in MQTT pipeline step 6 (per telemetry reading) |
| 1 credit donated (regular) | 50 XP | Awarded in donation route handler |
| 1 credit donated (to PPR) | 100 XP | 2x multiplier, checked by `to_pool LIKE 'PPR%'` |
| 7-day streak bonus | 200 XP | Awarded when streak_days % 7 === 0 |
| Anomaly resolved (dismissed) | 100 XP | Awarded when anomaly.is_dismissed set to true |
| Festival challenge won | 1,000 XP | Awarded when challenge.status = 'ended' and winner = user's kawasan |

### Credit System

```
credits_earned = floor(user_levels.monthly_kwh_saved)  (updated daily)
credits_donated = SUM(donations.credits WHERE from_user_id = user)
credits_available = credits_earned - credits_donated

credit_to_rm(credits) = credits × TNB_TARIFF_RATE (0.218)
```

### Badge Unlock Logic

```typescript
// src/services/gamification.service.ts

interface BadgeCheck {
  id: string;
  condition: (state: UserState) => boolean;
}

// UserState is aggregated from user_levels + donations + plugs
interface UserState {
  level: number;
  streak_days: number;
  monthly_savings_rm: number;
  total_donations: number;
  ppr_donations: number;
  league: string;
  league_rank: number;
}

const BADGE_CHECKS: BadgeCheck[] = [
  { id: 'streak', condition: (u) => u.streak_days >= 7 },
  { id: 'jiran',  condition: (u) => u.total_donations >= 15 && u.ppr_donations >= 5 },
  { id: 'celik',  condition: (u) => u.level >= 3 },
  { id: 'bulan',  condition: (u) => u.streak_days >= 30 && isRamadan(new Date()) },
  { id: 'taugeh', condition: (u) => u.monthly_savings_rm >= 50 },
  { id: 'merdeka',condition: (u) => u.august_kwh_saved >= 55 && new Date().getMonth() === 7 },
  // ... implement all 12 checks
  { id: 'dato',   condition: (u) => u.level >= 7 },
];

async function checkAllBadges(userId: string): Promise<string[]> {
  // 1. Aggregate user state from DB
  // 2. Get already-earned badge IDs
  // 3. For each unearned badge, test condition
  // 4. INSERT into user_badges for new unlocks
  // 5. Return array of newly unlocked badge IDs
}

// Helper: check if current date falls within Ramadan (March-April in 2026)
function isRamadan(date: Date): boolean {
  // 2026 Ramadan: approximately Feb 18 - Mar 19
  // Adjust for actual year
  return date.getMonth() >= 1 && date.getMonth() <= 2;
}
```

---

## PART 6 — API ENDPOINTS (30 endpoints, 7 route files)

### Two Supabase Clients

```typescript
// src/config/supabase.ts

import { createClient } from '@supabase/supabase-js';
import { env } from './env';

// Admin client: service_role key — bypasses RLS, used for inserts, leaderboard refresh, MQTT handler
export const supabaseAdmin = createClient(env.SUPABASE_URL, env.SUPABASE_SERVICE_KEY, {
  auth: { autoRefreshToken: false, persistSession: false },
});

// User context client: created per-request with the user's JWT
export function createUserClient(jwt: string) {
  return createClient(env.SUPABASE_URL, env.SUPABASE_ANON_KEY, {
    global: { headers: { Authorization: `Bearer ${jwt}` } },
  });
}
```

### Auth Middleware

```typescript
// src/middleware/auth.ts

import { Request, Response, NextFunction } from 'express';
import { supabaseAdmin } from '../config/supabase';

// Extend Express Request
declare global {
  namespace Express {
    interface Request {
      userId: string;
    }
  }
}

export async function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const header = req.headers.authorization;
  if (!header) return res.status(401).json({ error: 'Missing authorization header' });

  const token = header.replace('Bearer ', '');
  const { data: { user }, error } = await supabaseAdmin.auth.getUser(token);

  if (error || !user) return res.status(401).json({ error: 'Invalid token' });

  req.userId = user.id;
  next();
}
```

### Route 1: Auth (`src/routes/auth.routes.ts`)

```
POST /api/auth/register
  Body: { email: string, password: string, full_name: string, kawasan: string, league: string }
  1. Create user in Supabase Auth: supabaseAdmin.auth.admin.createUser({ email, password, email_confirm: true })
  2. INSERT into profiles (id, full_name, email, kawasan, league, privacy_tier=2)
  3. INSERT into user_levels (user_id, level=1, current_xp=0, streak_days=0, total_credits_earned=0)
  4. INSERT into notification_settings (user_id) — all defaults true
  Response: { user: { id, email }, profile: { full_name, kawasan, league } }

POST /api/auth/login
  Body: { email: string, password: string }
  1. supabaseAdmin.auth.signInWithPassword({ email, password })
  Response: { access_token, refresh_token, user: { id, email } }

GET /api/auth/me  [auth required]
  Query profiles + user_levels joined. Return:
  { id, email, full_name, kawasan, league, privacy_tier, level, current_xp, streak_days, credits_available }
```

### Route 2: Plugs (`src/routes/plugs.routes.ts`)

```
ALL endpoints require auth.

GET /api/plugs → Dashboard data source
  Query all user's plugs.
  Group by room, calculate room total_watts.
  Calculate summary: total_watts = SUM(current_power_w), daily_cost = sum_watts / 1000 × 24 × 0.218.
  Each plug has status:
    - online_normal:   is_online=true, current_power_w ≤ 200
    - online_high:     is_online=true, current_power_w 200-800
    - online_critical: is_online=true, current_power_w > 800
    - offline:         is_online=false
  Response:
  {
    "summary": { "total_watts": 420, "daily_cost_rm": 2.19, "trend_vs_yesterday_pct": 12, "plug_count": 4, "online_count": 4 },
    "rooms": [{ "name": "Living Room", "total_watts": 180, "plugs": [{ "id": "uuid", "name": "TV", "icon": "🖥️", "watts": 12, "status": "online_normal", "is_on": true }] }]
  }

GET /api/plugs/:id → Plug Detail data source
  Return plug row + calculated budget + active anomaly.
  Budget status: green (spent ≤70% of limit) | warning (70-90%) | exceeded (>90%).
  If no budget set: budget = null.
  Response:
  {
    "id": "uuid", "name": "AC Bedroom", "room": "Living Room", "icon": "❄️", "appliance_type": "ac",
    "is_on": true, "is_online": true, "is_always_on": false, "vampire_auto_off": true,
    "latest": { "power_w": 160, "voltage_v": 240, "current_a": 0.67, "energy_today_kwh": 2.4, "energy_month_kwh": 48, "daily_cost_rm": 0.35, "monthly_cost_rm": 10.50 },
    "budget": { "limit_rm": 60, "spent_rm": 48, "progress_pct": 80, "days_remaining": 12, "status": "warning" } | null,
    "anomaly": { "id": "uuid", "severity": "critical", "title_bm": "...", "description_bm": "..." } | null
  }

GET /api/plugs/:id/readings?range=7d|30d → Consumption Chart
  Query energy_readings. Group by hour: AVG(power_w), date_trunc('hour', timestamp).
  Range: 7d = 168 points, 30d = 720 points.
  Response: { "range": "7d", "points": [{ "timestamp": "2026-07-14T00:00:00Z", "power_w": 120 }] }

POST /api/plugs → Register plug after BLE pairing
  Body: { mqtt_topic: string, name: string, room: string, appliance_icon: string, appliance_type: string }
  INSERT into plugs with user_id from auth.
  Response: the created plug object

PATCH /api/plugs/:id → Update plug
  Body: { name?, room?, appliance_icon?, appliance_type?, monthly_budget_rm?, is_always_on?, vampire_auto_off? }
  UPDATE plugs SET ... WHERE id = :id AND user_id = auth.userId
  Response: updated plug

POST /api/plugs/:id/toggle → Toggle relay
  1. Get plug by id (verify ownership)
  2. Call mqttService.togglePlug(plug.mqtt_topic)
  3. Return: { success: true, new_state: !plug.is_on }

DELETE /api/plugs/:id → Remove plug
  DELETE FROM plugs WHERE id = :id AND user_id = auth.userId
  Response: { success: true }

GET    /api/plugs/:id/schedules      → List schedules for plug
POST   /api/plugs/:id/schedules      → Create: { name, days_of_week (bitmask), time_on, time_off }
PATCH  /api/plugs/schedules/:sid    → Update schedule
DELETE /api/plugs/schedules/:sid    → Delete schedule

POST /api/plugs/mass-off → "Leaving Home"
  1. Find all user's plugs WHERE is_always_on = false AND is_on = true
  2. For each: mqttService.setPower(plug.mqtt_topic, 'OFF')
  3. Response: { turned_off: 3, skipped: 1, message: "Turned off 3 devices. Fridge still running." }

POST /api/plugs/mass-on → "All On"
  1. Find all user's plugs WHERE is_on = false
  2. For each: mqttService.setPower(plug.mqtt_topic, 'ON')
  3. Response: { turned_on: 4, message: "All devices turned on." }
```

### Route 3: Gamification (`src/routes/gamification.routes.ts`)

```
ALL endpoints require auth.

GET /api/gamification/profile → Level/XP/Streak/Credits
  Query user_levels. Calculate level details.
  Response:
  { "level": 3, "title_bm": "Celik Tenaga", "title_en": "Energy Literate",
    "current_xp": 1840, "required_xp": 3000, "xp_to_next": 560,
    "next_title_bm": "Wira Hijau", "next_title_en": "Eco Champion",
    "progress_pct": 61, "streak_days": 7,
    "credits_available": 14, "total_credits_earned": 22,
    "monthly_kwh_saved": 84 }

GET /api/gamification/badges → All badges with earn status
  1. Query ALL badges (12 rows)
  2. Query user_badges for this user
  3. Split into earned (ones in user_badges) and locked (the rest)
  Response:
  { "earned": [{ "id": "streak", "name_bm": "7-Hari Rantaian", "emoji": "🔥", "unlock_condition_bm": "...", "unlocked_at": "..." }],
    "locked":  [{ "id": "taugeh", "name_bm": "Taugeh Champion", "emoji": "🌱", "unlock_condition_bm": "..." }] }

GET /api/gamification/credits → Credit balance + transaction history
  Response:
  { "available": 14, "earned": 22, "donated": 8,
    "history": [{ "id": "uuid", "credits": 5, "to_pool": "PPR Kerinchi", "amount_rm": 1.09, "created_at": "..." }] }

POST /api/gamification/donate → Donate credits
  Body: { "recipient_type": "pool"|"ppr", "recipient_ppr": "PPR Kerinchi"?, "credits": 5 }
  1. Validate credits > 0 and credits ≤ credits_available
  2. INSERT into donations (from_user_id, to_pool, credits, amount_rm = credits × 0.218)
  3. Award XP: 50 XP/credit (regular pool) or 100 XP/credit (to PPR pool). UPDATE user_levels
  4. Check badge unlock (specifically 'jiran' badge)
  5. Refresh leaderboard: supabaseAdmin.query('REFRESH MATERIALIZED VIEW public.leaderboard')
  6. Check remaining progress toward Jiran Terbaik badge
  Response:
  { "success": true, "credits_donated": 5, "amount_rm": 1.09, "ppr": "PPR Kerinchi",
    "badge_hint": { "badge_name_bm": "Jiran Terbaik", "remaining": 3, "total": 15 } }
```

### Route 4: Community (`src/routes/community.routes.ts`)

```
ALL endpoints require auth.

GET /api/community/leaderboard?league=all|condo|taman|ppr
  Query materialized view. Filter by league if specified.
  Mark is_current_user = true for entries matching auth userId.
  Response:
  { "league": "taman", "month": "Julai",
    "entries": [{ "rank": 1, "name": "Kumar", "kawasan": "Bangsar", "credits_donated": 22,
                  "credits_to_ppr": 12, "ppr_name": "PPR Kerinchi", "is_current_user": false },
                { "rank": 3, "name": "Aisyah", "kawasan": "TTDI", "credits_donated": 14,
                  "credits_to_ppr": 6, "ppr_name": "PPR Lembah Subang", "is_current_user": true }] }

GET /api/community/challenges
  Find active challenge WHERE user's kawasan matches (in kawasan_a or kawasan_b).
  Response:
  { "active": { "id": "uuid", "name": "Haze Shield", "month": "Julai",
      "target_kwh": 1200, "days_remaining": 4, "leading_kawasan": "TTDI",
      "kawasan_a": { "name": "TTDI", "homes": 38, "progress_pct": 72, "kwh_saved": 864,
                     "top_donor": "Raj", "top_donor_credits": 12, "streak_days": 14 },
      "kawasan_b": { "name": "Damansara", "homes": 29, "progress_pct": 58, "kwh_saved": 696,
                     "top_donor": "Mei Ling", "top_donor_credits": 9, "streak_days": 8 }
    } } | { "active": null }

GET /api/community/challenges/:id → Challenge detail

GET /api/community/impact → Aggregate community stats
  Query: total kWh saved across all users with privacy_tier ≥ 2.
  Calculate equivalents:
    ppr_units = kWh_saved / 20        (1 PPR unit uses ≈20 kWh/day)
    ac_8hrs   = kWh_saved / 112       (1 AC × 8hrs at 1.5kW)
    lamps     = kWh_saved / 21.4      (1 street lamp ≈21.4 kWh/month)
  Response:
  { "total_kwh_saved": 1240, "total_credits_shared": 84, "neighborhoods_count": 4,
    "equivalents": { "ppr_units_per_day": 62, "ac_units_8hrs": 11, "street_lamps_bukit_bintang": 58 } }
```

### Route 5: Insights (`src/routes/insights.routes.ts`)

```
ALL endpoints require auth.

GET /api/insights/anomalies → Anomalies + Nudges + Savings
  1. Query anomalies table (WHERE is_dismissed = false, limit 10)
  2. Generate nudges from nudge rules (see nudge.service.ts below)
  3. Calculate savings: sum kWh saved this month × 0.218
  Response:
  { "anomalies": [{ "id": "uuid", "plug_name": "Fridge", "plug_icon": "🧊", "severity": "critical",
      "title_bm": "Peti sejuk guna 35% lebih tenaga",
      "description_bm": "Peti sejuk anda guna 35% lebih tenaga semalam. Anggaran impak: RM 18/bulan.",
      "impact_rm_per_month": 18, "created_at": "2026-07-14T10:30:00Z" }],
    "suggestions": [{ "type": "nudge", "title_bm": "AC anda telah berjalan selama 4 jam. Buka tingkap?",
      "subtitle_bm": "Suhu luar 26°C — nyaman untuk buka tingkap.",
      "action_type": "open_schedule", "action_plug_id": "uuid" }],
    "savings": { "current_rm": 18.40, "goal_rm": 25.00, "progress_pct": 73 } }

PATCH /api/insights/anomalies/:id → Mark as read or dismissed
  Body: { action: "read" | "dismiss" }
  UPDATE anomalies SET is_read = true OR is_dismissed = true
  If dismissed: award 100 XP (anomaly resolved)
  Response: { success: true }
```

### Nudge Rules (`src/services/nudge.service.ts`)

```typescript
// Simple rule-based nudge generator for the demo
async function generateNudges(userId: string): Promise<Nudge[]> {
  const nudges: Nudge[] = [];
  const plugs = await getUserPlugs(userId);

  for (const plug of plugs) {
    // 1. AC running >4 hours → suggest opening windows
    if (plug.appliance_type === 'ac' && plug.is_on && plug.hoursSinceLastToggle() > 4) {
      nudges.push({
        type: 'nudge',
        title_bm: 'AC anda telah berjalan selama 4 jam. Buka tingkap?',
        title_en: 'Your AC has been running for 4 hours. Open windows?',
        subtitle_bm: 'Suhu luar 26°C — nyaman untuk buka tingkap.',
        subtitle_en: "It's 26°C outside — nice weather for open windows.",
        action_type: 'open_schedule',
        action_plug_id: plug.id,
      });
    }

    // 2. Vampire power detected: OFF relay but still drawing <5W
    if (!plug.is_on && plug.current_power_w > 0 && plug.current_power_w < 5 && plug.vampire_auto_off) {
      nudges.push({
        type: 'nudge',
        title_bm: `Vampire cutoff dicetuskan — ${plug.name} guna ${plug.current_power_w}W selama 30+ min.`,
        title_en: `Vampire cutoff triggered — ${plug.name} drawing ${plug.current_power_w}W for 30+ min.`,
        action_type: 'undo_vampire',
        action_plug_id: plug.id,
      });
    }
  }

  // 3. Weekly savings record
  const thisWeek = await getWeeklySavings(userId);
  const bestWeek = await getBestWeeklySavings(userId);
  if (thisWeek > bestWeek) {
    nudges.push({
      type: 'nudge',
      title_bm: 'Anda guna 18% kurang tenaga minggu ini. Rekod baru!',
      title_en: 'You used 18% less energy this week. New personal record!',
      action_type: 'view_stats',
    });
  }

  return nudges;
}
```

### Route 6: Chat (`src/routes/chat.routes.ts`)

```
POST /api/chat  [auth required]
  Body: { message: string }
  Parse the message with regex rules. No LLM — rule-based only.

  INTENT_PATTERNS (in order of priority):
  1. toggle_device:  /turn (on|off) (?:the )?(.+)/i  |  /(?:switch|flip) (on|off) (.+)/i
  2. top_consumer:   /what(?:'s| is) using the most/i  |  /highest (?:power|wattage)/i
  3. check_bill:     /why is my bill (?:so )?high/i  |  /bill (?:increase|spike)/i
  4. list_anomalies: /(?:show |list |any |what )?anomal/i
  5. query_wattage:  /how much (?:power|energy) (?:is|does) the (.+) us/i
  6. bill_forecast:  /(?:forecast|predict|estimate) (?:this month|bill)/i
  7. get_status:     /is the (.+) (?:on|off)/i

  For toggle_device:
    - Extract action (on/off) and device name
    - Fuzzy-find plug by name (case-insensitive contains match)
    - If found: mqttService.setPower(plug.mqtt_topic, action.toUpperCase())
    - Return: { text_bm: "Selesai ✓ — {name} di{tutup|buka}. Jimat ~RM {rate}/jam." }

  For top_consumer:
    - Query plugs ORDER BY current_power_w DESC LIMIT 3
    - Return: { text_bm: "{name} ({icon}) pada {watts}W ({pct}% dari jumlah). Seterusnya: {name2}." }

  For check_bill:
    - Compare this month kWh vs last month kWh
    - Return: { text_bm: "AC anda berjalan {hrs} jam/hari minggu ini vs {hrs2} minggu lepas. Nak saya cadangkan jadual baru?" }

  For list_anomalies:
    - Query active anomalies. If none: "Tiada anomali. Semua peranti berfungsi normal ✅"
    - If found: "Saya jumpa {count} anomali: ..."

  Response format:
  { "text_bm": "Selesai ✓ — AC Bedroom ditutup. Jimat ~RM 0.26/jam.",
    "text_en": "Done ✓ — AC Bedroom turned off. Saving ~RM 0.26/hour.",
    "action": { "type": "toggle", "plug_id": "uuid" } | null }
```

### Route 7: Settings (`src/routes/settings.routes.ts`)

```
ALL endpoints require auth.

GET /api/settings → Full settings
  Return: profile + notification_settings + privacy_tier + always-on plugs IDs
  Response:
  { "profile": { "full_name": "...", "email": "...", "address": "...", "level_title_bm": "...", "level": 3, "current_xp": 1840, "required_xp": 3000, "xp_to_next": 560, "next_title_bm": "..." },
    "notifications": { "all_enabled": true, "anomalies": true, "budget": true, "tips": false, "community": true },
    "privacy_tier": 2,
    "plugs": [{"id": "uuid", "name": "AC Bedroom", "icon": "❄️", "room": "Living Room", "is_online": true}],
    "always_on_plugs": ["uuid1", "uuid2"] }

PATCH /api/settings/profile → Update name, address
  Body: { full_name?, address? }
  UPDATE profiles SET ... WHERE id = auth.userId
  Response: updated profile

PATCH /api/settings/notifications → Update toggle prefs
  Body: { all_enabled?, anomalies?, budget?, tips?, community? }
  UPDATE notification_settings SET ... WHERE user_id = auth.userId
  Response: updated settings

PATCH /api/settings/privacy → Update privacy tier
  Body: { privacy_tier: 1|2|3 }
  UPDATE profiles SET privacy_tier = :val WHERE id = auth.userId
  Response: { privacy_tier: 2 }
```

---

## PART 7 — SERVICE FILES

Create these service files. Each exports a class with static or instance methods.

| File | Key Methods | Used By |
|------|-----------|---------|
| `mqtt.service.ts` | `start()`, `handleTelemetry()`, `handleRelayState()`, `togglePlug()`, `setPower()` | `index.ts` (on boot), plugs routes |
| `energy.service.ts` | `getDailyBaseline(userId)`, `kwhToRM(kwh)`, `getWeekdayTrend(userId)` | MQTT handler, Dashboard route |
| `gamification.service.ts` | `awardEnergyXP()`, `updateStreak()`, `checkAllBadges()`, `getLevelInfo()` | MQTT handler, gamification routes |
| `community.service.ts` | `maybeRefreshLeaderboard()`, `getChallengeProgress()`, `calculateImpact()` | MQTT handler, community routes |
| `anomaly.service.ts` | `check(plugId, currentWatts)` — full 14-day baseline algorithm | MQTT handler |
| `nudge.service.ts` | `generateNudges(userId)` → Nudge[] | insights routes |
| `chat.service.ts` | `parseIntent(message)` → intent, `executeIntent(userId, intent)` → response | chat routes |
| `auth.service.ts` | `registerUser()`, `loginUser()`, `getUserProfile()` | auth routes |

---

## PART 8 — DIRECTORY STRUCTURE AND FILES TO CREATE

```
backend/
├── package.json              # dependencies: express, @supabase/supabase-js, mqtt, cors, dotenv, helmet
├── tsconfig.json             # TypeScript config: target ES2022, module commonjs, outDir dist
├── .env.example              # env template (see PART 2)
│
├── supabase/
│   ├── schema.sql            # Complete DDL — all 13 objects + RLS policies (PART 4)
│   └── seed_badges.sql       # 12 badge INSERTs (PART 4)
│
└── src/
    ├── index.ts              # Entry point
    │   - Load dotenv config
    │   - Import and start MqttService (this starts the telemetry pipeline)
    │   - Import app from app.ts
    │   - app.listen(env.PORT, () => console.log('Server running on port', env.PORT))
    │
    ├── app.ts                # Express app factory
    │   - Create Express app
    │   - Apply middleware: cors(), helmet(), express.json()
    │   - Mount all route files under /api/*
    │   - Global error handler
    │   - Export app
    │
    ├── config/
    │   ├── supabase.ts       # supabaseAdmin (service_role) + createUserClient(jwt) (PART 6)
    │   └── env.ts            # Load and validate env vars, export typed env object
    │
    ├── middleware/
    │   └── auth.ts           # authMiddleware — extracts userId from JWT (PART 6)
    │
    ├── routes/
    │   ├── auth.routes.ts         # POST /register, POST /login, GET /me
    │   ├── plugs.routes.ts        # CRUD + toggle + readings + schedules + mass-on/off
    │   ├── gamification.routes.ts # profile, badges, credits, donate
    │   ├── community.routes.ts    # leaderboard, challenges, impact
    │   ├── insights.routes.ts     # anomalies + nudges + savings
    │   ├── chat.routes.ts         # POST /chat — regex intent parser
    │   └── settings.routes.ts     # profile, notifications, privacy
    │
    ├── services/
    │   ├── mqtt.service.ts        # THE CORE — connect, subscribe, 8-step pipeline
    │   ├── energy.service.ts      # baseline, kWh→RM, trend comparison
    │   ├── gamification.service.ts # XP awards, level-up, badge checks, credit calc
    │   ├── community.service.ts   # leaderboard refresh, challenge progress, impact calc
    │   ├── anomaly.service.ts     # 14-day baseline, 35% deviation, 24h debounce
    │   ├── nudge.service.ts       # generateNudges() — AC 4hr, vampire, weekly record
    │   ├── chat.service.ts        # parseIntent() + executeIntent()
    │   └── auth.service.ts        # registerUser(), loginUser(), getUserProfile()
    │
    └── models/
        ├── types.ts               # TypeScript interfaces for all entities
        └── constants.ts           # LEVEL_THRESHOLDS, LEVEL_TITLES, XP_RATES, BADGE_DEFS
```

---

## PART 9 — BUILD ORDER

Build files in this exact order. Each step depends on the previous.

```
PHASE 1: PROJECT SCAFFOLD
  1. package.json          — npm init with dependencies
  2. tsconfig.json          — TypeScript configuration
  3. .env.example           — environment variable template
  4. src/config/env.ts      — typed env loader
  5. src/config/supabase.ts — both Supabase clients

PHASE 2: MODELS
  6. src/models/types.ts    — TypeScript interfaces
  7. src/models/constants.ts — all hardcoded values

PHASE 3: DATABASE
  8. supabase/schema.sql    — full DDL
  9. supabase/seed_badges.sql — badge definitions

PHASE 4: SERVICES (build bottom-up)
  10. src/services/anomaly.service.ts
  11. src/services/energy.service.ts
  12. src/services/gamification.service.ts
  13. src/services/community.service.ts
  14. src/services/nudge.service.ts
  15. src/services/chat.service.ts
  16. src/services/auth.service.ts
  17. src/services/mqtt.service.ts    ← THE CORE, depends on all above

PHASE 5: MIDDLEWARE & ROUTES
  18. src/middleware/auth.ts
  19. src/routes/auth.routes.ts
  20. src/routes/plugs.routes.ts
  21. src/routes/gamification.routes.ts
  22. src/routes/community.routes.ts
  23. src/routes/insights.routes.ts
  24. src/routes/chat.routes.ts
  25. src/routes/settings.routes.ts

PHASE 6: APP ENTRY
  26. src/app.ts            — Express app with all middleware + routes
  27. src/index.ts          — load env, start MQTT, start Express
```

---

## PART 10 — CRITICAL IMPLEMENTATION RULES

1. **TWO Supabase clients.** Use `supabaseAdmin` (service_role) for: energy_readings inserts, anomaly inserts, badge unlocks, leaderboard refresh, MQTT handler. Use a per-request JWT client for: all route handlers that query user-owned data through RLS.

2. **MQTT service is a singleton.** Instantiate ONCE in `index.ts`. Pass it to routes that need toggle/setPower. Do NOT create multiple MQTT connections.

3. **Leaderboard refresh is expensive.** Use a last-refreshed timestamp. Skip if refreshed within last 5 minutes. Force refresh after every donation. Use `supabaseAdmin.query('REFRESH MATERIALIZED VIEW public.leaderboard')`.

4. **TimescaleDB is optional.** Wrap `create_hypertable` in a DO block. If TimescaleDB is not available (Supabase free tier might not have it), the table still works as a regular PostgreSQL table.

5. **All BM strings must match EXACTLY.** The Flutter app displays these directly. Do not translate on your own — use the strings provided in this document.

6. **Never expose the service_role key** to the client. The `SUPABASE_ANON_KEY` is for the Flutter SDK — the `SUPABASE_SERVICE_KEY` stays on the backend.

7. **Error handling.** Every route should have try/catch. Return `{ error: string }` with appropriate HTTP status codes (400, 401, 404, 500).

8. **Auth on every route except POST /register and POST /login.** Use the auth middleware.

9. **dev script.** Add to package.json: `"dev": "ts-node src/index.ts"` and `"build": "tsc"`.

10. **The backend must start even if MQTT broker is unreachable.** Log a warning, retry connection every 30 seconds, but keep the Express server running so the Flutter app can still query existing data.

---

## PART 11 — VERIFICATION CHECKLIST

After building, verify:
- [ ] `npm run dev` starts without errors
- [ ] MQTT connects and logs "MQTT connected to mqtt://localhost:1883"
- [ ] `POST /api/auth/register` creates user + profile + user_levels + notification_settings
- [ ] `POST /api/auth/login` returns JWT
- [ ] `GET /api/plugs` returns rooms grouped with DeviceCards (empty for new user)
- [ ] `POST /api/plugs` registers a new plug
- [ ] `POST /api/plugs/:id/toggle` publishes MQTT message
- [ ] `POST /api/plugs/mass-off` publishes OFF to all non-always-on plugs
- [ ] `GET /api/gamification/profile` returns level/XP/streak data
- [ ] `GET /api/gamification/badges` returns 12 badges (0 earned, 12 locked for new user)
- [ ] `POST /api/gamification/donate` inserts donation + awards XP
- [ ] `GET /api/community/leaderboard` returns ranked entries
- [ ] `GET /api/community/impact` returns aggregate stats
- [ ] `GET /api/insights/anomalies` returns anomalies + nudges + savings
- [ ] `POST /api/chat { message: "turn off tv" }` returns natural language response
- [ ] MQTT telemetry handler processes readings through full 8-step pipeline
- [ ] Anomaly detection fires when deviation ≥35% and is debounced within 24h
- [ ] Badge unlocks trigger when conditions are met
- [ ] Leaderboard refreshes after donation

---

**Build every file. Write complete, working TypeScript code. No pseudo-code or placeholders. Begin with Phase 1 and work through Phase 6 in order.**
