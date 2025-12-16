# Agent Guidance: Mobile Data Usage Monitor (MVP)

This repository targets an Android MVP that surfaces **mobile data usage per app per hour** for the last 7 days on **Pixel 7 / Android 16**. Follow these directions when modifying code or docs within this repo.

## Scope
All files in this repository follow the instructions in this document.

## Platform & Tech Stack
- **Language:** Kotlin
- **UI:** Jetpack Compose with Material 3
- **Data:** Room (SQLite)
- **Background:** WorkManager
- **Optional DI:** Hilt (keep minimal if used)
- **Charts:** Prefer lightweight Compose bar chart; avoid heavy dependencies.

## Permissions & Access Flow
- Require **Usage Access**; surface onboarding with CTA to `Settings.ACTION_USAGE_ACCESS_SETTINGS` and status indicator using `AppOpsManager.OPSTR_GET_USAGE_STATS`.
- Handle runtime subscriber context carefully: `READ_PHONE_STATE` may be needed to fetch `subscriberId`; when unavailable, show clear UI messaging that mobile stats are unavailable.
- If access is missing or stats cannot be read, keep the UI shell and show a prominent fallback notice.

## Data Model Expectations
Use Room entities aligned with:
- `HourlyUsageEntity` capturing UID, package name/label, hour-aligned timestamps, rx/tx/total bytes, and day-start for fast queries. Index `(packageName, hourStartMillis)`, `(dayStartMillis)`, `(uid, hourStartMillis)`.
- Optional `AppEntity` cache for package metadata.

Key DAO patterns:
- Daily totals per app (group by `packageName`, filter by `dayStartMillis`).
- Hourly records per app/day ordered by `hourStartMillis`.
- 7-day totals across apps via a bounded hour range.

## Core Logic
- Use `NetworkStatsManager.queryDetailsForUid(ConnectivityManager.TYPE_MOBILE, subscriberId, start, end, uid)` to aggregate rx/tx bytes; align hour/day helpers to device local time.
- Optimize by prioritizing today + yesterday first; backfill the remaining 5 days via background work.
- If multiple packages share a UID, map to a representative package for MVP and note limitations in UI copy.

## Background Work
- `TodayRefreshWorker`: update hourly buckets for the current day every 60–180 minutes (flex). Upsert results.
- `BackfillWorker`: chained one-time jobs for days D-1 through D-6. Constrain to `setRequiresBatteryNotLow(true)`; avoid requiring network.

## UI / UX Baseline
- **Onboarding:** explain permission need; button to open settings.
- **Dashboard:** day selector (Today..D-6), sort options, optional system-app toggle; list shows icon, name, total bytes. Footer note that stats may lag.
- **App Detail:** day selector, hourly bar chart (0–23), totals row, optional background refresh status.
- **Formatting:** consistent human-readable bytes (KB/MB/GB, choose base and stick with it).
- **Manual refresh** control should exist for the user.

## Testing Targets (Pixel 7 / Android 16)
- Grant usage access → dashboard shows today’s mobile data per app.
- Mobile data activity increases totals after refresh.
- Day switching reflects historical data (once backfilled).
- Revoking permission returns user to onboarding.

## CI & Delivery
- Single Android app module with Gradle wrapper committed; target SDK 35, min SDK 29+.
- GitHub Actions workflow to build debug APK on each push and upload `app/build/outputs/apk/debug/app-debug.apk` as an artifact. Emulator not required.

## Non-Goals (MVP)
- Wi-Fi stats, real-time packet capture, cloud sync/analytics/ads, account/login, CSV export.

