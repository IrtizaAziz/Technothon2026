# Backend Implementation Plan — Smart Plug Power Management Ecosystem

> **Project:** UM Technothon 2026  
> **Demo:** Pre-recorded video  
> **Stack:** Node.js + Express + TypeScript + Supabase (PostgreSQL, free tier)  
> **MQTT:** Mosquitto broker ← Sonoff S31 (Tasmota)

---

## 1. Architecture — How Data Flows From Plug To Screen

```
┌───────────────────────────────────────────────────────────────────────────┐
│                        SMART PLUG → APP DATA FLOW                         │
│                                                                           │
│  Sonoff S31 (Tasmota)                                                     │
│  │  tele/smartplug1/SENSOR   {"ENERGY":{"Power":160,"Voltage":240,...}}   │
│  │  tele/smartplug2/SENSOR   (every 60s per plug)                         │
│  ▼                                                                        │
│  Mosquitto MQTT Broker  ──── mqtt.js subscriber ────┐                    │
│  (localhost:1883)                                    │                    │
│                                                      ▼                    │
│                                          Express Backend (:3000)          │
│                                          │  parse JSON                    │
│                                          │  INSERT → energy_readings      │
│                                          │  UPDATE plugs cache            │
│                                          │  check anomalies               │
│                                          │  award XP + badge checks       │
│                                          │  refresh leaderboard           │
│                                          ▼                                │
│                                    Supabase PostgreSQL                     │
│                                          │                                │
│                                          ▼                                │
│                                    Flutter App                             │
│                               REST API fetch + Realtime sub               │
└───────────────────────────────────────────────────────────────────────────┘
```

**The backend is the bridge.** Plug → MQTT → Express → Supabase → Flutter. Every 60 seconds a plug reports, the backend processes it through a pipeline that updates power readings, detects anomalies, awards gamification XP, and refreshes the community leaderboard.

---

## 2. Frontend Screen → Backend API Mapping

Each Flutter screen currently uses hardcoded Dart mock data. The mapping below replaces every mock with a real API call.

| Flutter Screen | What It Needs | API Call | Data Source |
|---|---|---|---|
| **Dashboard** (`/dashboard`) | Summary card (total W, daily RM, trend), rooms with DeviceCards | `GET /api/plugs` | `plugs` table (cached latest) |
| **Plug Detail** (`/plug-detail/:id`) | PowerGauge (live W), Stat Row (V/A/kWh/RM), Consumption Chart (7d/30d), Budget Bar, Schedules, Vampire/Always-On toggles, ON/OFF button | `GET /api/plugs/:id` + `GET /api/plugs/:id/readings?range=7d` | `plugs` + `energy_readings` hypertable |
| **AI Insights** (`/insights`) | AnomalyCards (critical/warning), NudgeCards (suggestions), Savings Summary (circular progress) | `GET /api/insights/anomalies` | `anomalies` table + nudge rules |
| **Community Grid** (`/community`) | StreakBadgeRow, Credits Card, Leaderboard, NeighborhoodChallengeCard, Impact Card, Donation Modal | `GET /api/gamification/profile` + `GET /api/community/leaderboard` + `GET /api/community/challenges` + `GET /api/community/impact` | `user_levels`, `leaderboard` MV, `challenges`, aggregations |
| **Badge Collection** (`/badges`) | Level/XP bar, earned badges grid (3-col), locked badges grid (3-col, greyed) | `GET /api/gamification/profile` + `GET /api/gamification/badges` | `user_levels`, `badges`, `user_badges` |
| **Settings** (`/settings`) | Profile card (name/email/address/level/XP), Plugs Saya list, Sentiasa-On list, Data Sharing tier, Notification toggles | `GET /api/settings` + `GET /api/plugs` | `profiles`, `plugs`, `notification_settings` |
| **AI Chat** (modal) | Intent parsing, natural language responses | `POST /api/chat` | Chat service + live MQTT/DB |
| **Toggle Plug** (any screen) | ON/OFF command | `POST /api/plugs/:id/toggle` | → MQTT `cmnd/{topic}/POWER TOGGLE` |
| **Leaving Home** (dashboard chip) | Mass-OFF all non-always-on plugs | `POST /api/plugs/mass-off` | → MQTT `cmnd/{topic}/POWER OFF` each |
| **Donate** (community modal) | Credit slider, recipient selection, confirm | `POST /api/gamification/donate` | `donations` table + XP award |

**Realtime updates:** Supabase Realtime subscriptions push live `energy_readings` INSERTS and `plugs` UPDATES to the Flutter app so the Dashboard and Plug Detail screens update without polling.

---

## 3. Technology Stack

| Layer | Tech | Purpose | Free Tier |
|-------|------|---------|-----------|
| **Runtime** | Node.js 22 + Express 4 + TypeScript 5 | API server | — |
| **Database** | Supabase (PostgreSQL 15) | Auth, data, realtime | 500 MB, 50k MAU |
| **MQTT Client** | `mqtt.js` v6 | Subscribe/publish to Mosquitto | — |
| **MQTT Broker** | Mosquitto (local) | Plug ↔ App communication | Local = free |
| **Auth** | Supabase Auth | JWT-based, email/password | Unlimited |
| **Realtime** | Supabase Realtime | Push DB changes → Flutter | 200 concurrent |
| **Hosting** | Render / Railway | Backend deploy | 750 hrs/month |

---

## 4. Supabase Database Schema

### 4.1 Tables Overview

```
auth.users ──1:1──► profiles ──1:N──► plugs ──1:N──► energy_readings (hypertable)
    │                    │                │
    │                    │                ├──1:N──► schedules
    │                    │                └──1:N──► anomalies
    │                    │
    │                    ├──1:N──► user_badges ──N:1──► badges
    │                    ├──1:1──► user_levels
    │                    ├──1:N──► donations
    │                    ├──1:1──► notification_settings
    │
    └── (aggregated) ──► challenges + challenge_participants
                         leaderboard (materialized view)
```

### 4.2 All Tables (DDL)

```sql
-- ============================================================
-- 1. profiles — extends auth.users
-- App_Claude §8 Settings: name, email, address, kawasan
-- GK_Gamification_KL §4: league assignment
-- ============================================================
CREATE TABLE public.profiles (
  id              UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name       TEXT NOT NULL,
  email           TEXT NOT NULL,
  address         TEXT DEFAULT '',           -- "12, Jalan Bahagia, TTDI"
  kawasan         TEXT NOT NULL,             -- "TTDI", "Bangsar", "PPR Kerinchi"
  league          TEXT NOT NULL,             -- 'condo' | 'taman' | 'ppr'
  privacy_tier    SMALLINT DEFAULT 1,        -- 1=Default, 2=Eco Mode, 3=Grid Mode
  avatar_url      TEXT DEFAULT '',
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================
-- 2. plugs — registered smart plugs
-- App_Claude §3 Dashboard: DeviceCard (icon, name, watts, status dot)
-- App_Claude §4 Plug Detail: PowerGauge, Stat Row, Budget, Vampire, Always-On
-- App_Claude §5 Pairing: register new plug after BLE pairing
-- S31_Tasmota §5: mqtt_topic = "smartplug1", "smartplug2"
-- ============================================================
CREATE TABLE public.plugs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  mqtt_topic      TEXT NOT NULL UNIQUE,      -- Tasmota topic: "smartplug1"
  name            TEXT NOT NULL,             -- "AC Bedroom", "TV"
  room            TEXT NOT NULL,             -- "Living Room", "Kitchen"
  appliance_icon  TEXT DEFAULT '🔌',         -- emoji: ❄️ 🖥️ 🧊 🍚
  appliance_type  TEXT DEFAULT '',           -- 'ac', 'tv', 'fridge', 'rice_cooker'
  is_on           BOOLEAN DEFAULT FALSE,     -- relay state (from stat/+/POWER)
  is_online       BOOLEAN DEFAULT FALSE,     -- MQTT connected?
  is_always_on    BOOLEAN DEFAULT FALSE,     -- exclude from "Leaving Home"
  vampire_auto_off BOOLEAN DEFAULT TRUE,     -- auto-cut standby <5W
  monthly_budget_rm DECIMAL(10,2) DEFAULT NULL,
  current_power_w  REAL DEFAULT 0,           -- latest cached wattage
  current_voltage_v REAL DEFAULT 0,
  current_current_a REAL DEFAULT 0,
  energy_today_kwh REAL DEFAULT 0,           -- from Tasmota "Today"
  energy_month_kwh REAL DEFAULT 0,
  anomaly_status   TEXT DEFAULT NULL,        -- null | 'warning' | 'critical'
  created_at       TIMESTAMPTZ DEFAULT NOW(),
  updated_at       TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_plugs_user ON public.plugs(user_id);

-- ============================================================
-- 3. energy_readings — time-series telemetry (hypertable)
-- S31_Tasmota §7: tele/{id}/SENSOR JSON → Power, Voltage, Current, etc.
-- App_Claude §4: Consumption Chart 7d/30d, Stat Row (V, A, kWh)
-- 1 reading every 60s per plug → ~1,440/day/plug
-- ============================================================
CREATE TABLE public.energy_readings (
  id              BIGSERIAL PRIMARY KEY,
  plug_id         UUID NOT NULL REFERENCES public.plugs(id) ON DELETE CASCADE,
  timestamp       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  power_w         REAL NOT NULL,             -- Active Power
  apparent_power_w REAL DEFAULT 0,
  reactive_power_w REAL DEFAULT 0,
  voltage_v       REAL DEFAULT 0,
  current_a       REAL DEFAULT 0,
  power_factor    REAL DEFAULT 0,
  energy_kwh      REAL DEFAULT 0,            -- Total cumulative
  energy_today_kwh REAL DEFAULT 0,           -- Reset daily
  energy_yesterday_kwh REAL DEFAULT 0,
  relay_state     BOOLEAN DEFAULT TRUE
);

-- TimescaleDB hypertable (makes 7d/30d queries sub-10ms)
SELECT create_hypertable('energy_readings', 'timestamp',
  chunk_time_interval => INTERVAL '1 day');

CREATE INDEX idx_readings_plug_time
  ON public.energy_readings(plug_id, timestamp DESC);

-- ============================================================
-- 4. badges — 12 badge definitions (static reference)
-- GK_Gamification_KL §6: 4 earned + 8 locked
-- App_Claude §7.5: Badge Collection screen, 3-column grid
-- ============================================================
CREATE TABLE public.badges (
  id              TEXT PRIMARY KEY,           -- 'streak', 'jiran', 'bulan', etc.
  name_bm         TEXT NOT NULL,             -- "7-Hari Rantaian"
  name_en         TEXT NOT NULL,             -- "7-Day Streak"
  emoji           TEXT NOT NULL,             -- 🔥 🛡️ 🌙 🍃
  unlock_condition_bm TEXT NOT NULL,
  unlock_condition_en TEXT NOT NULL,
  category        TEXT DEFAULT 'general',    -- 'general' | 'festival' | 'league'
  sort_order      INT DEFAULT 0
);

-- ============================================================
-- 5. user_badges — which badges each user earned
-- ============================================================
CREATE TABLE public.user_badges (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  badge_id        TEXT NOT NULL REFERENCES public.badges(id) ON DELETE CASCADE,
  unlocked_at     TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, badge_id)
);
CREATE INDEX idx_user_badges ON public.user_badges(user_id);

-- ============================================================
-- 6. user_levels — XP, level, streak
-- GK_Gamification_KL §5: 7 levels, XP per level
-- App_Claude §7: streak badge, §8 profile: level/XP bar
-- ============================================================
CREATE TABLE public.user_levels (
  user_id         UUID PRIMARY KEY REFERENCES public.profiles(id) ON DELETE CASCADE,
  level           INT DEFAULT 1,             -- 1-7
  current_xp      INT DEFAULT 0,
  streak_days     INT DEFAULT 0,
  last_active_date DATE DEFAULT NULL,        -- for streak continuity
  monthly_kwh_saved REAL DEFAULT 0,
  total_credits_earned INT DEFAULT 0,
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================
-- 7. donations — credit donation ledger
-- App_Claude §7: Donation Modal, Leaderboard, Impact Card
-- ============================================================
CREATE TABLE public.donations (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_user_id    UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  to_user_id      UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  to_pool         TEXT DEFAULT NULL,         -- 'tabung_komuniti' | 'PPR Kerinchi'
  credits         INT NOT NULL,
  amount_rm       DECIMAL(10,2) NOT NULL,    -- credits × 0.218
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_donations_from ON public.donations(from_user_id);

-- ============================================================
-- 8. challenges — neighborhood challenges (1 per month)
-- App_Claude §7: NeighborhoodChallengeCard
-- GK_Gamification_KL §7: Festival Challenge Calendar (12 months)
-- ============================================================
CREATE TABLE public.challenges (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            TEXT NOT NULL,             -- "Haze Shield", "Merdeka 55"
  month           TEXT NOT NULL,             -- "Julai", "Ogos"
  kawasan_a       TEXT NOT NULL,             -- "TTDI"
  kawasan_b       TEXT NOT NULL,             -- "Damansara"
  target_kwh      REAL NOT NULL,             -- 1200
  start_date      DATE NOT NULL,
  end_date        DATE NOT NULL,
  winner_kawasan  TEXT DEFAULT NULL,
  status          TEXT DEFAULT 'active',     -- 'active' | 'ended'
  reward_badge_id  TEXT REFERENCES public.badges(id)
);

-- ============================================================
-- 9. challenge_participants — per-kawasan progress
-- ============================================================
CREATE TABLE public.challenge_participants (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  challenge_id    UUID NOT NULL REFERENCES public.challenges(id) ON DELETE CASCADE,
  kawasan         TEXT NOT NULL,
  homes_count     INT DEFAULT 0,
  kwh_saved       REAL DEFAULT 0,
  top_donor_name   TEXT DEFAULT '',
  top_donor_credits INT DEFAULT 0,
  streak_days     INT DEFAULT 0,
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(challenge_id, kawasan)
);

-- ============================================================
-- 10. schedules — per-plug time schedules
-- App_Claude §4: Schedule list with ON label, swipe-to-delete
-- ============================================================
CREATE TABLE public.schedules (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  plug_id         UUID NOT NULL REFERENCES public.plugs(id) ON DELETE CASCADE,
  name            TEXT DEFAULT '',           -- "Weekdays", "Weekend"
  days_of_week    SMALLINT NOT NULL,         -- bitmask: 1=Mon ... 64=Sun
  time_on         TIME NOT NULL,
  time_off        TIME NOT NULL,
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_schedules_plug ON public.schedules(plug_id);

-- ============================================================
-- 11. anomalies — detected appliance issues
-- App_Claude §6: AnomalyCard (critical red / warning amber)
-- Technothon_Project_Architecture §3B: Predictive anomaly detection
-- ============================================================
CREATE TABLE public.anomalies (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  plug_id         UUID NOT NULL REFERENCES public.plugs(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  severity        TEXT NOT NULL,             -- 'critical' | 'warning'
  title_bm        TEXT NOT NULL,
  title_en        TEXT NOT NULL,
  description_bm  TEXT NOT NULL,
  description_en  TEXT NOT NULL,
  impact_rm_per_month REAL DEFAULT 0,
  is_read         BOOLEAN DEFAULT FALSE,
  is_dismissed    BOOLEAN DEFAULT FALSE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_anomalies_user ON public.anomalies(user_id);

-- ============================================================
-- 12. notification_settings — per-user toggle prefs
-- App_Claude §8: Notification toggles
-- ============================================================
CREATE TABLE public.notification_settings (
  user_id         UUID PRIMARY KEY REFERENCES public.profiles(id) ON DELETE CASCADE,
  all_enabled     BOOLEAN DEFAULT TRUE,
  anomalies       BOOLEAN DEFAULT TRUE,
  budget          BOOLEAN DEFAULT TRUE,
  tips            BOOLEAN DEFAULT TRUE,
  community       BOOLEAN DEFAULT TRUE,
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================
-- 13. leaderboard — materialized view (by league)
-- App_Claude §7: "🏆 Penderma Teratas — Julai"
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
LEFT JOIN public.donations d ON d.from_user_id = p.id
  AND d.created_at >= date_trunc('month', CURRENT_DATE)
GROUP BY p.id, p.full_name, p.kawasan, p.league
ORDER BY p.league, credits_donated DESC;

-- Refresh: after each donation, hourly, or on-demand if stale > 5 min
-- REFRESH MATERIALIZED VIEW public.leaderboard;
```

### 4.3 RLS Policies

```sql
-- Profiles: user owns their own
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own profile" ON public.profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "update own"   ON public.profiles FOR UPDATE USING (auth.uid() = id);

-- Plugs: user owns their own; backend uses service_role to bypass
ALTER TABLE public.plugs ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own plugs" ON public.plugs FOR ALL USING (auth.uid() = user_id);

-- Energy readings: no direct client access (backend only via service_role)
ALTER TABLE public.energy_readings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "backend only" ON public.energy_readings FOR SELECT USING (true);

-- Badges: publicly readable
ALTER TABLE public.badges ENABLE ROW LEVEL SECURITY;
CREATE POLICY "public read" ON public.badges FOR SELECT USING (true);

-- User badges: own only
ALTER TABLE public.user_badges ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own badges" ON public.user_badges FOR SELECT USING (auth.uid() = user_id);

-- Donations: own + pool donations are public
ALTER TABLE public.donations ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own donations" ON public.donations FOR SELECT USING (auth.uid() = from_user_id);
CREATE POLICY "pool public"   ON public.donations FOR SELECT USING (to_pool IS NOT NULL);

-- Notifications: own only
ALTER TABLE public.notification_settings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "own notifs" ON public.notification_settings FOR ALL USING (auth.uid() = user_id);
```

---

## 5. MQTT Integration — Fetching Data From Smart Plugs

### 5.1 MQTT Topic Structure (from S31_Tasmota_Flashing_Guide.md)

| Topic | Direction | Payload | Frequency |
|-------|-----------|---------|-----------|
| `tele/smartplug1/SENSOR` | Plug → Broker | `{"ENERGY":{...}}` JSON | Every 60s |
| `stat/smartplug1/POWER` | Plug → Broker | `{"POWER":"ON"}` | On state change |
| `cmnd/smartplug1/POWER` | App → Broker → Plug | `ON` / `OFF` / `TOGGLE` | On demand |

Tasmota's `FullTopic` is set to `%topic%/%prefix%/`, so with `Topic smartplug1`:

```
tele/smartplug1/SENSOR   ← sensor readings
stat/smartplug1/POWER    ← relay confirmations
cmnd/smartplug1/POWER    ← commands
```

### 5.2 Sensor Payload (what the plug sends)

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

### 5.3 MQTT Service — The Heartbeat of the Backend

```typescript
// src/services/mqtt.service.ts
// Runs on startup, never stops. Connects to Mosquitto, subscribes,
// and processes every telemetry message through the full pipeline.

class MqttService {
  private client: mqtt.MqttClient;

  async start() {
    // 1. Connect: mqtt.connect('mqtt://localhost:1883')
    // 2. Subscribe: 'tele/+/SENSOR'    — all plugs
    // 3. Subscribe: 'stat/+/POWER'     — relay state changes

    this.client.on('message', async (topic, payload) => {
      const segments = topic.split('/');       // ['tele', 'smartplug1', 'SENSOR']
      const plugTopic = segments[1];           // 'smartplug1'

      if (topic.includes('/SENSOR')) {
        await this.handleTelemetry(plugTopic, JSON.parse(payload));
      } else if (topic.includes('/POWER')) {
        const { POWER } = JSON.parse(payload);
        await this.handleRelayState(plugTopic, POWER === 'ON');
      }
    });
  }

  async handleTelemetry(plugTopic: string, data: any) {
    const e = data.ENERGY;

    // a) Find plug by mqtt_topic
    const plug = await supabase.from('plugs')
      .select('id, user_id, monthly_budget_rm')
      .eq('mqtt_topic', plugTopic).single();

    if (!plug) return; // unregistered plug — skip

    // b) INSERT into energy_readings
    await supabase.from('energy_readings').insert({
      plug_id: plug.id,
      power_w: e.Power,
      voltage_v: e.Voltage,
      current_a: e.Current,
      power_factor: e.PowerFactor,
      energy_kwh: e.Total,
      energy_today_kwh: e.Today,
      energy_yesterday_kwh: e.Yesterday,
    });

    // c) UPDATE plugs cache (flatten latest for fast dashboard reads)
    await supabase.from('plugs').update({
      current_power_w: e.Power,
      current_voltage_v: e.Voltage,
      current_current_a: e.Current,
      energy_today_kwh: e.Today,
      is_online: true,
      updated_at: new Date(),
    }).eq('id', plug.id);

    // d) Anomaly check
    await anomalyService.check(plug.id, e.Power);

    // e) XP award
    await gamificationService.awardEnergyXP(plug.user_id, plug.id, e.Today);

    // f) Streak update
    await gamificationService.updateStreak(plug.user_id);

    // g) Badge checks
    await gamificationService.checkBadges(plug.user_id);

    // h) Leaderboard refresh (debounced — max once per 5 min)
    await communityService.maybeRefreshLeaderboard();
  }

  async handleRelayState(plugTopic: string, isOn: boolean) {
    await supabase.from('plugs').update({ is_on: isOn })
      .eq('mqtt_topic', plugTopic);
  }

  async togglePlug(plugTopic: string) {
    // Publish: cmnd/{plugTopic}/POWER TOGGLE
    this.client.publish(`cmnd/${plugTopic}/POWER`, 'TOGGLE', { qos: 1 });
  }

  async setPlugPower(plugTopic: string, state: 'ON' | 'OFF') {
    this.client.publish(`cmnd/${plugTopic}/POWER`, state, { qos: 1 });
  }
}
```

### 5.4 Processing Pipeline (per reading)

```
tele/+/SENSOR (every 60s)
    │
    ▼
1. PARSE JSON        → Extract Power, Voltage, Current, Today kWh
    ▼
2. STORE READING     → INSERT energy_readings (hypertable, auto-chunked daily)
    ▼
3. CACHE ON plugs    → UPDATE plugs SET current_power_w, voltage, online=true
    ▼
4. CHECK ANOMALY     → Compare 14-day avg vs current. If >35% deviation → INSERT anomaly
    ▼
5. AWARD XP          → kWh_below_baseline × 10 XP. UPDATE user_levels
    ▼
6. UPDATE STREAK     → If today < baseline and yesterday was active → streak++
    ▼
7. CHECK BADGES      → Run all 12 badge conditions. New unlocks → INSERT user_badges
    ▼
8. REFRESH LEADERBOARD → REFRESH MATERIALIZED VIEW (debounced)
```

### 5.5 Running MQTT Locally

```bash
# Windows (already set up per Architecture §7D)
mosquitto -c C:\mosquitto\mosquitto.conf -v

# Verify: listen for all topics
mosquitto_sub -t "#" -v
```

---

## 6. Express API — Every Endpoint

### 6.1 Route Tree

```
POST   /api/auth/register               # Supabase Auth signup
POST   /api/auth/login                  # Supabase Auth signin
GET    /api/auth/me                     # Current user profile

GET    /api/plugs                       # Dashboard: summary + rooms + DeviceCards
GET    /api/plugs/:id                   # Plug Detail: PowerGauge, Stat Row, Budget
GET    /api/plugs/:id/readings?range=7d # Consumption Chart: hourly time-series
POST   /api/plugs                       # Register new plug after BLE pairing
PATCH  /api/plugs/:id                   # Update name, room, icon, budget
POST   /api/plugs/:id/toggle            # → MQTT cmnd/{topic}/POWER TOGGLE
DELETE /api/plugs/:id                   # Remove plug
GET    /api/plugs/:id/schedules          # List schedules
POST   /api/plugs/:id/schedules          # Create schedule
PATCH  /api/plugs/schedules/:sid        # Update schedule
DELETE /api/plugs/schedules/:sid        # Delete schedule
POST   /api/plugs/mass-off              # "Leaving Home" → OFF all non-always-on
POST   /api/plugs/mass-on               # "All On"

GET    /api/gamification/profile        # Level, XP, streak, title, credits
GET    /api/gamification/badges          # Earned + locked badge list
GET    /api/gamification/credits         # Balance + transaction history
POST   /api/gamification/donate          # Donate credits → leaderboard + XP

GET    /api/community/leaderboard?league=taman   # By league, by month
GET    /api/community/challenges                  # Active challenge for user's kawasan
GET    /api/community/challenges/:id              # Challenge detail + progress
GET    /api/community/impact                      # Aggregate KL community stats

GET    /api/insights/anomalies          # Active anomalies + nudges + savings

POST   /api/chat                        # AI Chat: intent parse → response

GET    /api/settings                    # Profile + notif toggles + privacy tier
PATCH  /api/settings/profile            # Update name, address
PATCH  /api/settings/notifications      # Update notification toggles
PATCH  /api/settings/privacy            # Update privacy tier (1/2/3)
```

### 6.2 Response Shapes

#### `GET /api/plugs` → Dashboard

```json
{
  "summary": {
    "total_watts": 420,
    "daily_cost_rm": 1.37,
    "trend_vs_yesterday_pct": 12,
    "plug_count": 4,
    "online_count": 4
  },
  "rooms": [
    {
      "name": "Living Room",
      "total_watts": 180,
      "plugs": [
        {
          "id": "uuid", "name": "TV", "icon": "🖥️", "watts": 12,
          "status": "online_normal", "is_on": true
        }
      ]
    }
  ]
}
```

DeviceCard status values map to App_Claude §3 design:
- `online_normal` → green dot (≤200W)
- `online_high` → amber dot (200–800W)
- `online_critical` → red dot (>800W)
- `offline` → grey dot, dimmed

#### `GET /api/plugs/:id` → Plug Detail

```json
{
  "id": "uuid", "name": "AC Bedroom", "room": "Living Room",
  "icon": "❄️", "appliance_type": "ac",
  "is_on": true, "is_online": true, "is_always_on": false, "vampire_auto_off": true,
  "latest": {
    "power_w": 160, "voltage_v": 240, "current_a": 0.67,
    "energy_today_kwh": 2.4, "energy_month_kwh": 48,
    "daily_cost_rm": 0.35, "monthly_cost_rm": 10.50
  },
  "budget": {
    "limit_rm": 60, "spent_rm": 48, "progress_pct": 80,
    "days_remaining": 12, "status": "warning"
  },
  "anomaly": null
}
```

Budget status mapping (App_Claude §4):
- `green` → spent ≤70%
- `warning` → spent 70–90%, amber
- `exceeded` → spent >90%, red

#### `GET /api/plugs/:id/readings?range=7d` → Consumption Chart

```json
{
  "range": "7d",
  "points": [
    { "timestamp": "2026-07-14T00:00:00Z", "power_w": 120 },
    { "timestamp": "2026-07-14T01:00:00Z", "power_w": 0 }
  ]
}
```

Query: `AVG(power_w) OVER 1-hour buckets` from `energy_readings`. 7d = 168 points, 30d = 720 points.

#### `GET /api/gamification/profile` → Settings + Badge Collection

```json
{
  "level": 3,
  "title_bm": "Celik Tenaga",      "title_en": "Energy Literate",
  "current_xp": 1840,              "required_xp": 3000,
  "xp_to_next": 560,
  "next_title_bm": "Wira Hijau",   "next_title_en": "Eco Champion",
  "progress_pct": 61,
  "streak_days": 7,
  "credits_available": 14,
  "total_credits_earned": 22,
  "monthly_kwh_saved": 84
}
```

#### `GET /api/gamification/badges` → Badge Collection

```json
{
  "earned": [
    { "id": "streak", "name_bm": "7-Hari Rantaian", "name_en": "7-Day Streak",
      "emoji": "🔥", "unlock_condition_bm": "7 hari berturut-turut bawah baseline",
      "unlocked_at": "2026-07-14T00:00:00Z" }
  ],
  "locked": [
    { "id": "taugeh", "name_bm": "Taugeh Champion", "name_en": "Bean Sprout Champ",
      "emoji": "🌱", "unlock_condition_bm": "Jimat RM 50+ sebulan" }
  ]
}
```

#### `GET /api/community/leaderboard?league=taman`

```json
{
  "league": "taman", "month": "Julai",
  "entries": [
    { "rank": 1, "name": "Kumar", "kawasan": "Bangsar",
      "credits_donated": 22, "credits_to_ppr": 12,
      "ppr_name": "PPR Kerinchi", "is_current_user": false },
    { "rank": 3, "name": "Aisyah", "kawasan": "TTDI",
      "credits_donated": 14, "credits_to_ppr": 6,
      "ppr_name": "PPR Lembah Subang", "is_current_user": true }
  ]
}
```

#### `GET /api/community/challenges`

```json
{
  "active": {
    "id": "uuid", "name": "Haze Shield", "month": "Julai",
    "target_kwh": 1200, "days_remaining": 4, "leading_kawasan": "TTDI",
    "kawasan_a": { "name": "TTDI", "homes": 38, "progress_pct": 72,
      "kwh_saved": 864, "top_donor": "Raj", "top_donor_credits": 12,
      "streak_days": 14 },
    "kawasan_b": { "name": "Damansara", "homes": 29, "progress_pct": 58,
      "kwh_saved": 696, "top_donor": "Mei Ling", "top_donor_credits": 9,
      "streak_days": 8 }
  }
}
```

#### `GET /api/community/impact`

```json
{
  "total_kwh_saved": 1240,
  "total_credits_shared": 84,
  "neighborhoods_count": 4,
  "equivalents": {
    "ppr_units_per_day": 62,
    "ac_units_8hrs": 11,
    "street_lamps_bukit_bintang": 58
  }
}
```

#### `POST /api/gamification/donate`

Request: `{ "recipient_type": "pool", "recipient_ppr": "PPR Kerinchi", "credits": 5 }`

Response:
```json
{
  "success": true, "credits_donated": 5, "amount_rm": 1.09,
  "ppr": "PPR Kerinchi",
  "badge_hint": { "badge_name_bm": "Jiran Terbaik", "remaining": 3, "total": 15 }
}
```

#### `GET /api/insights/anomalies`

```json
{
  "anomalies": [
    {
      "id": "uuid", "plug_name": "Fridge", "plug_icon": "🧊",
      "severity": "critical",
      "title_bm": "Peti sejuk guna 35% lebih tenaga",
      "description_bm": "Peti sejuk anda guna 35% lebih tenaga semalam. Anggaran impak: RM 18/bulan.",
      "impact_rm_per_month": 18.00, "created_at": "2026-07-14T10:30:00Z"
    }
  ],
  "suggestions": [
    {
      "type": "nudge",
      "title_bm": "AC anda telah berjalan selama 4 jam. Buka tingkap?",
      "subtitle_bm": "Suhu luar 26°C — nyaman untuk buka tingkap.",
      "action_type": "open_schedule", "action_plug_id": "uuid"
    }
  ],
  "savings": { "current_rm": 18.40, "goal_rm": 25.00, "progress_pct": 73 }
}
```

---

## 7. Gamification Engine

### 7.1 Level Thresholds & XP Rules

| Level | XP | Title BM | Title EN |
|-------|----|----------|----------|
| 1 | 0 | Budak Baru | New Plug |
| 2 | 500 | Celik Tenaga | Energy Literate |
| 3 | 1,500 | Jimat Cermat | Smart Saver |
| 4 | 3,000 | Wira Hijau | Eco Champion |
| 5 | 6,000 | Pendekar Tenaga | Power Guardian |
| 6 | 12,000 | Jaguh Komuniti | Community Hero |
| 7 | 25,000 | Dato' Jimat | Sir Saves-a-Lot |

XP awards: **10 XP per kWh saved** below baseline · **50 XP per credit donated** (100 XP to PPR) · **200 XP per 7-day streak** · **100 XP per anomaly resolved** · **1,000 XP per challenge won**

### 7.2 Badge Unlock Rules

**Earned (4):**
| id | Name (BM) | Emoji | Unlock |
|----|-----------|-------|--------|
| `streak` | 7-Hari Rantaian | 🔥 | 7 consecutive days below baseline |
| `jiran` | Jiran Terbaik | 🛡️ | 15+ total donations, 5+ to PPR |
| `celik` | Celik Tenaga | 🍃 | Reach Level 3 |
| `bulan` | Anak Bulan Hero | 🌙 | 30 days below baseline during Ramadan |

**Locked (8):**
| id | Name (BM) | Emoji | Unlock |
|----|-----------|-------|--------|
| `taugeh` | Taugeh Champion | 🌱 | Save RM 50+/month |
| `merdeka` | Merdeka Saver | 🇲🇾 | Save 55 kWh in August |
| `kopitiam` | Kopitiam Regular | ☕ | 30 days off-peak savings (12PM–4PM) |
| `mamak` | Mamak Squad | 🫓 | 10 nights saving 8PM–12AM |
| `balik` | Balik Kampung | 🚗 | Auto-shutdown during festive exodus |
| `lrt` | LRT Warrior | 🚆 | 30 days LRT commute savings |
| `ppr` | PPR Champion | 🏅 | Top 3 Rumah Pangsa league |
| `dato` | Dato' Jimat | 👑 | Reach Level 7 |

### 7.3 Credit System

```
credits_available = floor(total_kwh_saved) - sum(donated_credits)
1 credit ≈ RM 0.218 (TNB lifeline tariff)
```

---

## 8. Community Grid Logic

### 8.1 Leaderboard

PostgreSQL materialized view partitioned by league, refreshed after each donation and on-demand when fetched.

### 8.2 Challenge Progress

```
kawasan.progress_pct = (kwh_saved / target_kwh) × 100
kawasan.homes_count = COUNT(DISTINCT profiles WHERE kawasan = X)
kawasan.top_donor = MAX(donor in kawasan this month)
```

### 8.3 Community Impact Equivalents

```
ppr_units = kWh_saved / 20        (1 PPR unit ≈ 20 kWh/day)
ac_8hrs   = kWh_saved / 112       (1 AC × 8hr ≈ 112 kWh)
lamps     = kWh_saved / 21.4      (1 street lamp ≈ 21.4 kWh/month)
```

### 8.4 Donation Flow

```
1. User picks recipient: Tabung Komuniti / PPR Kerinchi / PPR Lembah Subang
2. Slides credit amount (0 → credits_available)
3. Sees impact preview: "~RM X.XX bil elektrik"
4. Confirms → INSERT donation → deduct credits → award XP → refresh leaderboard
```

---

## 9. Anomaly Detection

```typescript
// Each reading: compare current power vs 14-day rolling average
if (Math.abs(current_w - baseline_avg_w) / baseline_avg_w * 100 >= 35) {
  severity = deviation >= 50 ? 'critical' : 'warning';
  // Debounce: skip if anomaly for this plug created within 24 hours
  impact_rm = (current_w - baseline_avg_w) × 24 × 30 / 1000 × 0.218;
  // INSERT anomaly → shows on AI Insights screen as AnomalyCard
}
```

---

## 10. Files & Directory Structure

```
backend/
├── package.json
├── tsconfig.json
├── .env
├── .env.example
│
├── supabase/
│   ├── schema.sql              # All 13 tables + hypertable + materialized view + RLS
│   ├── seed_badges.sql         # 12 badge definitions
│   ├── seed_demo_data.sql      # Aisyah + 4 personas + plugs + 30 days readings
│   └── seed_challenges.sql     # Haze Shield (Julai) challenge
│
└── src/
    ├── index.ts                # Entry: load env, start MQTT, start server
    ├── app.ts                  # Express setup: middleware, routes, error handler
    │
    ├── config/
    │   ├── supabase.ts         # Admin client (service_role key)
    │   └── env.ts              # Env validator
    │
    ├── middleware/
    │   └── auth.ts             # Supabase JWT verification middleware
    │
    ├── routes/
    │   ├── auth.routes.ts      # /api/auth/*
    │   ├── plugs.routes.ts     # /api/plugs/* + schedules + mass-off/on
    │   ├── gamification.routes.ts  # /api/gamification/*
    │   ├── community.routes.ts     # /api/community/*
    │   ├── insights.routes.ts      # /api/insights/*
    │   ├── chat.routes.ts          # /api/chat
    │   └── settings.routes.ts      # /api/settings/*
    │
    ├── services/
    │   ├── mqtt.service.ts         # MQTT connect, subscribe, telemetry handler
    │   ├── energy.service.ts       # Baseline calc, kWh→RM, trends
    │   ├── gamification.service.ts # XP, levels, badges, credits
    │   ├── community.service.ts    # Leaderboard, challenges, impact
    │   ├── anomaly.service.ts      # Statistical anomaly detection
    │   ├── nudge.service.ts        # Behavioral nudge rules
    │   ├── chat.service.ts         # Intent parser + response engine
    │   └── auth.service.ts         # Supabase Auth wrapper
    │
    └── models/
        ├── types.ts                # TypeScript interfaces
        └── constants.ts            # Levels, XP rates, badge defs, leagues
```

---

## 11. Demo Seeding

Pre-populate Supabase with realistic data for the video recording:

| Table | Rows | Content |
|-------|------|---------|
| `profiles` | 5 | Aisyah (TTDI), Kumar (Bangsar), Mei Ling (Cheras), Raj (Damansara Hts), Fatimah (PPR Kerinchi) |
| `plugs` | 4 | AC Bedroom ❄️, TV 🖥️, Fridge 🧊, Rice Cooker 🍚 — all owned by Aisyah |
| `energy_readings` | ~2,880 | 30 days × hourly × 4 plugs with realistic patterns |
| `badges` | 12 | All badge definitions |
| `user_badges` | 4 | Aisyah earned: 🔥 streak, 🛡️ jiran, 🌙 bulan, 🍃 celik |
| `user_levels` | 5 | Aisyah: Level 3, 1840 XP, 7-day streak, 14 credits |
| `donations` | 12 | Leaderboard data: Kumar→22cr, Mei Ling→16cr, Aisyah→14cr, etc. |
| `challenges` | 1 | Haze Shield Julai: TTDI vs Damansara, target 1200 kWh, 4 days left |
| `challenge_participants` | 2 | TTDI 72% (38 homes), Damansara 58% (29 homes) |
| `anomalies` | 1 | Fridge critical: 35% more, RM 18/month |
| `schedules` | 2 | AC Bedroom: Weekdays 8PM–6AM, Weekend 9AM–11PM |

```sql
-- Key demo data: Aisyah's profile matches the exact mock data from the design docs
INSERT INTO profiles (id, full_name, email, address, kawasan, league, privacy_tier)
VALUES ('uid-aisyah', 'Aisyah Binti Rahman', 'aisyah.r@email.com',
        '12, Jalan Bahagia, TTDI', 'TTDI', 'taman', 2);

INSERT INTO user_levels (user_id, level, current_xp, streak_days, total_credits_earned)
VALUES ('uid-aisyah', 3, 1840, 7, 22);
```

---

## 12. Build Order (5 Days)

| Day | Focus | Key Deliverables |
|-----|-------|-----------------|
| **1** | Foundation | `package.json`, `tsconfig.json`, Supabase project, full schema DDL, Express skeleton, Auth routes + middleware |
| **2** | MQTT Pipeline | mqtt.service.ts (subscribe, parse, store), energy.service.ts (baseline, trends), plugs routes (CRUD, toggle, readings, dashboard aggregation) |
| **3** | Gamification | gamification.service.ts (XP, levels, badges, credits), gamification routes (profile, badges, donate) |
| **4** | Community + Anomalies | community.service.ts (leaderboard, challenges, impact), anomaly.service.ts, nudge.service.ts, chat.service.ts, all remaining routes |
| **5** | Seed + Integrate | Run all seed SQL, Flutter app replaces mock data with API calls, test full flow, record video |

---

## 13. Environment Variables

```env
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOi...       # public, for Flutter SDK
SUPABASE_SERVICE_KEY=eyJhbGciOi...    # secret, for backend admin ops

MQTT_BROKER_URL=mqtt://localhost
MQTT_BROKER_PORT=1883

PORT=3000
NODE_ENV=development
TNB_TARIFF_RATE=0.218
```

---

*Aligned with: Technothon_Project_Architecture.md, App_Claude_Design_Prompt.md, GK_Gamification_KL_Full.md, GK_Gamification_KL_Design_Updates.md, S31_Tasmota_Flashing_Guide.md*
