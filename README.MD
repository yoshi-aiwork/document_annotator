# Mobile Data Usage Monitor (rename repo to `mobile-data-usage-monitor`)

This repository houses the MVP for a Pixel 7 / Android 16 app that surfaces mobile data usage per app per hour with 7-day history.

> Repository hygiene: please rename this repo from `document_annotator` to `mobile-data-usage-monitor` (or similar) and set the default branch to `main` so CI workflows and downstream scripts align with the project scope.

## MVP Snapshot
- Show top mobile-data-using apps for today with a 7-day navigation window.
- App detail view with hourly breakdown, totals (rx/tx/total bytes), and a short lag disclaimer.
- Mobile-only; Wi-Fi capture, analytics, ads, login, and CSV export are out of scope for this MVP.

## Tech & Platform
- Kotlin, Jetpack Compose + Material 3 UI
- Room for persistence; WorkManager for periodic refresh/backfill
- Uses `NetworkStatsManager` for per-UID mobile counters, time-aligned to hourly buckets

## Permissions & Access Flow
- Usage access grant via `Settings.ACTION_USAGE_ACCESS_SETTINGS` and `AppOpsManager.OPSTR_GET_USAGE_STATS`
- Mobile subscriber context may require `READ_PHONE_STATE`; if unavailable, surface a clear "mobile stats unavailable" message while keeping the UI shell usable

## Development Notes
- Gradle wrapper committed; single Android app module targeting SDK 35 (min 29+)
- GitHub Actions should build the debug APK and publish `app/build/outputs/apk/debug/app-debug.apk` as an artifact—no emulator required
- Include a manual refresh control and keep background work battery-friendly (`setRequiresBatteryNotLow(true)`)

For detailed implementation guidance, see `AGENTS.md`.

## MVP task list (prioritized for seeing mobile usage on-device)
1) **Repo + build readiness**
   - Rename repo to `mobile-data-usage-monitor`, set default branch to `main`, keep Gradle wrapper and Android app module checked in.
   - Add GitHub Actions debug APK build + artifact upload to verify the project is installable on a Pixel 7 without a laptop.

2) **Permissions & gating UX**
   - Implement onboarding screen that checks `AppOpsManager.OPSTR_GET_USAGE_STATS` and links to `Settings.ACTION_USAGE_ACCESS_SETTINGS`.
   - Gate main UI until access is granted; show a fallback message if `NetworkStatsManager` is unavailable or subscriber info is missing.

3) **Time alignment + helpers**
   - Implement `dayStartMillis` and `hourStartMillis` helpers using device local timezone; enforce hour-aligned buckets for inserts.

4) **Data sourcing (mobile-only)**
   - Build `NetworkStatsRepository` that queries `NetworkStatsManager.queryDetailsForUid(ConnectivityManager.TYPE_MOBILE, subscriberId, start, end, uid)` for each UID/hour window.
   - Handle missing `subscriberId` gracefully (dual-SIM or restricted devices) and surface "mobile stats unavailable" while keeping UI usable.

5) **Persistence**
   - Create Room `HourlyUsageEntity` with indexes on `(packageName, hourStartMillis)`, `(dayStartMillis)`, `(uid, hourStartMillis)`; optional `AppEntity` for labels/icons.
   - DAO: totals per app/day, hourly per app/day, and 7-day totals across apps.

6) **UI flows (Compose + Material 3)**
   - Onboarding screen with CTA + status.
   - Dashboard: day selector (Today..D-6), sort (highest/alphabetical), optional system-app toggle, list of apps with icon/name/total bytes, footer lag disclaimer.
   - App detail: day selector, hourly bar chart (0–23), totals row (rx/tx/total), optional background refresh status.
   - Manual refresh control available from dashboard/detail.

7) **Background jobs (WorkManager)**
   - `TodayRefreshWorker`: runs every 60–180 minutes with battery-not-low constraint; upserts current-day hourly buckets.
   - `BackfillWorker`: chained one-time jobs for days D-1..D-6; optionally gate on charging; no network requirement.

8) **Delivery to device**
   - After each push, download `app/build/outputs/apk/debug/app-debug.apk` from Actions and install on Pixel 7.
   - Validate that today’s top apps appear and that hourly breakdowns change after using mobile data.
