name: flutter-android-live-update-skill
description: Create, adapt, or review reusable Flutter real-time ongoing notification integrations that can support Android 16 Live Update and status chips when the app has a compatible use case such as delivery, transit, mobility, navigation, scheduler timers, countdown workflows, tracking, uploads, or any continuously updated progress notification.
---

# Flutter Live Update Skill

Use this skill when the user wants to add or review reusable real-time notification support in a Flutter app, especially when the app can benefit from Android 16 Live Update promoted ongoing notifications and status chips for delivery, movement, route progress, or scheduler/countdown flows.

## What this skill covers

- Flutter app suitability for real-time ongoing notifications
- Android Live Update feature requirements and platform constraints
- Android 16 Live Update promoted ongoing notification requirements
- Status chip text via `setShortCriticalText`
- Manifest permissions, runtime checks, and notification channel setup
- Integration points between Flutter and Android native notification code
- Flutter `MethodChannel` / platform bridge design for posting, updating, and cancelling live updates
- Delivery / movement / scheduler timer use cases
- Common Live Update and ongoing-notification failure modes

## Workflow

1. Confirm the Flutter app has a real real-time use case such as delivery, transit, navigation, movement tracking, scheduler timers, countdown sessions, uploads, downloads, workouts, or other continuously updated progress.
2. Check Flutter-to-native integration points and where ongoing notification state should be emitted.
3. Check `compileSdk`, `targetSdk`, `androidx.core` version, manifest permissions, and notification channel creation timing.
4. Map the Dart-side state model to a native Android notification manager that owns channel creation, post/update/cancel, and diagnostics.
5. Verify the notification is ongoing, eligible for promotion, and not using patterns that break Android 16 Live Update.
6. Keep chip text short and immediately understandable.
7. Validate device-side conditions:
   - notification permission
   - promoted notification permission/state
   - app-level Live Update setting visibility

## Read next

- For implementation details, reusable Kotlin snippets, and guidance for Flutter apps with compatible real-time notifications, read [references/live-update.md](references/live-update.md).
- For Flutter-to-Android integration structure based on a real Android test app, read [references/flutter-integration.md](references/flutter-integration.md).

## Android Live Update basics

- Android Live Update is an Android 16 feature and should be treated as an Android-native notification capability, not a Flutter-only UI feature.
- In practice, the Android side should be built with `compileSdk = 36` and `targetSdk = 36` to use the Android 16 Live Update APIs and behavior correctly.
- A Flutter app can drive the state, but the actual Live Update notification still requires Android native notification implementation.
- If the project does not have an Android native layer for notifications, this skill should guide the user to add one through `MethodChannel`, plugin code, or direct Android app module changes.
- A Live Update should only be used for finite, trackable, user-initiated, and time-sensitive work.

## Guardrails

- Prefer creating the Live Update channel during app startup, not only when the service starts.
- Avoid `setColorized(true)` for Live Update candidates.
- Require `contentTitle`.
- Do not use `customContentView` / `RemoteViews`.
- Do not use `setGroupSummary(true)` for the live update notification.
- Do not use a channel with `IMPORTANCE_MIN`.
- Do not recommend Live Update for one-shot or low-urgency notifications.
- For scheduler apps, only use Live Update after the user explicitly starts a running session, timer, or countdown.
- Prefer keeping the Dart layer focused on state changes and the Android layer focused on notification presentation.
- Alert only for critical status changes. Do not re-alert on minor ETA or progress adjustments.
- Consider `setDeleteIntent` so the app can detect when the user dismisses or demotes the live update.
- If the app still relies on `http://` traffic or debug-only notification settings, call that out explicitly before shipping.
