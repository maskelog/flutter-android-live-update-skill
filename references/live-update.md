# Flutter 실시간 알림용 Android 16 Live Update / 상태칩 적용 가이드

이 문서는 Flutter 앱에서 재사용 가능한 실시간 ongoing 알림 구조를 만들고, Android 16(API 36)의 Live Update(= promoted ongoing notification)와 상태칩(`setShortCriticalText`)까지 지원하도록 정리한 범용 가이드다.

배달, 이동, 운동 추적, 타이머, 스케줄러, 내비게이션, 업로드, 다운로드처럼 지속적으로 진행 상태를 보여줘야 하는 Flutter 앱에 적용한다.

## Flutter 앱에 맞는 적용 기준

다음 조건을 만족하면 이 스킬을 적용하기 좋다.

- Dart 레이어에서 진행 상태나 실시간 상태 변화를 지속적으로 계산할 수 있음
- Android 네이티브 레이어에서 ongoing notification을 안정적으로 갱신할 수 있음
- 단발성 알림이 아니라 일정 시간 동안 상태 변화가 이어짐
- 사용자 입장에서 잠금 화면이나 시스템 UI에 현재 상태가 노출될 가치가 있음

## Android Live Update 기능 설명

Android Live Update는 Android 16에서 제공되는 promoted ongoing notification 기반 기능이다.

핵심은 아래와 같다.

- 일반 Flutter 위젯 기능이 아니라 Android 시스템 알림 기능이다.
- Dart에서 상태를 계산할 수는 있지만, 실제 Live Update 표시는 Android 네이티브 알림 구현이 담당해야 한다.
- 상태칩, promoted ongoing 승격, diagnostics 확인도 Android 알림 객체를 기준으로 동작한다.

즉, Flutter 앱에서 이 기능을 쓰려면 아래 두 가지가 사실상 필요하다.

1. Android 16 기준 설정
   - `compileSdk = 36`
   - `targetSdk = 36`
2. Android 네이티브 알림 구현
   - `Notification.Builder` 또는 `NotificationCompat.Builder`
   - notification channel 생성
   - `setShortCriticalText(...)`
   - promoted ongoing 요청 처리

Flutter만 수정해서는 Android Live Update를 완성할 수 없고, Android 네이티브 쪽 구현이 반드시 있어야 한다.

대표 예시:

- 대중교통 도착 추적
- 배달 상태 추적
- 이동 / 경로 / 주행 상태 추적
- 내비게이션 진행 상태
- 운동 / 러닝 세션 상태
- 타이머 / 카운트다운
- 스케줄러 앱에서 시작 후 진행 중인 남은 시간 표시
- 파일 업로드 / 다운로드 진행률

## 스케줄러 앱 적용 패턴

스케줄러 앱에서도 이 스킬을 사용할 수 있다. 다만 단순 일정 저장이 아니라 사용자가 시작 버튼을 눌러 진행 중 상태가 된 뒤, 남은 시간을 실시간 ongoing 알림으로 보여주는 경우에 적합하다.

권장 흐름:

1. 사용자가 특정 작업 / 세션 / 루틴 / 예약 실행을 시작한다.
2. Dart 레이어에서 종료 시각과 남은 시간을 계산한다.
3. Android 네이티브 레이어에서 ongoing notification을 시작한다.
4. 상태칩에는 남은 시간을 아주 짧게 넣는다.
   - 예: `25분`, `09:42`, `3분`
5. 진행 중에는 본문 또는 제목에 현재 작업명을 넣고, 상태칩에는 남은 시간만 유지한다.
6. 완료되면 Live Update를 종료하고 일반 완료 알림 또는 세션 완료 상태로 전환한다.

스케줄러 앱 예시 상태:

```kotlin
enum class SchedulerLiveState(
    val title: String,
    val body: String,
    val chipText: String,
    val progress: Int,
) {
    RUNNING("집중 세션 진행 중", "남은 시간을 확인하세요.", "25분", 10),
    NEAR_END("곧 종료", "세션 종료가 가까워졌습니다.", "3분", 85),
    DONE("세션 완료", "예약된 작업이 끝났습니다.", "완료", 100),
}
```

주의:

- 스케줄 자체만 저장된 상태에서는 Live Update를 띄우지 않는 것이 맞다.
- 사용자가 실제로 시작한 실행 단위에만 ongoing notification을 연결한다.
- 상태칩은 남은 시간처럼 즉시 해석 가능한 값만 유지한다.

## 핵심 개념

Android 16 Live Update가 제대로 보이려면 아래 3가지를 함께 만족해야 한다.

1. 알림이 ongoing 상태여야 함
2. promoted ongoing 승격 요청이 들어가야 함
3. OS가 승격 가능한 알림이라고 판단해야 함

상태칩은 그 위에 추가로 아래가 필요하다.

- `setShortCriticalText("5분")`
- 짧고 즉시 이해 가능한 텍스트

## 언제 쓰면 좋은가

추천 시나리오:

- 배달 도착까지 남은 시간
- 버스 / 지하철 / 택시 추적
- 주행 / 러닝 / 운동 진행 상태
- 타이머 / 카운트다운
- 파일 업로드 / 다운로드 진행 상황

비추천 시나리오:

- 단발성 알림
- 장기 보관용 일반 공지
- 실시간성이 약한 이벤트

## 필수 준비 사항

### SDK / 라이브러리

- `compileSdk = 36`
- `targetSdk = 36`
- `androidx.core:core-ktx`는 Android 16 notification API를 지원하는 버전 사용
  - 실전에서는 `1.17.0+` 권장

### Manifest 권한

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.POST_PROMOTED_NOTIFICATIONS" />
```

설명:

- `POST_NOTIFICATIONS`: Android 13+ 일반 알림 권한
- `POST_PROMOTED_NOTIFICATIONS`: Android 16 Live Update 승격용 권한 선언

## 가장 중요한 포인트: 채널을 앱 시작 시 미리 생성

이 부분이 빠지면 앱 설정의 알림 메뉴에서 "실시간 정보(Live Updates)" 토글이 안 보일 수 있다.

즉, 서비스 시작 시점이 아니라 앱 시작 시점에 Live Update 채널을 먼저 생성하는 것이 안전하다.

```kotlin
private const val LIVE_UPDATE_CHANNEL_ID = "live_updates"

fun Context.ensureLiveUpdateChannel() {
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) return

    val manager = getSystemService(NotificationManager::class.java)
    val channel = NotificationChannel(
        LIVE_UPDATE_CHANNEL_ID,
        "Live updates",
        NotificationManager.IMPORTANCE_DEFAULT,
    ).apply {
        description = "Shows real-time ongoing progress and status chip."
        lockscreenVisibility = Notification.VISIBILITY_PUBLIC
        setShowBadge(false)
    }

    manager.createNotificationChannel(channel)
}
```

권장 위치:

- `Application.onCreate()`
- 또는 `MainActivity.onCreate()` 초기화 루틴

## 런타임 권한 / 승격 가능 여부 확인

```kotlin
fun Context.canPostNotifications(): Boolean {
    return if (Build.VERSION.SDK_INT >= 33) {
        checkSelfPermission(Manifest.permission.POST_NOTIFICATIONS) ==
            PackageManager.PERMISSION_GRANTED
    } else {
        true
    }
}

fun Context.canPostPromotedNotifications(): Boolean {
    return NotificationManagerCompat.from(this).canPostPromotedNotifications()
}
```

확인 포인트:

- `POST_NOTIFICATIONS` 런타임 허용 여부
- `canPostPromotedNotifications()` 결과
- 기기 설정에서 앱별 Live Update 허용 여부

## Live Update 알림 작성 템플릿

```kotlin
@Suppress("NewApi")
fun Context.buildLiveUpdateNotification(
    title: String,
    body: String,
    chipText: String,
    progress: Int,
    contentIntent: PendingIntent,
): Notification {
    val style = Notification.ProgressStyle()
        .setProgress(progress)
        .setStyledByProgress(true)
        .setProgressSegments(
            listOf(
                Notification.ProgressStyle.Segment(progress).setColor(0xFF1976D2.toInt()),
                Notification.ProgressStyle.Segment((100 - progress).coerceAtLeast(0))
                    .setColor(0xFFE0E0E0.toInt()),
            ),
        )
        .setProgressPoints(
            listOf(
                Notification.ProgressStyle.Point(1).setColor(0xFF4CAF50.toInt()),
                Notification.ProgressStyle.Point(99).setColor(0xFFFF5722.toInt()),
            ),
        )

    return Notification.Builder(this, LIVE_UPDATE_CHANNEL_ID)
        .setSmallIcon(R.drawable.ic_notification)
        .setContentTitle(title)
        .setContentText(body)
        .setContentIntent(contentIntent)
        .setOnlyAlertOnce(true)
        .setOngoing(true)
        .setCategory(Notification.CATEGORY_PROGRESS)
        .setVisibility(Notification.VISIBILITY_PUBLIC)
        .setStyle(style)
        .setProgress(100, progress, false)
        .setShortCriticalText(chipText)
        .setRequestPromotedOngoing(true)
        .build()
}
```

게시:

```kotlin
fun Context.postLiveUpdate(notificationId: Int, notification: Notification) {
    NotificationManagerCompat.from(this).notify(notificationId, notification)
}
```

## 상태칩 텍스트 작성 규칙

상태칩 텍스트는 아주 짧고 즉시 이해 가능해야 한다.

좋은 예:

- `5분`
- `2정거장`
- `도착`
- `배송중`
- `3km`

주의:

- 너무 길면 잘리거나 표시 우선순위가 떨어질 수 있음
- 설명문 대신 핵심 상태만 넣는 것이 좋음
- 제목/본문에 상세 설명을 중복 제공하는 것이 안전함

예시 상태 모델:

```kotlin
enum class LiveState(
    val title: String,
    val body: String,
    val chipText: String,
    val progress: Int,
) {
    START("주행 시작", "목적지까지 이동 중입니다.", "시작", 5),
    MID("목적지까지 5분", "현재 정상 이동 중입니다.", "5분", 60),
    NEAR("곧 도착", "잠시 후 도착합니다.", "도착임박", 90),
    DONE("도착 완료", "이동이 완료되었습니다.", "도착", 100),
}
```

## 실전에서 놓치기 쉬운 함정

### `setColorized(true)` 사용 금지

`colorized` 알림은 Android 16에서 Live Update / 상태칩 승격 자격을 잃을 수 있다.

```kotlin
.setColorized(true) // 사용하지 않는 것을 권장
```

### 채널을 늦게 만들면 "실시간 정보" 토글이 안 보일 수 있음

- 서비스가 시작될 때만 채널 생성
- 사용자가 앱 설정에 먼저 진입

이 순서면 OS가 promoted channel을 아직 모르기 때문에 토글이 안 뜰 수 있다.
