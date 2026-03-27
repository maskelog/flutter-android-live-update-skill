# flutter-android-live-update-skill

Reusable skill for adding Android 16 Live Update support to Flutter apps that have real-time ongoing notification use cases.

실시간 ongoing 알림이 필요한 Flutter 앱에 Android 16 Live Update를 적용할 때 재사용할 수 있는 스킬입니다.

## Supports

- Delivery apps
- Transit and mobility apps
- Navigation and movement tracking apps
- Scheduler and countdown apps
- Upload and download progress apps
- Workout and activity tracking apps

- 배달 앱
- 대중교통 / 이동 앱
- 내비게이션 / 이동 추적 앱
- 스케줄러 / 카운트다운 앱
- 업로드 / 다운로드 진행률 앱
- 운동 / 활동 추적 앱

## What This Skill Helps With

- Promoted ongoing notification integration on Android 16
- Status chip text with `setShortCriticalText`
- Notification channel setup
- Permission checks
- Flutter-to-Android integration points
- Reusable implementation guidance for apps with continuously updated progress

- Android 16 promoted ongoing notification 연동
- `setShortCriticalText` 기반 상태칩 처리
- 알림 채널 구성
- 권한 점검
- Flutter와 Android 네이티브 연결 지점 정리
- 지속적으로 상태가 바뀌는 앱에 맞는 재사용 가능한 구현 가이드

## Repository Structure

```text
flutter-android-live-update-skill/
├── SKILL.md
└── references/
    └── live-update.md
```

## Use In Codex

Place this folder under your Codex skills directory.

Example:

```text
$CODEX_HOME/skills/flutter-android-live-update-skill
```

Then use it by referencing the skill in your workflow or by asking Codex to use the Flutter Android Live Update skill for compatible apps.

Codex에서 사용하려면 이 폴더를 Codex skills 디렉터리 아래에 두면 됩니다.

예시:

```text
$CODEX_HOME/skills/flutter-android-live-update-skill
```

그 다음 Flutter 앱의 실시간 알림 기능 작업 시 이 스킬을 사용하도록 요청하면 됩니다.

## Use In Claude Code

Place this folder where your Claude Code skill set or shared skill library expects custom skills.

Common pattern:

```text
.claude/skills/flutter-android-live-update-skill
```

Then reference the skill when working on Flutter apps that need Android Live Update support.

Claude Code에서 사용하려면 사용자 환경의 커스텀 스킬 경로 또는 공유 스킬 라이브러리 위치에 이 폴더를 두면 됩니다.

일반적인 예시:

```text
.claude/skills/flutter-android-live-update-skill
```

이후 Android Live Update가 필요한 Flutter 앱 작업에서 이 스킬을 참조하면 됩니다.

## Notes

- This skill is intended for apps with real-time state changes, not one-shot notifications.
- Scheduler apps should use Live Update only after the user starts an active session, timer, or countdown.
- Android-only Live Update behavior still requires native Android notification implementation.

- 이 스킬은 단발성 알림이 아니라 실시간 상태 변화가 있는 앱에 적합합니다.
- 스케줄러 앱은 사용자가 실제로 세션이나 타이머를 시작한 뒤에만 Live Update를 쓰는 것이 맞습니다.
- Android Live Update 자체는 Android 네이티브 알림 구현이 필요합니다.
