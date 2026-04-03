### Service
- 정의
  + UI 없이 백그라운드에서 동작하는 컴포넌트
  + Activity와 독립적인 생명주기 가짐
  + 주로 오래 걸리는 작업, 장기 실행 작업, 앱 간 기능 제공 시 사용

- Service 종류
  + | 구분                     | 설명                                              | 사용 예시                           |
    | ---------------------- | ----------------------------------------------- | ------------------------------- |
    | **Started Service**    | `startService()`로 시작 → `onStartCommand()` 실행    | 음악 재생, 파일 다운로드                  |
    | **Bound Service**      | `bindService()`로 시작 → 클라이언트와 **IPC(프로세스 간 통신)** | 앱-서비스 간 데이터 교환, Messenger, AIDL |
    | **Foreground Service** | 알림(Notification)을 동반해 실행, 시스템이 잘 안 죽임           | 음악 플레이어, 위치 추적, 피트니스 트래킹        |
    | **Background Service** | UI 없이 실행되나, 최신 OS에서 제한 많음                       | (거의 사용 안 권장, WorkManager로 대체)   |
 
 - Service 생명주기
   + Started Service
     * ```text
       onCreate() → onStartCommand() → (실행 중) → onDestroy()
       ``` 
   + Bound Service 
     * ```text
       onCreate() → onBind() → (클라이언트와 통신) → onUnbind() → onDestroy()
       ```  

- Service 주요 메서드
  + | 메서드                | 설명                                   |
    | ------------------ | ------------------------------------ |
    | `onCreate()`       | 서비스 최초 생성 시 1회 호출                    |
    | `onStartCommand()` | `startService()` 호출 시 실행, 작업 시작      |
    | `onBind()`         | `bindService()` 호출 시 실행, 통신 인터페이스 반환 |
    | `onUnbind()`       | 마지막 클라이언트 연결 해제 시 호출                 |
    | `onDestroy()`      | 서비스 종료 시 호출                          |
 
- 서비스 실행 시 주의사항
  + 메인 스레드에서 동작 → 무거운 작업 직접 실행 시 ANR 발생 가능
    * HandlerThread, Executor, Coroutine, IntentService 사용
  + 안드로이드 8.0(Oreo) 이상 제한
    * 백그라운드에서 임의 실행 불가
    * 반드시 Foreground Service 또는 JobScheduler / WorkManager 사용 필요
  + 배터리 최적화
    * Doze 모드, 앱 standby 모드에서 서비스가 제한될 수 있음
    * 중요한 경우 setForegroundServiceType 활용 

- 실무 활용 예시
  + 음악 앱: Foreground Service + MediaSession API
  + 메신저: Bound Service (앱 ↔ 서비스 양방향 통신)
  + 위치 추적: Foreground Service + Notification
  + 데이터 동기화: WorkManager (서비스 대체)

- 면접 질문 포인트
  + Service와 Thread 차이는 무엇인가요?
    * Service는 안드로이드 컴포넌트로 생명주기 관리
    * Thread는 자바 실행 단위일 뿐, 앱 종료 시 같이 사라짐
    * Service 내부에서 별도의 Thread를 사용해야 함
  + Foreground Service는 왜 알림(Notification)을 강제하나요?
    * 사용자에게 명시적으로 실행 중임을 알려야 하기 때문
    * 배터리 소모, 개인 정보 추적 같은 민감 작업에 투명성 보장
  + 안드로이드 8.0 이상에서 백그라운드 서비스가 제한된 이유는?
    * 배터리 수명, 시스템 성능 최적화를 위해
    * 대체 기술: WorkManager, JobScheduler, Foreground Service


---


## 4대 컴포넌트 Activity
- 정의
  + 앱에서 화면 하나를 담당하는 컴포넌트
  + 사용자와 직접 상호작용(UI)
  + UI 렌더링 + 이벤트 처리 + 앱 흐름 제어

- 특징
  + | 항목                 | 내용                                            |
    | ------------------ | --------------------------------------------- |
    | **UI 구성 단위**       | 앱의 화면 1개 = Activity 1개                        |
    | **생명주기 존재**        | `onCreate()` \~ `onDestroy()`                 |
    | **호출 방식**          | 명시적/암시적 Intent                                |
    | **Manifest 등록 필수** | `AndroidManifest.xml`에 선언 필요                  |
    | **백스택 관리**         | `startActivity()` 호출 시 이전 Activity는 백스택에 push |
    | **Context 역할**     | 리소스 접근, 시스템 서비스 호출, 다른 컴포넌트 실행 가능             |

- Intent
  + 명시적(Intent Explicit)
    * ```kotlin
      val intent = Intent(this, DetailActivity::class.java)
      startActivity(intent)
      ```
  + 암시적(Intent Implicit)
    * ```kotlin
      val intent = Intent(Intent.ACTION_VIEW, Uri.parse("http://google.com"))
      startActivity(intent) // OS가 적절한 앱을 Finder
      ```
    * 링크를 띄울 수 있는 액션을 OS에서 받아 하단 Chooser 브라우저 목록 표시

- Activity vs Fragment
  + | 항목    | Activity | Fragment          |
    | ----- | -------- | ----------------- |
    | 실행 단위 | 독립 실행 가능 | Activity 내부에서만 실행 |
    | 생명주기  | 독립적      | Activity에 종속      |
    | 목적    | 전체 화면 관리 | 부분 화면, 동적 UI      |
    | 재사용성  | 낮음       | 높음                |
    | 관리 주체 | OS       | FragmentManager   |
  + 👉 Fragment 장점
    * 메모리 효율성 (View만 교체 가능)
    * 재사용성 ↑ (다양한 Activity에서 호출 가능)
    * 동적 UI 대응 (뷰페이저, 탭 등)  
 
- Activity 메모리 누수 발생 상황
  + 전역 static Context 보관
  + Handler / Runnable 내부 클래스 참조
  + 비동기 콜백 (Retrofit, RxJava) 완료 전 Activity 종료
  + ViewBinding 해제 누락 (onDestroyView()에서 해제 필요)
  + 👉 해결 방법
    * WeakReference 사용
    * lifecycleScope, LiveData 활용
    * onDestroy()에서 해제 

- Activity 화면 회전 시 상태 보존
  + 기본 동작: Activity 재생성
  + 해결 방법
    * onSaveInstanceState() → Bundle 저장
    * onCreate() or onRestoreInstanceState() → 복원
    * ViewModel (구성 변경에도 데이터 유지)
  
- 면접 관련 질문
  + Activity 간 데이터 전달 방법?
    * Activity 간 데이터는 주로 Intent의 Extra를 통해 전달
    * 복잡한 데이터는 Serializable 또는 Parcelable 객체를 사용하며, 경우에 따라 Bundle을 통해 전달
  + Activity & Fragment 차이
    * | 항목      | Activity | Fragment              |
      | ------- | -------- | --------------------- |
      | 실행 단위   | 독립 실행 가능 | Activity 내부에서만 실행     |
      | 생명주기    | 독립적      | Activity 생명주기에 종속     |
      | 사용 목적   | 전체 화면 구성 | 부분 화면, 동적 UI 구성       |
      | 재사용성    | 낮음       | 높음                    |
      | Manager | 시스템이 관리  | `FragmentManager`가 관리 |
    * Fragment 사용 시 장점
      1. 메모리 효율성 높음 (View 만 교체 가능 → 가벼움)
      2. 화면 재사용 용이 (여러 Activity 에서 호출 가능)
    * Activity는 UI뿐 아니라 자체 Window와 Context를 포함하기에 시스템 차원에서 더 많은 초기화 비용이 발생.
    * 반면 Fragment는 Activity 위에서 동작하며, View와 Lifecycle만 별도로 관리하므로 생성 비용이 훨씬 가벼움
  + Activity 에서 메모리 누수 발생 상황?
    * "Activity는 View, Context 등을 포함하고 있어서, 생명주기와 무관한 참조가 남아 있으면 GC되지 않아 메모리 누수 발생할 수 있음
      1. 익명 클래스 Handler
      2. 전역 static 변수에 Context 저장
      3. 비동기 작업 (예: Retrofit, RxJava) 완료 전에 Activity가 종료됨
      4. ViewBinding을 onDestroy에서 해제하지 않음 (Fragment도 포함)
    * 해결 방법
      1. WeakReference 사용
      2. lifecycleScope, viewLifecycleOwner.lifecycleScope 활용
      3. onDestroy()에서 리소스 해제
  + Activity 화면 회전 시 상태 보존 방법?
    * 화면 회전 시 Activity는 재생성되므로, 
      1. **onSaveInstanceState()**를 사용해 데이터를 임시 저장, 
      2. **onCreate() 또는 onRestoreInstanceState()**에서 복원
    * ViewModel 사용하여, 구성 변경(Configuration Change) 시에도 데이터를 유지


---


## 4대 컴포넌트 BroadcastReceiver
- 정의
  + 시스템 또는 앱에서 발생하는 브로드캐스트 이벤트를 수신하고, 이에 대응하는 코드를 실행하는 컴포넌트
    * 안드로이드 시스템은 다양한 **이벤트(알람, 네트워크 변화, 부팅 완료 등)**를 브로드캐스트로 보냄
    * 앱도 커스텀 브로드캐스트를 만들어 보낼 수 있음
    
- 특징
  + | 항목    | 설명                                |
    | ----- | --------------------------------- |
    | 실행 방식 | 콜백 함수인 `onReceive()`가 호출되어 처리     |
    | 생명주기  | 화면 UI 없음. `onReceive()`만 실행되고 종료됨 |
    | 등록 방식 | 정적 등록 (Manifest), 동적 등록 (코드에서)    |
    | 목적    | 이벤트 감지 및 간단한 처리를 빠르게 수행           |

- 사용 방법
  + 정적 등록(Manifest)
    * ```xml
      <receiver android:name=".BootReceiver">
        <intent-filter>
           <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
      </receiver>
      ```
  + 동적 등록(코드 방식)
    * ```kotlin
      val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
          Log.d("Receiver", "Battery changed!")
        }
      }

      val filter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
      registerReceiver(receiver, filter)
      ```
      1. 앱이 실행 중일 때만 수신 가능
      2. 주로 Activity, Service 내에서 사용

- 주의사항
  + onReceive()는 UI 작업 금지 (짧고 빠른 처리만 수행해야 함)
  + 오래 걸리는 작업은 JobIntentService, WorkManager, ForegroundService 연계 필요
  + Android 8.0(Oreo) 이상에서는 Manifest 등록 제한이 많아 동적 등록 또는 백그라운드 제한 우회 필요

- 면접 관련 질문
  + Android 8.0(Oreo) 이후 백그라운드 앱의 정적 브로드캐스트 수신이 제한
    * 보안과 배터리 최적화
      1. Manifest 등록한 브로드캐스트 리시버는 일부 시스템 이벤트만 수신할 수 있게 되었고,
      2. 대부분 앱이 실행 중일 때 동적으로 등록해야 수신 가능
  + onReceive() 에서 긴 시간의 작업 실행 시?
    * 내부적으로는 앱 프로세스가 일시적으로 깨워진 상태에서 동작함
    * 긴 작업을 실행 시 **ANR(Application Not Responding)**이 발생하거나 시스템이 강제로 종료
  + BroadcastReceiver 대신 사용할 수 있는 대
    * Android 8.0 이상 JobScheduler, JobIntentService, WorkManager 같은 백그라운드 작업 매니저
    * LiveData, StateFlow, EventBus 방법 등으로 대체 가능
  + BroadcastReceiver vs Service
    * | 항목           | **BroadcastReceiver**                          | **Service**                                             |
      | ------------ | ---------------------------------------------- | ------------------------------------------------------- |
      | **목적**       | 이벤트 수신 후 짧은 작업 처리                              | 백그라운드에서 장시간 작업 수행                                       |
      | **트리거 방식**   | 외부 or 내부에서 **브로드캐스트**로 호출됨 (`sendBroadcast()`) | 명시적으로 **시작(`startService`)** 또는 바인딩(`bindService`)      |
      | **생명주기**     | `onReceive()` 단 하나, 실행 후 바로 종료                 | `onCreate()` → `onStartCommand()` → `onDestroy()` 등 존재  |
      | **UI와의 관계**  | UI 없음 / 사용자와 상호작용 X                            | UI 없음 / Activity와는 분리되어 있음                              |
      | **작업 시간 제약** | **매우 짧아야 함** (몇 초 이내) → 길면 ANR 발생              | 상대적으로 **장시간 작업 가능**                                     |
      | **실행 상태 유지** | 호출 후 자동 종료됨                                    | 명시적으로 종료되기 전까지 **계속 실행** 가능                             |
      | **멀티스레드 처리** | 자체 스레드 아님 (메인 스레드에서 동작)                        | 직접 스레드 생성 가능 (`HandlerThread`, `Coroutine`, `Thread` 등) |
      | **사용 사례**    | 네트워크 상태 변경 감지, 부팅 완료 감지, 앱 내 메시지 처리            | 파일 다운로드, 음악 재생, 알람 대기 등                                 |


---


## PendingIntent
- 정의
  + `Intent`를 다른 앱/시스템이 대신 실행할 수 있도록 위임하는 객체
  + 즉, "나중에 이 Intent 실행해줘"라고 OS나 외부 앱에 맡기는 역할
  + 사용 예시: Notification, AlarmManager, AppWidget 등

- 필요한 이유?
  + 보안 + Context 문제 해결
    * 시스템 서비스(AlarmManager, NotificationManager)나 외부 앱은 직접 내 앱의 Intent를 실행할 권한이 없음
    * PendingIntent는 "내 앱이 실행을 위임"한 것이라, 대신 실행 가능
  + 실행 주체가 바뀌어도 내 앱의 권한(Context) 그대로 유지됨

- 주요 사용 케이스
  + | 상황          | 예시                                         |
    | ----------- | ------------------------------------------ |
    | 알림 클릭 시     | Notification + PendingIntent.getActivity() |
    | 특정 시간 작업 예약 | AlarmManager + PendingIntent               |
    | 브로드캐스트 예약   | getBroadcast()                             |
    | 서비스 실행      | getService()                               |
    | 위젯 버튼 클릭    | RemoteViews.setOnClickPendingIntent()      |

- 생성 방법
  + ```kotlin
    // Activity 실행
    val intent = Intent(context, MyActivity::class.java)
    val pendingIntent = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_IMMUTABLE)

    // Broadcast 실행
    val intent = Intent(context, MyReceiver::class.java)
    val pendingIntent = PendingIntent.getBroadcast(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT)

    // Service 실행
    val intent = Intent(context, MyService::class.java)
    val pendingIntent = PendingIntent.getService(context, 0, intent, PendingIntent.FLAG_IMMUTABLE)
    ```

- 주요 Flag
  + | 플래그                   | 설명                                            |
    | --------------------- | --------------------------------------------- |
    | `FLAG_IMMUTABLE`      | **Intent 내용 변경 불가** (보안 강화, Android 12+에서 필수) |
    | `FLAG_MUTABLE`        | 시스템이 Intent 내부를 수정할 수 있도록 허용                  |
    | `FLAG_UPDATE_CURRENT` | 같은 PendingIntent가 있으면 업데이트                    |
    | `FLAG_CANCEL_CURRENT` | 기존 PendingIntent 취소 후 새로 생성                   |
    | `FLAG_NO_CREATE`      | 이미 존재하는 경우에만 반환, 없으면 null                     |
    | `FLAG_ONE_SHOT`      |  한 번만 사용되고 나면 자동으로 소멸되도록 설정                  |
  
  + (| 또는 or) 연산으로 조합 가능
    * PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_ONE_SHOT

- 실무 팁
  + Android 12(API 31) 이상에서는 FLAG_IMMUTABLE 또는 FLAG_MUTABLE 중 하나를 반드시 명시해야 함
  + PendingIntent는 시스템에 캐싱되므로 중복 생성 주의
  + Request Code가 같고, Intent 내용이 같으면 같은 PendingIntent로 간주됨
  + 보안을 위해 불필요하게 mutable한 Intent는 지양

- 면접 관련 질문
  + PendingIntent란 무엇이며 왜 필요한가요?
    * Intent를 시스템/외부 앱이 대신 실행하도록 위임하는 객체
    * 시스템 서비스가 내 앱 Context 없이 작업 실행 가능
  + getActivity(), getBroadcast(), getService() 차이는?
    * 실행 대상이 Activity냐 / BroadcastReceiver냐 / Service냐에 따라 다름
  + FLAG_IMMUTABLE vs FLAG_MUTABLE?
    * Immutable: Intent 수정 불가 (보안 ↑, 기본적으로 이걸 사용)
    * Mutable: 시스템이 내용을 변경할 필요가 있을 때만 (예: Notification inline reply)
  + 왜 Android 12 이상에서 반드시 명시해야 하나요?
    * 보안 강화 목적. PendingIntent가 의도치 않게 수정돼 악용되는 것을 방지하기 위해


---
