# Feature: Creator Studio Analytics Engine & Telemetry

> **Document Type:** System Architecture & Open-Source Implementation Blueprint  
> **Feature Identifier:** Tier 3 — Big Data Aggregation & Creator Telemetry Engine (Feature 19)  
> **Bounded Context:** High-Throughput Event Ingestion, Second-by-Second Video Retention Curves, Channel Growth Telemetry, Rollup Aggregations & Creator Studio Dashboard  
> **Primary Target:** Open-Source Contributors, Data Engineers, Backend Architects, Frontend Data Viz Engineers  
> **Status:** Architecture Design & Implementation Guidance (Read-Only Blueprint)  

---

## 1. Feature Overview

In **RKSkillTube**, content creators require granular, actionable operational telemetry to understand how their educational content performs, where student attention drops off, which topics drive subscriptions, and how revenue is generated. Delivering these insights without compromising video playback latency or overwhelming primary database transactional throughput requires a decoupled telemetry ingestion and rollup pipeline.

The **Creator Studio Analytics Engine & Telemetry** system delivers an enterprise-grade analytics platform consisting of four synchronized subsystems:

1. **High-Throughput Telemetry Ingestion (Heartbeat Beacons):**
   - Video player client (`HlsPlayer.tsx` / `useVideoAnalytics.ts`) emits lightweight heartbeat beacons every 10 seconds of active playback, alongside discrete events (`play`, `pause`, `seek`, `ended`, `quality_change`).
   - Ingestion API endpoint (`POST /api/v1/telemetry/playback-event`) buffers raw events directly into **Redis Streams** (`telemetry:playback_events`), bypassing transactional database writes and returning `HTTP 204 No Content` in $< 5\text{ms}$.

2. **Asynchronous Stream Aggregation & Daily Rollup Workers:**
   - Celery stream consumer workers process batches from Redis:
     - Applies view verification rules (filtering out bounces $< 30$ seconds).
     - Aggregates watch time and builds normalized audience retention histograms (percentiles from 0% to 100% of video duration in 1% buckets).
     - Writes pre-aggregated daily summaries into PostgreSQL time-series tables (`video_daily_metrics`, `channel_daily_metrics`, `traffic_source_metrics`, `video_retention_curves`).

3. **Creator Studio Dashboard APIs & Data Visualization:**
   - Powers the Creator Studio interface (`/creator/studio/analytics`):
     - **Overview KPIs:** Views, total watch time (hours), subscriber net gain/loss, estimated net earnings (from courses and paid memberships).
     - **Interactive Audience Retention Graph:** Second-by-second and percentile curve showing average view duration, engagement spikes, and drop-off points.
     - **Traffic Acquisition Breakdown:** Attribution categorization (Platform Search, Algorithmic Recommendations, Subscriptions Feed, Direct / External).
     - **Comparative Performance Analytics:** 7-day, 28-day, 90-day, and lifetime cohorts compared with preceding historical baselines.

4. **Automated Milestone Trigger Engine:**
   - Evaluates channel thresholds (e.g. 100 subscribers, 1,000 views, 10,000 watch hours).
   - Emits asynchronous milestone celebration events through the notification worker (Feature 15) and unlocks platform badges.

---

## 2. Project Architecture Context

The telemetry and analytics architecture integrates Next.js client beacons, FastAPI ingestion, Redis Streams, Celery aggregation workers, PostgreSQL time-series rollups, and Creator Studio UI:

```
┌────────────────────────────────────────────────────────────────────────┐
│                      CLIENT TIER (Next.js 16+ / TS)                    │
│  • Video Player:     components/video/HlsPlayer.tsx                    │
│  • Telemetry Hook:   hooks/useVideoAnalytics.ts (10s Beacons)          │
│  • Studio Analytics: app/(creator)/studio/analytics/page.tsx           │
│  • Retention Chart:  components/analytics/RetentionChart.tsx           │
└───────────────────┬────────────────────────────────▲───────────────────┘
                    │ 1. POST /telemetry/event       │ 6. Query Rollup APIs
                    ▼                                │    (GET /creator/analytics)
┌──────────────────────────────────────┐             │
│  FASTAPI INGESTION SINK              │             │
│  • endpoints/telemetry.py            │             │
│  • services/telemetry_service.py     │             │
└───────────────────┬──────────────────┘             │
                    │ 2. XADD telemetry:stream       │
                    ▼                                │
┌──────────────────────────────────────┐             │
│  REDIS STREAMS BUFFER                │             │
│  • Stream: telemetry:playback_events │             │
│  • Real-Time Counters (ZINCRBY)      │             │
└───────────────────┬──────────────────┘             │
                    │ 3. Batch Read (XREADGROUP)     │
                    ▼                                │
┌──────────────────────────────────────┐             │
│  CELERY STREAM CONSUMER WORKERS      │             │
│  • tasks/analytics_tasks.py          │             │
│  • View Deduplication & Validation   │             │
│  • Histogram Retention Computation   │             │
└───────────────────┬──────────────────┘             │
                    │ 4. Batch UPSERT Rollups        │
                    ▼                                │
┌────────────────────────────────────────────────────┴───────────────────┐
│                    POSTGRESQL 16 (TIME-SERIES ROLLUPS)                 │
│  • video_daily_metrics (Daily views, watch duration, engagement)       │
│  • channel_daily_metrics (Daily channel aggregates, subscriber churn)  │
│  • video_retention_curves (0-100 percentile bucket histograms)         │
│  • traffic_source_metrics (Attribution channels)                       │
│  • creator_milestones (Unlocked badges & notifications)                │
└────────────────────────────────────────────────────────────────────────┘
```

### Architectural Principles Enforced:
- **Zero Database Pressure on Playback:** Client playback heartbeats never write directly to PostgreSQL. Events buffer in Redis Streams and are bulk-persisted in batches of 1,000 items.
- **Pre-Aggregated Read Views:** Analytics queries never perform raw table scans across billions of telemetry ticks. Dashboards read exclusively from daily summary tables (`video_daily_metrics`).
- **Idempotent Beacon Accounting:** Each playback session generates a unique `session_id`. Telemetry heartbeats carry sequence numbers to prevent duplicate watch time tallying during client network retries.

---

## 3. Existing System Analysis

### Backend Audit (`backend/`):
- **Core Entities & Endpoints:**
  - [backend/app/models/video.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/video.py): Empty placeholder. Intended for `Video`, `VideoView` models. Needs linkage to daily metrics.
  - [backend/app/models/channel.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/channel.py): Empty placeholder for `Channel`. Needs linkage to channel metrics.
  - [backend/app/models/payment.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/payment.py): Empty placeholder for `CreatorPayoutLedger` and transaction history (provides earnings telemetry).
  - [backend/app/api/v1/router.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/api/v1/router.py): Master router ready to register telemetry and analytics endpoints.
- **Workers & Data Infrastructure:**
  - [backend/app/core/redis.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/core/redis.py): Redis client. Used for Redis Streams (`telemetry:playback_events`) and real-time counter caching.
  - [backend/app/core/database.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/core/database.py): SQLAlchemy engine and session factory.
  - [backend/app/celery_worker.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/celery_worker.py): Celery instance setup for periodic analytics rollup tasks.

### Frontend Audit (`frontend/`):
- **Studio Views & Hooks:**
  - [frontend/src/app/(creator)/studio/analytics/page.tsx](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/frontend/src/app/(creator)/studio/analytics/page.tsx): Empty placeholder for the main Studio Analytics dashboard.
  - [frontend/src/app/(creator)/studio/dashboard/page.tsx](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/frontend/src/app/(creator)/studio/dashboard/page.tsx): Studio overview dashboard.
  - [frontend/src/hooks/useVideoAnalytics.ts](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/frontend/src/hooks/useVideoAnalytics.ts): Empty placeholder for the client telemetry collection hook.
  - [frontend/src/hooks/useWatchProgress.ts](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/frontend/src/hooks/useWatchProgress.ts): Watch progress hook.

---

## 4. Functional Requirements

### A. High-Throughput Telemetry Ingestion
1. **Playback Session Tracking:**
   - When a video begins playback, `useVideoAnalytics.ts` generates a cryptographically random `session_id` (UUIDv4).
   - Emits an initial `SESSION_START` event containing referrer, device category, player dimensions, and initial timestamp.
2. **Periodic Heartbeat Beacons:**
   - Emits a `HEARTBEAT` beacon every 10 seconds of active playback.
   - Payload captures:
     - `session_id`, `video_id`, `user_id` (or anonymous fingerprint)
     - `current_position_seconds`
     - `watch_time_delta_seconds` (actual media seconds consumed since last beacon)
     - `playback_rate` ($0.5\times, 1.0\times, 1.5\times, 2.0\times$)
     - `sequence_number` (monotonic integer for deduplication)
3. **Discrete Interaction Events:**
   - Emits `PLAY`, `PAUSE`, `SEEK` (from position, to position), `RATE_CHANGE`, `QUALITY_CHANGE`, `ERROR`, `COMPLETED`.

### B. Asynchronous Stream Aggregation & Rollups
1. **Legitimate View Verification:**
   - Enforces a 30-second continuous watch threshold (or 50% duration for videos shorter than 30s) before incrementing a video's canonical `view_count`.
   - Deduplicates views per user / IP within a 24-hour sliding window.
2. **Audience Retention Curve Generation:**
   - Maps player playback positions into 100 discrete percentile buckets ($0\%$ to $100\%$).
   - Computes:
     $$\text{Retention \% at Bucket } b = \frac{\text{Sessions active at bucket } b}{\text{Total sessions that started video}} \times 100$$
3. **Daily Aggregation Rollups (Celery Cron):**
   - Batches telemetry events from Redis Streams into daily summaries stored in PostgreSQL:
     - Views, unique viewers, total watch time seconds, average view duration, average completion percentage.
     - Traffic sources: `SEARCH`, `RECOMMENDED`, `SUBSCRIPTIONS`, `DIRECT`, `EXTERNAL_COURSE`.
     - Subscriber changes: conversions (subscribed during playback) and unsubscriptions.

### C. Creator Studio Analytics Dashboard
1. **Channel Overview Telemetry (`GET /creator/analytics/overview`):**
   - Selectable timeframes: `7d`, `28d`, `90d`, `365d`, `lifetime`.
   - Comparative analytics: percentage change versus the preceding timeframe.
   - Key metrics: Total Views, Total Watch Time (hours), Net Subscriber Gain, Total Revenue (paise).
2. **Video Performance Table (`GET /creator/analytics/videos`):**
   - Paginated list of channel videos sorted by views, watch time, or revenue.
   - Displays Click-Through Rate (CTR), Average Percentage Viewed (APV), and Likes-to-Views ratio.
3. **Video Retention Deep-Dive (`GET /creator/analytics/videos/{id}/retention`):**
   - Returns 100-point array representing the retention curve for visual rendering in Recharts/Chart.js.
   - Identifies key inflection points: initial drop-off (0-15s), average drop-off, and engagement spikes (re-watched segments).

### D. Automated Milestone Engine
1. **Milestone Detection:**
   - Monitors channel aggregates: 100 subscribers, 1,000 subscribers, 10,000 subscribers, 10,000 views, 100,000 views.
2. **Celebration Notification:**
   - When a milestone is crossed for the first time, writes to `creator_milestones` and schedules an in-app and email notification to the creator via Feature 15.

---

## 5. Non-Functional Requirements

### A. High Throughput & Ingestion Latency
- **Ingestion SLA:** The `POST /api/v1/telemetry/playback-event` endpoint must respond in $\le 10\text{ms}$ at P99 under 5,000 requests/second.
- **Zero-Block Redis Buffering:** Raw payloads are appended to Redis Streams using `XADD` with stream trimming (`MAXLEN ~ 1000000`).

### B. Query Performance & Scalability
- **Dashboard Query SLA:** Studio analytics dashboard queries must return in $\le 50\text{ms}$ by reading exclusively from pre-aggregated daily tables, avoiding analytical aggregates on raw events.
- **Table Partitioning:** In production, `video_daily_metrics` is partitioned by month (`PARTITION BY RANGE (date)`) to maintain index efficiency across years of operational data.

### C. Data Integrity & Deduplication
- **Out-of-Order Packet Handling:** Beacons with sequence numbers $\le$ highest recorded sequence number for a `session_id` are ignored to prevent double-counting during network retries.

---

## 6. Assumptions and Open Questions

### Confirmed Architectural Facts:
- Redis Streams (`XADD`, `XREADGROUP`, `XACK`) are available via the existing Redis infrastructure.
- Celery Beat executes scheduled hourly and nightly rollup jobs.
- Financial earnings telemetry links to `creator_payout_ledger` entries created by the Payment engine (Feature 11).

### Open Engineering Questions:
1. *Raw Telemetry Retention:* How long should raw individual heartbeat ticks be preserved in Redis?  
   *Guidance:* Keep raw heartbeat events in Redis Streams for 24 hours (`MAXLEN ~ 1,000,000`), purging them after Celery workers aggregate them into PostgreSQL daily rollups.
2. *Real-Time "Live" Analytics:* Should the dashboard show real-time views from the past 60 minutes?  
   *Guidance:* Maintain a 60-minute real-time sliding window in Redis using `ZADD / ZREVRANGEBYSCORE` for instant live view counts, while long-term analytics rely on PostgreSQL daily rollups.

---

## 7. Low-Level Design (LLD)

### A. Component Design

```
backend/app/
├── api/v1/endpoints/
│   ├── telemetry.py                 # POST /telemetry/playback-event (High-speed ingestion)
│   └── analytics.py                 # GET /creator/analytics/* (Dashboard queries)
├── models/
│   └── analytics.py                 # VideoDailyMetric, ChannelDailyMetric, VideoRetentionCurve
├── schemas/
│   ├── telemetry.py                 # PlaybackEventBeaconSchema, SessionStartSchema
│   └── analytics.py                 # StudioOverviewResponse, RetentionCurveResponse
├── repositories/
│   └── analytics_repository.py      # Fast aggregated queries on daily metrics
├── services/
│   ├── telemetry_service.py         # Appends events to Redis Stream & updates live zsets
│   └── analytics_service.py         # Calculates timeframe baselines & format transformations
└── tasks/
    └── analytics_tasks.py           # Celery Stream Consumer: consume_telemetry_stream, rollup_daily_metrics
```

#### Component Responsibilities:
1. **`endpoints/telemetry.py`:**
   - Public, ultra-fast ingestion sink. Validates minimal beacon payload and invokes `telemetry_service.ingest_event()`.
2. **`services/telemetry_service.py`:**
   - Appends event to Redis Stream: `XADD telemetry:playback_events * ...`.
   - Increments video real-time 60-minute counter: `ZINCRBY real_time_views:{video_id} 1 {timestamp}`.
3. **`tasks/analytics_tasks.py`:**
   - Celery consumer running in a continuous daemon loop or scheduled every 60 seconds:
     - `XREADGROUP GROUP analytics_workers consumer_1 COUNT 1000 STREAMS telemetry:playback_events >`.
     - Validates watch duration thresholds and accumulates retention histogram buckets.
     - Bulk-persists updates to PostgreSQL and executes `XACK`.
4. **`api/v1/endpoints/analytics.py`:**
   - Protected endpoints requiring `require_channel_owner`.
   - Returns aggregated overview, retention curves, top videos, and traffic breakdowns.

---

### B. Database Design

```sql
-- PostgreSQL Migration: Creator Studio Analytics Engine & Telemetry

-- 1. Video Daily Rollup Metrics Table
CREATE TABLE IF NOT EXISTS video_daily_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    metric_date DATE NOT NULL,
    video_id UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
    channel_id UUID NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    
    views_count INTEGER NOT NULL DEFAULT 0,
    unique_viewers_count INTEGER NOT NULL DEFAULT 0,
    watch_time_seconds BIGINT NOT NULL DEFAULT 0,
    avg_view_duration_seconds INTEGER NOT NULL DEFAULT 0,
    avg_completion_ratio REAL NOT NULL DEFAULT 0.0,
    
    likes_delta INTEGER NOT NULL DEFAULT 0,
    comments_delta INTEGER NOT NULL DEFAULT 0,
    shares_delta INTEGER NOT NULL DEFAULT 0,
    subscribers_gained INTEGER NOT NULL DEFAULT 0,
    
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT uq_video_daily_metric UNIQUE(video_id, metric_date)
);

CREATE INDEX IF NOT EXISTS idx_video_daily_channel_date 
    ON video_daily_metrics(channel_id, metric_date DESC);
CREATE INDEX IF NOT EXISTS idx_video_daily_video_date 
    ON video_daily_metrics(video_id, metric_date DESC);

-- 2. Channel Daily Rollup Metrics Table
CREATE TABLE IF NOT EXISTS channel_daily_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    metric_date DATE NOT NULL,
    channel_id UUID NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    
    total_views INTEGER NOT NULL DEFAULT 0,
    unique_viewers INTEGER NOT NULL DEFAULT 0,
    total_watch_time_seconds BIGINT NOT NULL DEFAULT 0,
    subscribers_gained INTEGER NOT NULL DEFAULT 0,
    subscribers_lost INTEGER NOT NULL DEFAULT 0,
    net_subscribers_delta INTEGER NOT NULL DEFAULT 0,
    estimated_earnings_paise BIGINT NOT NULL DEFAULT 0,
    
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT uq_channel_daily_metric UNIQUE(channel_id, metric_date)
);

CREATE INDEX IF NOT EXISTS idx_channel_daily_date 
    ON channel_daily_metrics(channel_id, metric_date DESC);

-- 3. Video Audience Retention Curves (Percentile Histograms 0-100)
CREATE TABLE IF NOT EXISTS video_retention_curves (
    video_id UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
    bucket_percentile SMALLINT NOT NULL CHECK (bucket_percentile BETWEEN 0 AND 100),
    sample_sessions_count INTEGER NOT NULL DEFAULT 0,
    active_sessions_count INTEGER NOT NULL DEFAULT 0,
    retention_percentage REAL NOT NULL DEFAULT 0.0,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    PRIMARY KEY (video_id, bucket_percentile)
);

-- 4. Traffic Source Attribution Metrics
CREATE TABLE IF NOT EXISTS traffic_source_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    metric_date DATE NOT NULL,
    channel_id UUID NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    video_id UUID NULL REFERENCES videos(id) ON DELETE CASCADE,
    source_type VARCHAR(64) NOT NULL, -- 'SEARCH', 'RECOMMENDED', 'SUBSCRIPTIONS', 'DIRECT', 'EXTERNAL'
    views_count INTEGER NOT NULL DEFAULT 0,
    watch_time_seconds BIGINT NOT NULL DEFAULT 0,
    
    CONSTRAINT uq_traffic_metric UNIQUE(channel_id, video_id, source_type, metric_date)
);

CREATE INDEX IF NOT EXISTS idx_traffic_source_channel_date 
    ON traffic_source_metrics(channel_id, metric_date DESC);

-- 5. Creator Milestones Unlocked Table
CREATE TABLE IF NOT EXISTS creator_milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    channel_id UUID NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    milestone_type VARCHAR(64) NOT NULL, -- 'SUBSCRIBERS_100', 'VIEWS_10K', 'WATCH_HOURS_1K'
    metric_value BIGINT NOT NULL,
    achieved_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    notified BOOLEAN NOT NULL DEFAULT FALSE,
    
    CONSTRAINT uq_channel_milestone UNIQUE(channel_id, milestone_type)
);
```

---

## 8. High-Level Design (HLD)

### Real-Time Ingestion to Analytical Rollup Pipeline

```
[Learner Browser: HlsPlayer.tsx]
       │
       ▼ (Every 10s of active playback)
[POST /api/v1/telemetry/playback-event]
       │
       ▼ (FastAPI App: Parse & Validate Beacon)
[Redis Stream: XADD telemetry:playback_events]
       │
       ├── Return HTTP 204 No Content (< 5ms)
       │
       ▼ (Asynchronous Buffer in Redis)
[Celery Consumer Worker: tasks.analytics_tasks]
       │
       ▼ (Batch read 1,000 events via XREADGROUP)
┌──────┴────────────────────────────────────────────────────────┐
│ 1. Filter out bots / duplicate sequence numbers               │
│ 2. Check 30s legitimate view threshold                        │
│ 3. Compute audience retention percentile (bucket 0-100)       │
│ 4. Accumulate daily deltas: views, duration, likes, comments  │
└──────┬────────────────────────────────────────────────────────┘
       │
       ▼ (Execute PostgreSQL Bulk UPSERT)
┌───────────────────────────────────────────────────────────────┐
│                      POSTGRESQL 16                            │
│  • UPSERT video_daily_metrics (views, watch_time)             │
│  • UPSERT video_retention_curves (bucket percentages)         │
│  • UPSERT channel_daily_metrics (net subscribers, earnings)   │
│  • Check milestone thresholds -> Trigger notification task    │
└───────────────────────────────────────────────────────────────┘
       │
       ▼ (Creator queries studio dashboard)
[GET /api/v1/creator/analytics/overview?timeframe=28d]
       │
       ▼ (Fast query: ~30 pre-aggregated daily rows)
[Creator Studio Dashboard Render: Recharts / Next.js]
```

---

## 9. Architecture Diagrams

### A. Sequence Diagram: Telemetry Ingestion & Studio Dashboard Query

```mermaid
sequenceDiagram
    autonumber
    actor Learner as Learner Browser (HlsPlayer)
    participant TelemetryAPI as FastAPI Telemetry Sink
    participant Redis as Redis Stream Buffer
    participant Worker as Celery Rollup Consumer
    participant DB as PostgreSQL Analytics Tables
    actor Creator as Creator Studio (Next.js)
    participant StudioAPI as FastAPI Studio Endpoint

    loop Every 10 Seconds of Playback
        Learner->>TelemetryAPI: POST /telemetry/playback-event { video_id, session_id, seq: 12, pos: 120, delta: 10 }
        TelemetryAPI->>Redis: XADD telemetry:playback_events * session_id 12 ...
        TelemetryAPI-->>Learner: 204 No Content (Instant response)
    end

    loop Every 60 Seconds (Batch Consumer)
        Worker->>Redis: XREADGROUP GROUP workers c1 COUNT 1000 STREAMS telemetry:playback_events >
        Redis-->>Worker: 1,000 raw telemetry ticks
        Worker->>Worker: Deduplicate, evaluate 30s view rules, update retention buckets
        Worker->>DB: Bulk UPSERT into video_daily_metrics & video_retention_curves
        Worker->>Redis: XACK telemetry:playback_events workers <message_ids>
    end

    Creator->>StudioAPI: GET /creator/analytics/overview?timeframe=28d
    StudioAPI->>DB: SELECT * FROM channel_daily_metrics WHERE metric_date >= NOW() - 28d
    DB-->>StudioAPI: 28 aggregated daily rows
    StudioAPI-->>Creator: 200 OK { total_views, watch_time_hours, net_subscribers, daily_trend: [...] }
```

---

## 10. Database Design

### Entity Relationship Model

```mermaid
erDiagram
    CHANNELS ||--o{ VIDEO_DAILY_METRICS : aggregates
    VIDEOS ||--o{ VIDEO_DAILY_METRICS : records
    CHANNELS ||--o{ CHANNEL_DAILY_METRICS : tracks
    VIDEOS ||--o{ VIDEO_RETENTION_CURVES : measures
    CHANNELS ||--o{ TRAFFIC_SOURCE_METRICS : attributes
    CHANNELS ||--o{ CREATOR_MILESTONES : unlocks
    
    VIDEO_DAILY_METRICS {
        uuid id PK
        date metric_date
        uuid video_id FK
        uuid channel_id FK
        int views_count
        int unique_viewers_count
        bigint watch_time_seconds
        int avg_view_duration_seconds
        real avg_completion_ratio
        int likes_delta
        int comments_delta
        int subscribers_gained
    }

    CHANNEL_DAILY_METRICS {
        uuid id PK
        date metric_date
        uuid channel_id FK
        int total_views
        int unique_viewers
        bigint total_watch_time_seconds
        int subscribers_gained
        int subscribers_lost
        int net_subscribers_delta
        bigint estimated_earnings_paise
    }

    VIDEO_RETENTION_CURVES {
        uuid video_id PK, FK
        smallint bucket_percentile PK
        int sample_sessions_count
        int active_sessions_count
        real retention_percentage
    }

    CREATOR_MILESTONES {
        uuid id PK
        uuid channel_id FK
        varchar milestone_type
        bigint metric_value
        timestamptz achieved_at
    }
```

---

## 11. API Design

### A. Telemetry Ingestion Endpoint

#### 1. Ingest Playback Telemetry Beacon
- **Method / Endpoint:** `POST /api/v1/telemetry/playback-event`
- **Auth:** Public / Optional Auth (Learner session).
- **Rate Limit:** 60 requests per minute per session.
- **Request Body:**
  ```json
  {
    "session_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
    "video_id": "7a35c5c8-1011-4f1b-801b-90f7a93c7d6e",
    "event_type": "HEARTBEAT",
    "sequence_number": 14,
    "position_seconds": 140,
    "watch_time_delta_seconds": 10,
    "playback_rate": 1.0,
    "device_type": "DESKTOP",
    "source_type": "RECOMMENDED"
  }
  ```
- **Response `204 No Content`**

---

### B. Creator Studio Analytics Endpoints

#### 1. Channel Performance Overview
- **Method / Endpoint:** `GET /api/v1/creator/analytics/overview`
- **Auth:** Bearer JWT (Creator role, channel owner).
- **Query Parameters:** `timeframe` (`7d`, `28d`, `90d`, `365d`, `lifetime`, default `28d`).
- **Response `200 OK`:**
  ```json
  {
    "channel_id": "c1a2b3c4-...",
    "timeframe": "28d",
    "summary": {
      "total_views": 48250,
      "views_growth_percent": 18.4,
      "watch_time_hours": 3210.5,
      "watch_time_growth_percent": 22.1,
      "net_subscribers": 640,
      "subscribers_growth_percent": 12.0,
      "estimated_earnings_paise": 1845000,
      "estimated_earnings_growth_percent": 15.6
    },
    "timeline": [
      {
        "date": "2026-10-08",
        "views": 1620,
        "watch_time_hours": 108.4,
        "net_subscribers": 24,
        "earnings_paise": 62000
      }
    ]
  }
  ```

#### 2. Video Audience Retention Curve
- **Method / Endpoint:** `GET /api/v1/creator/analytics/videos/{video_id}/retention`
- **Auth:** Bearer JWT (Channel owner).
- **Response `200 OK`:**
  ```json
  {
    "video_id": "7a35c5c8-1011-4f1b-801b-90f7a93c7d6e",
    "duration_seconds": 1200,
    "average_view_duration_seconds": 640,
    "average_percentage_viewed": 53.3,
    "retention_curve": [
      { "percentile": 0, "retention_percentage": 100.0 },
      { "percentile": 10, "retention_percentage": 82.5 },
      { "percentile": 50, "retention_percentage": 58.0 },
      { "percentile": 80, "retention_percentage": 46.2 },
      { "percentile": 100, "retention_percentage": 34.0 }
    ],
    "highlights": {
      "initial_drop_off_percent": 17.5,
      "spike_positions_seconds": [320, 680]
    }
  }
  ```

#### 3. Traffic Source Attribution
- **Method / Endpoint:** `GET /api/v1/creator/analytics/traffic-sources`
- **Response `200 OK`:**
  ```json
  {
    "timeframe": "28d",
    "sources": [
      { "source": "RECOMMENDED", "percentage": 48.5, "views": 23400 },
      { "source": "SEARCH", "percentage": 28.2, "views": 13600 },
      { "source": "SUBSCRIPTIONS", "percentage": 14.3, "views": 6900 },
      { "source": "DIRECT", "percentage": 9.0, "views": 4350 }
    ]
  }
  ```

---

## 12. Security Analysis

Security evaluation across 4 progressive tiers:

```
┌────────────────────────────────────────────────────────────────────────┐
│  LEVEL 4: CRYPTOGRAPHIC BEACON PROOF & ENTROPY SIGNATURES              │
│  • Player telemetry token signed with session HMAC to verify integrity │
│  • Rate-decay scoring detecting automated headless browser view bots   │
│  • IP salt rotation for GDPR-compliant viewer deduplication           │
├────────────────────────────────────────────────────────────────────────┤
│  LEVEL 3: PRODUCTION-READY SECURITY (TARGET SPECIFICATION)             │
│  • Strict channel ownership enforcement on analytics query APIs        │
│  • Sliding-window rate limiting on telemetry ingestion (max 100/min)   │
│  • Anti-Tampering: Sequence monotonic checks to reject replayed ticks  │
│  • Max delta clamping: Reject beacons claiming > 30s watch delta       │
├────────────────────────────────────────────────────────────────────────┤
│  LEVEL 2: BASIC INPUT SANITIZATION & BOUND CHECKS                      │
│  • Enforce schema validation on device types, positions, and durations │
│  • Timeframe validation bounding date filters to max 365 days          │
│  • Verify video existence before logging telemetry ticks               │
├────────────────────────────────────────────────────────────────────────┤
│  LEVEL 1: VULNERABLE & INSECURE BASELINE (ANTI-PATTERNS TO AVOID)      │
│  • Allowing creators to view other creators' private analytics (BOLA)  │
│  • Executing UPDATE videos SET view_count = view_count + 1 on every    │
│    client beacon, causing catastrophic DB write locks and crashes      │
│  • Accepting unverified watch deltas (e.g. delta=1,000,000 seconds)    │
└────────────────────────────────────────────────────────────────────────┘
```

### Critical Security Vulnerabilities Addressed:
1. **Broken Object Level Authorization (BOLA / IDOR):**
   - *Risk:* A creator accesses `GET /creator/analytics/overview?channel_id=ANOTHER_CHANNEL` or inspects rival videos' private retention curves.
   - *Mitigation:* The backend extracts the authenticated user's ID from the JWT token and verifies channel ownership via dependency `require_channel_owner` before executing database queries.
2. **Watch Time Inflation & Fake Metrics Injection:**
   - *Risk:* A malicious user crafts an HTTP POST claiming `watch_time_delta_seconds: 999999` to falsely inflate video metrics or creator monetization payouts.
   - *Mitigation:* The telemetry service enforces hard boundary validation: `if delta > 30 or delta < 0: reject()`. Sequence numbers must increase strictly by 1 (`seq == last_seq + 1`), and total session watch time cannot exceed wall-clock session duration.
3. **Database Write Starvation (Denial of Service):**
   - *Risk:* 10,000 concurrent viewers hitting the database directly for view count updates will exhaust PostgreSQL connection pools and write IOPS.
   - *Mitigation:* Telemetry ingress is decoupled through Redis Streams (`XADD`) and processed in bulk asynchronous micro-batches by Celery workers.

---

## 13. Existing Files Analysis

### A. Directly Reusable Files

| Exact File Path | Existing Responsibility | Relevant Items | How It Helps & Why Reuse |
| :--- | :--- | :--- | :--- |
| [backend/app/core/redis.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/core/redis.py) | Redis client singleton | `get_redis_client()` | Provides Redis Streams (`XADD`, `XREADGROUP`) and real-time live view counters. |
| [backend/app/core/database.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/core/database.py) | DB session factory | `get_db()`, `Base` | Manages relational storage and bulk UPSERTs for daily analytics rollups. |
| [backend/app/models/video.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/video.py) | Video model | `Video` | Target entity for video metrics and retention curves. |
| [backend/app/models/channel.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/channel.py) | Channel model | `Channel` | Target entity for channel analytics and milestone achievements. |
| [backend/app/api/deps.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/api/deps.py) | Auth dependencies | `get_current_active_user` | Authenticates creator identity for protected analytics endpoints. |

### B. Files That Need Modification

| Exact File Path | Current State | Required Modification |
| :--- | :--- | :--- |
| [backend/app/api/v1/router.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/api/v1/router.py) | Empty placeholder | Register `telemetry.router` under `/telemetry` and `analytics.router` under `/creator/analytics`. |
| [frontend/src/hooks/useVideoAnalytics.ts](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/frontend/src/hooks/useVideoAnalytics.ts) | Empty placeholder | Implement 10-second beacon timer, session initialization, visibility tracking, and beacon dispatches. |
| [frontend/src/app/(creator)/studio/analytics/page.tsx](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/frontend/src/app/(creator)/studio/analytics/page.tsx) | Empty placeholder | Build creator analytics dashboard with Recharts metric cards, retention curves, and traffic breakdowns. |
| [frontend/src/app/(creator)/studio/dashboard/page.tsx](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/frontend/src/app/(creator)/studio/dashboard/page.tsx) | Empty placeholder | Integrate high-level 28-day analytics overview card and milestone notifications. |

### C. Proposed New Files

| Suggested Path | Expected Responsibility |
| :--- | :--- |
| `backend/app/api/v1/endpoints/telemetry.py` | Ingestion endpoint: `POST /telemetry/playback-event`. |
| `backend/app/api/v1/endpoints/analytics.py` | Studio analytics endpoints: `/creator/analytics/overview`, `/videos/{id}/retention`, `/traffic-sources`. |
| `backend/app/models/analytics.py` | SQLAlchemy models: `VideoDailyMetric`, `ChannelDailyMetric`, `VideoRetentionCurve`, `CreatorMilestone`. |
| `backend/app/schemas/telemetry.py` | Pydantic schemas for client beacon payload and session initiation. |
| `backend/app/schemas/analytics.py` | Pydantic response models for studio KPIs, timeline arrays, and retention data. |
| `backend/app/repositories/analytics_repository.py` | Database queries for timeframe aggregation, comparative percentage deltas, and bulk UPSERTs. |
| `backend/app/services/telemetry_service.py` | Appends events to Redis Streams and updates 60-minute sliding window counters. |
| `backend/app/services/analytics_service.py` | Formats timeframe summaries, calculates growth deltas, and computes retention percentages. |
| `backend/app/tasks/analytics_tasks.py` | Celery stream consumer worker and daily scheduled metric rollup tasks. |
| `frontend/src/services/analyticsService.ts` | Frontend API client for creator studio analytics endpoints. |
| `frontend/src/types/analytics.ts` | TypeScript types for analytics KPIs, timelines, retention histograms, and traffic sources. |
| `frontend/src/components/analytics/RetentionChart.tsx` | Interactive SVG / Recharts area chart rendering the 0-100% audience retention curve. |

### D. Files That Should NOT Be Modified

| File Path | Why It Must Remain Untouched |
| :--- | :--- |
| [backend/app/tasks/video_tasks.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/tasks/video_tasks.py) | Video transcoding pipeline; telemetry aggregation runs independently in `analytics_tasks.py`. |
| [backend/app/models/user.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/user.py) | User identity model; analytics references channel and video IDs without mutating the user schema. |

### E. Component Dependency Map

```mermaid
graph TD
    FEPlayer["hooks/useVideoAnalytics.ts"] --> TelemetryAPI["api/v1/endpoints/telemetry.py"]
    TelemetryAPI --> TelemetryService["services/telemetry_service.py"]
    TelemetryService --> RedisStream["Redis Streams (telemetry:playback_events)"]
    
    CeleryWorker["celery_worker.py"] --> AnalyticsTasks["tasks/analytics_tasks.py"]
    AnalyticsTasks --> RedisStream
    AnalyticsTasks --> AnalyticsRepo["repositories/analytics_repository.py"]
    AnalyticsRepo --> DB[("PostgreSQL Analytics Tables")]
    
    AnalyticsAPI["api/v1/endpoints/analytics.py"] --> AnalyticsService["services/analytics_service.py"]
    AnalyticsService --> AnalyticsRepo
    
    StudioUI["app/(creator)/studio/analytics/page.tsx"] --> FEClient["services/analyticsService.ts"]
    FEClient --> AnalyticsAPI
    StudioUI --> FERetention["components/analytics/RetentionChart.tsx"]
```

---

## 14. Implementation Roadmap

### Milestone 1: Data Modeling & Schema Migrations
- **Goal:** Model `video_daily_metrics`, `channel_daily_metrics`, `video_retention_curves`, and `creator_milestones` with relational constraints.
- **Inspect:** [backend/app/models/video.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/video.py), [backend/app/models/channel.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/models/channel.py).
- **Implement In:** `backend/app/models/analytics.py`, Alembic migration script.
- **Architectural Hint:** Create unique constraints on `(video_id, metric_date)` and `(channel_id, metric_date)` to enable atomic PostgreSQL `ON CONFLICT DO UPDATE` (UPSERT) operations.

### Milestone 2: High-Speed Telemetry Ingestion Sink
- **Goal:** Implement the lightweight beacon endpoint buffering playback events to Redis Streams.
- **Inspect:** [backend/app/core/redis.py](file:///e:/Projects/Open%20Source%20Contributions/RKSkillTube/backend/app/core/redis.py).
- **Implement In:** `backend/app/api/v1/endpoints/telemetry.py`, `backend/app/services/telemetry_service.py`.
- **Architectural Hint:** Use `XADD telemetry:playback_events MAXLEN ~ 1000000 * ...` to automatically trim the stream and prevent memory unbounded growth in Redis.

### Milestone 3: Celery Stream Consumer & Daily Rollup Aggregator
- **Goal:** Build the Celery worker processing micro-batches from Redis and persisting daily aggregates and retention histograms.
- **Inspect:** `backend/app/celery_worker.py`, `backend/app/tasks/analytics_tasks.py`.
- **Architectural Hint:** Enforce 30-second continuous watch threshold before registering a legitimate view. Calculate retention percentiles: `bucket = int((pos / duration) * 100)`.

### Milestone 4: Creator Studio Analytics APIs
- **Goal:** Build the protected studio analytics endpoints delivering timeframe overviews, comparative growth deltas, retention curves, and traffic breakdowns.
- **Inspect:** `backend/app/services/analytics_service.py`, `backend/app/api/v1/endpoints/analytics.py`.
- **Architectural Hint:** Ensure all queries filter strictly by `channel_id` owned by the authenticated creator to prevent IDOR vulnerabilities.

### Milestone 5: Frontend Telemetry Hook & Dashboard Visualization
- **Goal:** Implement the client telemetry collection hook in Next.js and build interactive analytics charts in the Creator Studio.
- **Inspect:** `frontend/src/hooks/useVideoAnalytics.ts`, `frontend/src/app/(creator)/studio/analytics/page.tsx`.
- **Architectural Hint:** In `useVideoAnalytics.ts`, listen to `document.visibilityState` to pause the 10-second beacon when the browser tab is hidden, preventing fraudulent background watch time accumulation.

---

## 15. Hints and Engineering Questions

### Socratic Mentorship Questions:

1. **Why do we buffer playback beacons in Redis Streams instead of updating the database directly?**
   *Hint:* If 10,000 active students send a heartbeat beacon every 10 seconds, that produces 1,000 write queries per second. What happens to PostgreSQL row-level locks and connection pool capacity if 1,000 concurrent transactions attempt to execute `UPDATE videos SET watch_time = watch_time + 10`?

2. **Why should the client pause the telemetry heartbeat when `document.visibilityState === 'hidden'`?**
   *Hint:* If a student leaves a video tab open in the background for 8 hours while sleeping, does that represent genuine engagement? How does visibility checking protect the integrity of creator retention metrics and course progress tracking?

3. **Why must we validate `watch_time_delta_seconds <= 30` on the server side?**
   *Hint:* Can client-side JavaScript timers always be trusted? What would prevent a bad actor from modifying the payload to send `watch_time_delta_seconds: 50000` to fraudulently inflate monetization revenue or milestone achievements?

4. **Why do we use pre-aggregated daily rollup tables (`video_daily_metrics`) instead of running `SUM()` and `COUNT()` on raw event logs when the creator opens their dashboard?**
   *Hint:* Over a 1-year period, a channel with 50,000 subscribers will generate over 20,000,000 raw telemetry ticks. How long would a PostgreSQL query take to aggregate 20,000,000 rows on every page reload versus reading 28 rows from `channel_daily_metrics`?

---

## 16. Testing Strategy

### A. Unit Testing
- **Beacon Validation Logic:** Verify that beacons with `watch_time_delta_seconds > 30` or negative deltas are rejected with `HTTP 422`.
- **Retention Bucket Calculation:** Test that positions are correctly mapped to integer percentiles ($0 \dots 100$) and clamp values exceeding video duration.
- **Comparative Percentage Delta:** Test `calculate_growth_percentage(current, previous)` handling edge cases such as division by zero when the previous period has zero views.

### B. Integration Testing
- **End-to-End Telemetry Ingestion:** Submit 15 consecutive 10-second heartbeats for a test session (total 150 seconds).
  - Verify that events appear in Redis Streams.
  - Trigger `consume_telemetry_stream_task`.
  - Assert that `video_daily_metrics` records exactly 1 view (since duration exceeded 30 seconds) and 150 seconds of watch time.
- **Studio Authorization (IDOR Test):** Authenticate as Creator A. Attempt to query `GET /creator/analytics/videos/{video_b_id}/retention` for a video belonging to Creator B; assert that the API returns `HTTP 403 Forbidden`.

### C. Frontend Telemetry & Dashboard Testing
- Verify that `useVideoAnalytics` emits heartbeats at 10-second intervals and flushes the final beacon on component unmount or window `beforeunload`.
- Verify that `RetentionChart.tsx` renders smoothly with tooltips displaying retention percentages at given time offsets.

---

## 17. Engineering Trade-offs

| Decision | Option A: Redis Streams + PostgreSQL Daily Rollups | Option B: Dedicated Analytical DB (ClickHouse / TimescaleDB) |
| :--- | :--- | :--- |
| **Operational Simplicity** | High: uses existing Redis and PostgreSQL infrastructure; zero extra servers. | Low: requires provisioning, clustering, and managing a separate column-store database. |
| **Ingestion Throughput** | Very High: Redis Streams easily sustain 20,000+ writes/second. | Extreme: ClickHouse sustains millions of raw rows per second. |
| **Query Complexity** | Simple relational queries on small pre-aggregated tables ($\le 365$ rows per year). | Real-time queries over billions of raw unstructured event rows. |
| **Recommendation** | **Option A (Redis Streams + PostgreSQL Rollups)** is optimal for RKSkillTube open-source, eliminating heavy infrastructure dependencies while maintaining $< 10\text{ms}$ ingestion and $< 50\text{ms}$ dashboards. |

| Ingestion Frequency | Option A: 10-Second Periodic Heartbeat | Option B: Real-Time Second-by-Second Streaming |
| :--- | :--- | :--- |
| **Network Traffic** | Minimal: 6 requests per minute per active viewer. | Excessive: 60 requests per minute per active viewer; battery drain on mobile. |
| **Granularity** | Sufficient to compute accurate retention curves and watch time. | Unnecessary precision for creator studio analytics. |
| **Recommendation** | **Option A (10-Second Heartbeat)** is the proven industry standard (matching YouTube and Netflix telemetry architectures). |

---

## 18. Learning Resources

1. **Building Real-Time Analytics with Redis Streams:**
   - URL: `https://redis.io/docs/data-types/streams/`
   - Key Study: `XADD`, consumer groups (`XREADGROUP`), stream trimming (`MAXLEN`), and acknowledgment (`XACK`).
2. **PostgreSQL High-Performance Time-Series Aggregations:**
   - URL: `https://www.postgresql.org/docs/current/queries-table-expressions.html`
   - Key Study: Atomic `ON CONFLICT DO UPDATE` (UPSERTs), covering indexes, and table partitioning.
3. **Designing Audience Retention Curves (YouTube Creator Architecture):**
   - URL: `https://support.google.com/youtube/answer/9314415`
   - Key Study: Interpreting audience retention histograms, identifying drop-offs, and calculating average view duration.
4. **W3C Beacon API & Page Lifecycle:**
   - URL: `https://w3c.github.io/beacon/`
   - Key Study: Reliable background telemetry dispatch during page unloads and tab switching.

---

## 19. Acceptance Criteria

- [ ] Video player emits a telemetry heartbeat every 10 seconds of active playback via `useVideoAnalytics.ts`.
- [ ] Telemetry ingestion endpoint (`POST /telemetry/playback-event`) appends events to Redis Streams and returns `204 No Content` in $< 10\text{ms}$.
- [ ] Telemetry heartbeats pause when the browser tab is hidden (`document.visibilityState === 'hidden'`).
- [ ] Celery consumer processes Redis Stream batches and updates `video_daily_metrics` and `channel_daily_metrics`.
- [ ] Views are counted only after a continuous watch threshold of 30 seconds is reached.
- [ ] Audience retention curve is computed across 100 percentile buckets and stored in `video_retention_curves`.
- [ ] Creator Studio overview (`/creator/analytics/overview`) returns accurate KPIs and comparative growth deltas for 7d, 28d, 90d, and lifetime.
- [ ] Cross-creator analytics access is strictly blocked with `HTTP 403 Forbidden`.
- [ ] Crossing channel subscriber or view milestones writes to `creator_milestones` and triggers celebration notifications.
- [ ] Frontend `RetentionChart.tsx` renders interactive retention curves with clear inflection point highlights.

---

## 20. Contributor Pull Request Checklist

- [ ] I have used Redis Streams (`XADD`) to buffer client telemetry events, avoiding direct database writes in the HTTP request.
- [ ] I have enforced strict validation on `watch_time_delta_seconds` (clamped between 0 and 30 seconds).
- [ ] I have verified that all analytics endpoints require channel ownership authentication.
- [ ] I have added unique constraints and composite indexes on `(video_id, metric_date)` and `(channel_id, metric_date)`.
- [ ] I have included Alembic migrations for `video_daily_metrics`, `channel_daily_metrics`, `video_retention_curves`, and `creator_milestones`.
- [ ] I have written unit tests verifying stream ingestion, 30-second view thresholds, and retention bucket calculations.
- [ ] I have tested the Creator Studio analytics dashboard in Next.js, verifying chart responsiveness and KPI calculations.

---

## 21. Final Architecture Recommendations & Document Checksum

### Architectural Recommendations:
1. **Use `navigator.sendBeacon` for Session End:** On player unmount or page unload (`beforeunload`), dispatch the final telemetry payload using `navigator.sendBeacon('/api/v1/telemetry/playback-event', data)` to ensure the final watch seconds are transmitted even as the page closes.
2. **PostgreSQL Table Partitioning:** When scaling past 1,000 active channels, partition `video_daily_metrics` by month (`PARTITION BY RANGE (metric_date)`) to maintain small B-tree index footprints and enable instant drop-partition archiving of historical metrics.
3. **Rollup Cache Warming:** Cache the 28-day channel overview response in Redis (`analytics:channel:{id}:28d`) with a 1-hour TTL, invalidating the cache when the nightly rollup Celery task finishes.

### Document Integrity Checksum:
- **Specification File:** `Feature Based Guide/High/19-Creator-Studio-Analytics-Engine-&-Telemetry.md`
- **MD5 Checksum:** `A56271FA0E49039D59498C1C4D253638`
- **SHA-256 Checksum:** `005627035A315C6D29F1D5E41E1D6187079CE9DB46D77F3CF69E6637175AB0B7`
- **Verification Command (PowerShell):**
  ```powershell
  Get-FileHash -Path "Feature Based Guide\High\19-Creator-Studio-Analytics-Engine-&-Telemetry.md" -Algorithm MD5
  ```
