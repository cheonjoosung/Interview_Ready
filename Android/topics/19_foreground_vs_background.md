### Foreground vs Background
- Foreground
  + 사용자가 보고/터치 중인 앱
  + onResume() 상태
  + 시스템 자원에 대한 제한 없음 (네트워크, CPU, 센서 등)
  + ⚡ 빠른 응답성 필요 (ANR 발생시 사용자 즉시 인지)
  + 특징
    * 시스템 자원을 자유롭게 사용 가능 (CPU, 네트워크, 위치, 센서 등)
    * 사용자의 주의가 앱 집중되어 있어야 하므로 **빠른 응답이 요구**됨
    * 이 상태에서는 백그라운드의 제한이 적용되지 않음

- Background
  + 화면에는 보이지 않지만 메모리 상 실행 중
  + onPause() → onStop()
  + Android 8.0(API 26) 이후 백그라운드 제한 강화
    * 백그라운드 서비스 실행 불가 → Foreground Service 또는 WorkManager 필요
    * 백그라운드 위치 권한 분리 (ACCESS_BACKGROUND_LOCATION)
  + 특징
    * 시스템 자원을 **제한적으로 사용 가능**
    * Android 8.0(API 26) 이후부터 백그라운드 작업에 제한 강화
    * 백그라운드에서 일정시간 지나면 앱이 종료될 수 있음
    * 서비스 또는 브로드캐스트를 사용할때도 제한이 있음

- Foreground Service
  + 앱이 Background 상태여도 계속 실행 보장
  + Notification 필수 표시 (사용자 인지 가능)
  + 예시: 음악 재생, 네비게이션 안내, 운동 기록
  + 시스템이 잘 종료하지 않음
  + 사용 이유
    * 일반 서비스는 백그라운드에서 쉽게 종료될 수 있음
    * Foreground Service는 시스템이 우선순위를 높게 잡고, 종료시키지 않음
  + 구현흐름
    * ```kotlin
      val notification = NotificationCompat.Builder(this, "channel_id")
          .setContentTitle("서비스 실행 중")
          .build()

      startForeground(1, notification)  // 반드시 알림과 함께 실행       
      ```

- 시스템 제한사항 (Android 8.0 이후)
  + ⚠️ Background 실행 제한
    * 오래된 방식의 Background Service 실행이 제한 됨
    * 해결방법 : Foreground Service, JobScheduler, WorkManager 사용
  + 🔐 Background 위치 권한
    * 백그라운드 GPS, BLE 같은 민감한 자원 접근 시 권한 강화
    * Android 10 부터는 ACCESS_BACKGROUND_LOCATION 별도 요청 필요

- 앱 상태 구분하는 기준
  + | 시나리오      | 상태 변화                                                    |
    |--------------|-----------------------------------------------------------|
    | 앱 실행       | `onCreate()` → `onStart()` → `onResume()` → **Foreground** |
    | 홈 버튼 누름   | `onPause()` → `onStop()` → **Background**                  |
    | 앱 다시 열기   | `onRestart()` → `onStart()` → `onResume()` → **Foreground**|
    | 앱 종료       | `onPause()` → `onStop()` → `onDestroy()`                   |


- 면접 관련 질문
  + Foreground Service를 쓰면 무조건 종료 안 되나요?
    * 아닙니다. 시스템이 메모리가 극도로 부족할 경우 Foreground Service도 종료될 수 있습니다.
    * 다만 우선순위가 가장 높기 때문에 마지막에 종료됨.
  + Android 8.0 이후 Background 서비스 실행이 제한된 이유는?
    * 불필요한 백그라운드 실행 → 배터리 소모 + 성능 저하
    * 해결: JobScheduler, WorkManager, Foreground Service 활용
  + Background에서 위치 권한 접근이 어떻게 달라졌나요?
    * Android 10(API 29) 이상: ACCESS_BACKGROUND_LOCATION 별도 권한 필요
    * 사용자 동의 없이는 백그라운드에서 GPS 추적 불가
  + 앱이 Cached 상태일 때 어떤 일이 발생하나요?
    * 앱은 실행 중이 아니지만, 메모리에 남아있음
    * 재실행 시 cold start 대신 warm start → 실행 속도 빠름
    * 메모리가 부족하면 시스템이 Cached 앱부터 제거 

- 면접 질문
  + 앱이 Foreground 상태에서 Background 상태로 전환될 때, 시스템은 어떤 기준으로 앱의 우선순위를 결정하고 리소스를 제한하나요?
    * 시스템은 사용자의 경험과 배터리 수명, 시스템 성능을 최적화하기 위해 앱의 우선순위를 결정합니다.
  + Android 8.0 (API 26) 이상에서 Background 실행 제한이 강화된 주된 이유는 무엇이며, 이로 인해 개발자가 유의해야 할 점은 무엇인가요?
    + 강화된 주된 이유는 무분별한 Background 작업으로 인한 배터리 소모를 줄이기와 Background 앱들이 CPU, 메모리 등의 시스템 리소스를 과도하게 점유하여 전반적인 시스템 성능을 저하시키는 것을 방지하기 위함입니다.
    + 유의해야 될 점으로는 startService()를 사용하여 Background에서 자유롭게 Service를 실행하는 것이 제한됩니다. 앱이 Background 상태로 전환되면 몇 분 내에 시스템이 Service를 중지시켜버리기 때문입니다.



---
