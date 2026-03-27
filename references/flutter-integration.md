# Flutter 통합 구조 가이드

이 문서는 `C:\Users\sh953\Documents\code\live-update` 의 Android 테스트 앱 구조를 참고해, 이를 Flutter 앱에서 재사용 가능한 형태로 옮길 때의 기준을 정리한다.

## 권장 아키텍처

Flutter 앱에서는 역할을 아래처럼 나누는 것이 가장 단순하다.

### Dart 레이어

- 진행 상태 계산
- 현재 단계 / 남은 시간 / 진행률 관리
- Live Update 시작 / 업데이트 / 취소 요청
- 화면 UI 상태 표시

### Android 네이티브 레이어

- notification channel 생성
- `Notification.Builder` 또는 `NotificationCompat.Builder` 선택
- `setShortCriticalText(...)` 적용
- `setRequestPromotedOngoing(true)` 적용
- `notify(...)` / `cancel(...)`
- diagnostics 수집

## 테스트 앱에서 가져갈 핵심 패턴

Android 테스트 앱의 핵심은 사실상 아래 두 파일이다.

- `MainActivity.kt`
- `LiveUpdateNotificationManager.kt`

Flutter 앱으로 옮길 때는 이 구조를 이렇게 바꾸면 된다.

### Flutter 쪽

- `LiveUpdateService` 또는 `AndroidLiveUpdateBridge`
- 상태 모델 예: `DeliveryLiveState`, `TransitLiveState`, `SchedulerLiveState`

### Android 쪽

- `LiveUpdateNotificationManager`
- `MethodChannel` 핸들러

즉, 테스트 앱의 버튼 기반 호출을 Flutter에서 MethodChannel 호출로 바꾸는 구조다.

## 권장 MethodChannel API

채널 이름 예시:

```kotlin
const val LIVE_UPDATE_CHANNEL = "app/live_update"
```

권장 메서드:

- `initializeLiveUpdate`
- `postLiveUpdate`
- `updateLiveUpdate`
- `cancelLiveUpdate`
- `getLiveUpdateDiagnostics`
- `canPostPromotedNotifications`

### Dart 호출 예시

```dart
import 'package:flutter/services.dart';

class AndroidLiveUpdateBridge {
  static const _channel = MethodChannel('app/live_update');

  static Future<void> initialize() async {
    await _channel.invokeMethod('initializeLiveUpdate');
  }

  static Future<void> post({
    required String title,
    required String body,
    required String chipText,
    required int progress,
    bool ongoing = true,
    int notificationId = 1001,
  }) async {
    await _channel.invokeMethod('postLiveUpdate', {
      'title': title,
      'body': body,
      'chipText': chipText,
      'progress': progress,
      'ongoing': ongoing,
      'notificationId': notificationId,
    });
  }

  static Future<void> cancel({int notificationId = 1001}) async {
    await _channel.invokeMethod('cancelLiveUpdate', {
      'notificationId': notificationId,
    });
  }
}
```

### Kotlin 핸들러 예시

```kotlin
MethodChannel(flutterEngine.dartExecutor.binaryMessenger, "app/live_update")
    .setMethodCallHandler { call, result ->
        when (call.method) {
            "initializeLiveUpdate" -> {
                liveUpdateNotificationManager.ensureChannel()
                result.success(null)
            }
            "postLiveUpdate" -> {
                val title = call.argument<String>("title").orEmpty()
                val body = call.argument<String>("body").orEmpty()
                val chipText = call.argument<String>("chipText").orEmpty()
                val progress = call.argument<Int>("progress") ?: 0
                val ongoing = call.argument<Boolean>("ongoing") ?: true
                val notificationId = call.argument<Int>("notificationId") ?: 1001

                liveUpdateNotificationManager.postGeneric(
                    notificationId = notificationId,
                    title = title,
                    body = body,
                    chipText = chipText,
                    progress = progress,
                    ongoing = ongoing,
                )
                result.success(null)
            }
            "cancelLiveUpdate" -> {
                val notificationId = call.argument<Int>("notificationId") ?: 1001
                liveUpdateNotificationManager.cancel(notificationId)
                result.success(null)
            }
            else -> result.notImplemented()
        }
    }
```

## 상태 모델 설계 원칙

Flutter 쪽 상태 모델은 앱 도메인마다 달라도, Android 네이티브로 내려가는 공통 페이로드는 단순하게 유지하는 편이 좋다.

공통 필드 예시:

- `title`
- `body`
- `chipText`
- `progress`
- `ongoing`
- `notificationId`

### 배달 앱

- `chipText`: `8분`, `배달중`, `도착`

### 이동 / 대중교통 앱

- `chipText`: `7개소전`, `2정거장`, `도착`

### 스케줄러 앱

- `chipText`: `25분`, `3분`, `완료`

## 스케줄러 앱 패턴

스케줄러 앱에서는 일정 저장 자체가 아니라 "실행 중인 세션"에만 Live Update를 붙이는 것이 중요하다.

권장 흐름:

1. 사용자가 시작 버튼을 누른다.
2. Dart에서 종료 시각과 남은 시간 계산을 시작한다.
3. 정기 타이머나 ticker로 상태를 갱신한다.
4. 남은 시간이 변할 때마다 네이티브 Live Update를 업데이트한다.
5. 완료되면 Live Update를 종료한다.

## Diagnostics 패턴

테스트 앱에서 좋은 점은 diagnostics를 바로 보여준다는 것이다. Flutter 앱에도 비슷한 진단 API를 두는 것이 좋다.

추천 진단 값:

- active
- builtPromotable
- promoted
- builtShortCriticalText
- activeShortCriticalText

이 값이 있으면 아래를 빠르게 분리할 수 있다.

- 앱이 알림을 잘 만들었는가
- OS가 promotable로 판단했는가
- 실제로 promoted ongoing으로 승격됐는가
- 상태칩 텍스트가 빌드/활성 알림에 살아있는가

## 구현 시 주의

- Flutter는 상태 계산에 집중하고, Android는 알림 표현에 집중하게 분리한다.
- Android 16 미만에서는 일반 ongoing notification으로 자연스럽게 degrade 시킨다.
- 앱 시작 시 채널을 미리 생성한다.
- `setColorized(true)`는 피한다.
- chip text는 항상 짧게 유지한다.
