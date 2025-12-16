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
