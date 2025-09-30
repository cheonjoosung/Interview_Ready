# 🤖 Android 가이드 : 개념 및 개발

Android 개발에 필요한 핵심 개념, 구조, 실무 적용 예시들을 정리한 문서입니다.

---

## 📘 주요 개념

### Activity 생명주기
- 정의
  + Activity 는 사용자와 화면 단위로 상호작용하는 컴포넌트
  + 사용자가 앱 실행/나가기/회전 등 에 따라 Activity는 여러 상태를 거치며 전환
  + 액티비티 생명주기에 대한 이해를 통해 리소스 누수나 사용자 경험 저하 방지 필요

- 상세설명
  + | 메서드             | 호출 시점                          | 주로 하는 일                       |
    | --------------- | ------------------------------ | ----------------------------- |
    | **onCreate()**  | Activity 최초 생성 시               | 뷰 바인딩, 초기 변수 설정, 리소스 할당       |
    | **onStart()**   | Activity가 화면에 보이기 시작할 때        | UI 준비 (아직 포커스는 없음)            |
    | **onResume()**  | Activity가 포커스를 얻고 사용자와 상호작용 가능 | 애니메이션 시작, 센서 리스너 등록           |
    | **onPause()**   | 다른 Activity가 앞에 나올 때 (부분 가림)   | 리소스 해제 (예: 센서, 카메라 등)         |
    | **onStop()**    | 완전히 화면에서 보이지 않을 때              | DB 저장, 리소스 정리, Broadcast 해제 등 |
    | **onRestart()** | 정지되었다가 다시 보여질 때                | 필요한 UI 갱신                     |
    | **onDestroy()** | Activity 종료되거나 시스템에 의해 제거될 때   | 모든 리소스 해제, GC 대상화             |

- 생명주기 상태 예시
  +  앱 시작 시 `onCreate()` → `onStart()` → `onResume()`
  +  다른 앱 열어서 가릴 때 `onPause()` → `onStop()`
  +  다시 돌아왔을 때 `onRestart()` → `onStart()` → `onResume()`
  +  앱 완전 종료 또는 시스템에 의한 제거 `onPause()` → `onStop()` → `onDestroy()`

- 핵심 포인트
  + `onCreate()` - UI 설정
  + `onPause()` - 리소스(센서, GPS, 애니메이션) 해제
  + `onStop()` - 저장이 필요한 데이터 처리
  + 화면 회전 시 
    * `onPause()` → `onStop()` → `onSaveInstanceState` → `onDestroy()`
    * →`onCreate()` → `onStart()` → `onRestoreInstanceState()` → `onResume()`
    * 왜 액티비티는 재생성 할까?
      1. 레이아웃을 다시 계산해야하는데 기존을 파괴하고 새로 만드는 것이 깔끔한 방법
      2. 이전 상태 유지시 레이아웃도 꼬이고 메모리 누수 위험이 커짐

- Cold/Warm/Hot Start 
  + | 구분             | 정의                       | Activity 생명주기 흐름                           | 특징                               |
    | -------------- | ------------------------ | ------------------------------------------ | -------------------------------- |
    | **Cold Start** | 앱이 완전히 종료된 상태에서 처음 실행됨   | `onCreate()` → `onStart()` → `onResume()`  | 리소스 로딩, Application 객체 생성, 가장 느림 |
    | **Warm Start** | 앱이 백그라운드에 있다가 다시 돌아옴     | `onRestart()` → `onStart()` → `onResume()` | Activity는 살아있고, 앱 프로세스도 유지됨      |
    | **Hot Start**  | 화면 꺼짐, 잠깐 홈으로 나갔다가 다시 복귀 | `onResume()` 단독                            | UI도 메모리에 있고, 거의 즉시 복구됨           |
  + Cold Start 최적화
    * Application 에서 무거운 초기화 지양
    * SplashActivity 최소화
    * 비동기 초기화 (WorkManager, Coroutine, Executors)
  + Warm Start 대응 : 재개 시 필요한 데이터만 빠르게 로딩
  + Hot Start 대응 : Resume 시 불필요한 연산 배제

- 실무에서 자주 하는 실수
  + onPause()에서 오래 걸리는 작업(DB write, 네트워크 요청) → UI 멈춤 현상 발생
    * → onStop()이나 백그라운드 스레드로 처리해야 함
  + onDestroy()는 항상 보장되지 않음 (시스템이 메모리 부족 시 프로세스 kill 가능)
    * → 반드시 중요한 데이터는 onPause() 또는 onStop()에서 저장해야 함
  + 화면 회전 시 ViewModel + LiveData 사용 안 하고 매번 DB 재쿼리 → 성능 저하

- 면접 관련 질문
  + onPause()와 onStop() 차이는?
    * onPause() → Activity가 부분적으로 가려짐 (다른 Activity 투명 다이얼로그 등)
    * onStop() → Activity가 완전히 화면에서 안 보임
  + 왜 화면 회전 시 Activity를 재생성할까?
    * UI 리소스(layout-land) 적용 필요
    * 메모리 안정성 (상태 꼬임 방지)
    * 새 인스턴스에서 클린하게 다시 그림
  + Q3. Cold / Warm / Hot Start 차이 설명해보세요.
    * Cold: 프로세스 없음 → onCreate부터 (가장 느림)
    * Warm: 백그라운드 프로세스 존재 → onRestart부터
    * Hot: Activity만 잠깐 멈춤 → onResume만


---


### Fragment 생명주기
- 정의
  + Fragment는 자체적인 생명주기를 가지는 UI 구성요소로, Activity 내에서 모듈식 UI를 구축하는 데 사용 됩니다.
  + Fragment의 생명주기를 이해하는 것은 리소스를 올바르게 관리하고 사용자 경험을 향상시키는 데 매우 중요합니다.

- Fragment 생명주기 콜백 정리
  + | 메서드                       | 호출 시점                    | 주요 역할                                      |
    | ------------------------- | ------------------------ | ------------------------------------------ |
    | **onAttach()**            | Fragment가 Activity에 붙을 때 | `Context` 획득, Activity와 통신 준비              |
    | **onCreate()**            | Fragment 객체 생성 시         | 변수 초기화, View와 직접 관련 없는 로직 수행               |
    | **onCreateView()**        | UI 레이아웃을 처음 생성할 때        | `LayoutInflater`로 XML → View 생성            |
    | **onViewCreated()**       | 뷰가 완전히 생성된 직후            | View Binding, LiveData 관찰, RecyclerView 세팅 |
    | **onStart()**             | Fragment가 사용자에게 보이기 시작   | UI 표시 준비                                   |
    | **onResume()**            | 포커스를 얻고 사용자와 상호작용 가능     | 애니메이션 시작, Listener 등록                      |
    | **onPause()**             | 다른 UI가 덮을 때              | 애니메이션 중지, 간단한 저장                           |
    | **onStop()**              | 화면에서 완전히 안 보일 때          | 무거운 리소스 해제 (카메라, 위치 등)                     |
    | **onSaveInstanceState()** | 예기치 않게 소멸될 가능성 있을 때      | Bundle에 상태 저장                              |
    | **onDestroyView()**       | UI 뷰가 제거될 때              | View Binding null 처리, Adapter 해제           |
    | **onDestroy()**           | Fragment 자체가 소멸될 때       | 전체 리소스 해제, 스레드 중단                          |
    | **onDetach()**            | Activity와 연결 끊길 때        | Context 참조 해제                              |

- Activity vs Fragment 비교 포인트
  + Activity: 앱 전체 화면 단위
  + Fragment: Activity 내부의 UI 구성요소 단위
  + Activity 생명주기 안에서 Fragment 생명주기가 함께 움직임
  + Fragment는 뷰와 논리 로직의 생명주기가 분리되어 있다는 점이 중요함
    * onCreate()는 Fragment 객체 (논리 로직)
    * onCreateView() ~ onDestroyView()는 UI(View) 관련

- Cold / Warm / Hot Start 관점에서
  + Cold Start
    * `onAttach()` → `onCreate()`→ `onCreateView()`→ `onViewCreated()`→ `onStart()`→ `onResume()`
  + Warm Start
    * UI는 살아있지만 다시 화면에 보일 때
    * `onStart()`→ `onResume()`
  + Hot Start
    * 이미 화면에 떠 있는데 다시 복귀
    * `onResume()` 단독

- 면접 관련 질문
  + Fragment 생명주기에서 onCreate()와 onCreateView() 차이는?
    * onCreate()는 Fragment 객체 자체 초기화 (뷰 관련 X)
    * onCreateView()는 Fragment의 UI(View) 생성
  + Fragment에서 View Binding은 어디서 해제해야 하나요?
    * onDestroyView()에서 null 처리해야 함 (메모리 누수 방지)
  + Activity와 Fragment 생명주기의 가장 큰 차이는?
    * Fragment는 뷰 생명주기(View Lifecycle) 와 객체 생명주기(Fragment Lifecycle) 가 분리
    * 따라서 View 관련 로직은 onViewCreated(), 리소스 해제는 onDestroyView()에서 수행


---


### Content Provider
- 정의
  + 안드로이드 4대 컴포넌트 중 하나
  + 앱 간 데이터 공유 표준 인터페이스
  + SQLite, 파일, 네트워크 등 내부 저장소 구조를 숨기고 통일ㅚㄴ CRUD API로 제공

- 주요 역할
  + 데이터 공유: 다른 앱이 표준 인터페이스로 데이터 접근
  + 데이터 캡슐화: 내부 저장 방식(파일/DB 등) 노출 X
  + 접근 제어: 퍼미션 기반 권한 관리 (READ_CONTACTS, WRITE_CONTACTS 등)
  + CRUD 제공: query, insert, update, delete, getType 
  
- 핵심 구성 요소
  + Content URI
    * content://<authority>/<path>/<id>
      1. authority: ContentProvider의 고유 이름 (AndroidManifest에 등록)
      2. path: 테이블/데이터 종류
      3. id(optional): 특정 row 식별
    * content://contacts/people/3
      1. contacts authority의 people 테이블에서 ID=3 레코드 
  + ContentResolver
    * 다른 앱에서 Content Provider 접근 시 사용하는 클라이언트 API 
    * ```kotlin
      val cursor = contentResolver.query(
        ContactsContract.Contacts.CONTENT_URI,
        null, null, null, null
      )
      ```
    * 앱은 직접 Provider에 접근하지 않고 ContentResolver → ContentProvider 경유
 
- 표준 CRUD 메서드
  + | 메서드         | 역할                   |
    | ----------- | -------------------- |
    | `query()`   | 데이터 조회 (`Cursor` 반환) |
    | `insert()`  | 새로운 데이터 추가           |
    | `update()`  | 기존 데이터 수정            |
    | `delete()`  | 데이터 삭제               |
    | `getType()` | 특정 URI의 MIME 타입 반환   |
  
- 실무 예시
  + 안드로이드 기본 앱
    * 연락처 (ContactsContract)
    * 캘린더 (CalendarContract)
    * 미디어 파일 (MediaStore)
  + 우리 앱에서 직접 구현
    * DB 데이터를 외부 앱과 공유해야 할 때
    * 예: 메신저 앱이 content://messenger/chat/1 URI로 채팅 데이터 노출

- 비교
  + | 항목 | Activity/Fragment | ContentProvider |
    | -- | ----------------- | --------------- |
    | 목적 | UI 제공             | 데이터 공유          |
    | 실행 | 명시적/암시적 Intent    | Content URI     |
    | 권한 | Intent-filter 기반  | 퍼미션 기반 접근 제어    |
 

- 표준 CRUD 인터페이스
  + 데이터를 조작하기 위한 표준 메서드 집합을 제공합니다.
    * query() : 데이터 검색
    * insert() : 새 데이터 삽입
    * update() : 기존 데이터 수정
    * delete() : 데이터 삭제
    * getType() : 특정 URI에 해당하는 데이터의 MIME 타입을 반환

- 콘텐트 URI
  + Content Provider 내의 특정 데이터를 식별하기 위한 고유한 URI 체계를 사용합니다. 
  + content:// : 스킴, 콘텐츠 프로바이더임을 나타냅니다.
  + <authority>: 콘텐츠 프로바이더를 식별하는 고유한 이름
  + <path>: 프로아비더가 제공하는 데이터의 종류를 나타내는 경로
  + <optional_id>: 특정 레코드를 식별하기 위한 ID

- 면접 관련 질문
  + Content Provider는 언제 직접 구현하나요?
    * 내 앱 데이터를 다른 앱과 공유할 필요가 있을 때만
    * 자기 앱 내부에서만 쓰면 Room/DB/Repository로 충분
  + ContentResolver와 직접 DB 접근 차이는?
    * ContentResolver는 앱 간 표준 인터페이스
    * DB 접근은 내부 전용
    * 즉, ContentProvider는 앱 간 데이터 추상화 계층
  + Content Provider 성능 이슈 방지 방법은?
    * query는 반드시 백그라운드 스레드에서 처리
    * Cursor는 다 쓴 후 close()
    * 필요한 최소한의 데이터만 projection


---


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


### Bundle
- 정의
  + Key-Value 기반 데이터 저장 객체
  + Parcelable 구현 → 프로세스 간 통신(IPC)도 가능
  + Android의 Activity, Fragment, Service, BroadcastReceiver, ContentProvider 등 모든 주요 컴포넌트 간 데이터 교환에 사용됨

- 사용 예시
  + Activity <-> Activity
    * ```kotlin
      // 전송
      val intent = Intent(this, DetailActivity::class.java)
      val bundle = Bundle().apply {
          putString("name", "Bae")
          putInt("age", 30)
      }
      intent.putExtras(bundle)
      startActivity(intent)

      // 수신
      val name = intent.getStringExtra("name")
      val age = intent.getIntExtra("age", 0)
      ``` 
  + Fragment 초기 데이터 전달
    * ```kotlin
      val fragment = MyFragment().apply {
        arguments = Bundle().apply {
            putString("title", "Hello Fragment")
        }
      }

      // Fragment 내부
      val title = arguments?.getString("title")
      ``` 
  + 상태 저장/복원
    * ```kotlin
      override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("count", counter)
      }

      override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val restoredCount = savedInstanceState?.getInt("count") ?: 0
      }
      ``` 
  + Service, Broadcast, ContentProvider
    * Intent 내부 extras 로 전달 → 결국 Bundle이 사용됨 

- 특징
  + Key-Value 저장소
  + 다양한 타입 지원 (primitive, String, CharSequence, Parcelable, Serializable, ArrayList 등)
  + Parcelable 기반 직렬화 → 성능 좋음
  + 데이터 크기 제한 있음
  + 너무 큰 데이터 전달 시 TransactionTooLargeException 발생 (Binder 트랜잭션 한계, 보통 1MB 내외)
  + 대용량 데이터는 File, DB, SharedPreferences 등으로 처리 후 경로만 전달하는 방식 추천

- Bundle 사용 시 주의 사항
  + Null 안전성: get 계열 호출 시 항상 null 체크 or defaultValue 제공
  + Key 관리: 하드코딩 지양 → object Keys { const val USER_NAME = "user_name" }
  + 타입 일치: putInt() 했으면 반드시 getInt()로 꺼내야 함
  + Serializable vs Parcelable
    * Parcelable : 안드로이드 최적화된 직렬화, 빠름, 권장
    * Serializable : 구현 간단하지만 리플렉션 기반, 느림

- 면접 관련 질문
  + Bundle과 Intent의 차이는?
    * Intent는 컴포넌트 실행을 위한 "메시지 객체"
    * Bundle은 그 안에서 데이터를 담는 "데이터 컨테이너"
  + Serializable과 Parcelable 차이는?
    * Serializable: 자바 표준 직렬화, 느림, 리플렉션 기반
    * Parcelable: 안드로이드 전용 직렬화, 빠름, 직접 구현 필요
  + Bundle에 큰 데이터 넣으면 어떻게 되나요?
    * Binder 트랜잭션 한계(보통 1MB) 초과 시 TransactionTooLargeException 발생
    * 큰 데이터는 파일, DB, 캐시에 저장하고 식별자(id, uri, path)만 전달


---


### 객체 직렬화
- **Serializable**
  + 개념 및 정의
    * 자바 표준 직렬화 방식
    * JVM 레벨에서 사용 (파일 저장, 네트워크 전송)
    * 마커 인터페이스 (Serializable)만 구현하면 됨
    * 내부적으로 **리플렉션(Reflection)**을 사용 → 느림, 메모리 사용량 큼
  + 특징
    * serialVersionUID 필요 (클래스 변경 시 버전 불일치 방지)
    * transient 키워드 → 직렬화에서 제외할 필드
    * static 필드 → 직렬화 대상 아님
    * 직렬화 대상 객체의 모든 필드가 Serializable 가능해야 함  
  + 직렬화 사용 이유
    * 저장 : 객체 상태를 파일에 저장하거나 DB에 저장
    * 전송 : 네트워크를 통해 객체를 다른 JVM으로 전달
    * 캐싱 : 객체를 메모리/디스크에 저장해 재사용
    * RPC : 원격 호출에서 객체 데이터를 주고 받을때 사용
  + 사용방법
    * **Serializable** 인터페이스 구현
      ```java
          import java.io.Serializable;
       
          public class User implements Serializable {
              private static final long serialVersionUID = 1L;

              private String name;
              private int age;
           
              // 생성자, getter, setter 등
          }
      ```
    * Serializable은 마커 인터페이스로, 메서드가 하나도 없고 단지 직렬화 대상이라는 표시만 함
    * 직렬화 예제 (저장)
      ```java
          ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.dat"));
          User user = new User("철수", 25);
          out.writeObject(user);
          out.close();
      ```
    * 역질렬화 예제 (복원)
      ```java
          ObjectInputStream in = new ObjectInputStream(new FileInputStream("user.dat"));
          User user = (User) in.readObject();
          in.close();
      ```
    + 주의할 점
      * serialVersionUID : 클래스 버전 관리를 위한 고유 ID (버전이 불일치할 경우 오류 발생 가능)
      * transient 키워드 : 직렬화에서 제외할 필드에 사용 (transient String password;)
      * static 필드 : 클래스에 속한 값이므로 직렬화 되지 않음
      * 직렬화 대상 객체의 모든 필드 : 직렬화가 가능해야 함 (안될경우 NotSerializableException 발생)

- **Parcelable**
  + 개념 및 정의
    * 안드로이드 전용 직렬화 방식
    * Intent, Bundle 등에서 객체 전달에 최적화
    * 객체를 바이트 배열로 직접 변환 → 성능 우수
    * Android Studio + Kotlin → @Parcelize 덕분에 구현 간단해짐
  + 특징
    * 수동 구현 시 writeToParcel() & CREATOR 필요
    * Kotlin에서는 @Parcelize로 자동 처리
    * 성능 최적화 : GC 적게 발생, 메모리 사용 효율적
  + 예제
    * 데이터 클래스 만들기
    ```kotlin
        import android.os.Parcelable
        import kotlinx.parcelize.Parcelize
        
        @Parcelize
        data class User (val name: String, val age: Int) : Parcelable
    ```
    * Intent에 담아서 전달
    ```kotlin
        val user = User("철수", 25)
        val intent = Intent(this, DetailActivity::class.java)
        
        intent.putExtra("user", user)
        startActivity(intent)
    ```
    * 받은 쪽에서 꺼내기
    ```kotlin
        val user = intent.getParcelableExtra<User>("user")
    ```

- Parcelable vs Serializable
  + | 항목         | Parcelable 🚀      | Serializable 🐢 |
    | ---------- | ------------------ | --------------- |
    | **속도**     | 빠름 (메모리 직접 처리)     | 느림 (리플렉션 기반)    |
    | **용도**     | Android IPC 전용     | JVM 전반 (범용)     |
    | **구현 난이도** | 다소 복잡 (직접 구현 필요)   | 쉬움 (인터페이스만 추가)  |
    | **성능**     | 고성능, 모바일 최적화       | 저성능, 메모리 낭비     |
    | **권장 상황**  | 안드로이드 컴포넌트 간 객체 전달 | 파일 저장, 네트워크 전송  |
 
- 실무 팁
  + Intent/Bundle → Parcelable
    * 성능 차이가 크므로 Android에서는 Parcelable 권장
  + 네트워크 전송, 파일 저장 → Serializable/Gson/Protobuf
    * Parcelable은 안드로이드 전용이므로 범용 저장/전송은 다른 직렬화 방식 사용사용
  + 대용량 객체 전달 X
    * Bundle/Intent 크기 제한 (보통 1MB 내외) → TransactionTooLargeException 발생 가능
    * DB, SharedPreferences, File 저장 후 경로만 전달하는 방식이 안정적 

- 면접 관련 질문
  + Parcelable이 Serializable보다 빠른 이유는?
    * Serializable은 리플렉션 기반 (많은 오버헤드)
    * Parcelable은 직접 메모리에 write/read (GC 발생 적음)
  + Parcelable과 Serializable 언제 각각 쓰나요?
    * 안드로이드 내부 데이터 전달 (Intent, Bundle) → Parcelable
    * 범용 저장/전송 (파일, 네트워크, 캐싱) → Serializable 또는 JSON, ProtoBuf
  + Bundle에 큰 데이터 넣으면 어떻게 되나요?
    * TransactionTooLargeException 발생 가능
    * 해결책: DB/파일에 저장 후 key만 전달


---


### Application 클래스 역할
- 정의
  + 앱 프로세스가 시작될 때 생성되어, 앱 전역에서 사용 가능한 **싱글톤 컨텍스트(Context)**를 제공하는 클래스
  + onCreate()는 앱이 실행될 때 가장 먼저 호출
  + 앱이 완전히 종료되기 전까지 인스턴스는 살아있음
  + 전역 상태 관리, 초기화, 라이브러리 설정 등에 사용

- 주요 역할
  + | 역할                | 설명                                                             |
    | ----------------- | -------------------------------------------------------------- |
    | **전역 Context 제공** | 어떤 컴포넌트에서도 `applicationContext`로 접근 가능                         |
    | **앱 시작 시 초기화 작업** | DI 초기화, Logger 설정, SDK 초기화 등                                   |
    | **전역 상태/데이터 관리**  | 로그인 상태, 글로벌 설정, 싱글톤 객체 관리 등                                    |
    | **생명주기 감지**       | `registerActivityLifecycleCallbacks()`를 통해 Activity 생명주기 추적 가능 |
    | **공통 처리 추상화**     | 공통 에러 처리, 공통 폰트, 공통 테마 적용 등                                    |

- 사용 방법
  + MyApp 클래스 생성 및 Application 상속 
    * ```kotlin
      class MyApp : Application() {
         override fun onCreate() {
           super.onCreate()
           // 예: Timber 초기화, Hilt, Firebase 등
         }
      }
      ```
  + AndroidManifest.xml 선언
    * ```xml
      <application
         android:name=".MyApp" >
      </application>
      ```

- Application과 Context 관계
  + Application은 앱 프로세스 단위의 Context
  + Activity, Service, BroadcastReceiver 등은 모두 Application Context를 내부적으로 참조함
  + 따라서 applicationContext는 항상 안전하게 앱 전체에서 사용할 수 있음
  + ❌ 단, UI 관련 작업 (View.inflate, Dialog 등)은 Activity Context 필요

- 전역 상태 관리의 위험성
  + Application은 앱 실행 내내 살아있음
  + 전역 변수처럼 쓰면 편하지만, 잘못 관리하면 메모리 누수, 동시성 문제 유발
  + 특히 Activity, Fragment, View 참조를 저장하면 GC가 불가능해져 누수 발생
  + ✅ 안전한 활용법
    * Application에는 라이브러리 초기화, DI Container, 공용 util만 저장
    * UI 컴포넌트 참조는 절대 저장하지 않음

- 주의사항
  + | 주의점                   | 설명                                    |
    | --------------------- | ------------------------------------- |
    | **메모리 누수 주의**         | Activity, Context 등 참조 저장 금지 (GC 안 됨) |
    | **성능 영향**             | `onCreate()`에서 너무 무거운 작업 금지           |
    | **Thread-safe 설계 필요** | 전역 변수 접근 시 동시성 문제 고려                  |

- 면접 관련 질문
  + Application 생성과 소멸 시점?
    * 앱 프로세스 시작 시 onCreate()가 호출되고, 앱이 완전히 종료될 때 소멸
  + Application과 Activity의 Context 차이?
    * Application은 앱 전역의 Context, Activity는 UI에 특화된 Context
    * Activity UI와 관련된 작업(ViewInflate, Theme 등)은 Application Context 에서 하면 안 됨
  + 전역 데이터 Application에 저장?
    * 간단한 설정 or 일시적 정보는 괜찮음
    * DB나 SharedPreferences가 필요한 복잡한 데이터 별도 관리 권장
    * 장기 보관은 ViewModel, Repository 등을 사용


---


### AndroidManifest.xml
- 정의 
  + Android 앱의 핵심 구성 파일 입니다.
  + 앱에 대한 필수 정보를 Android 빌드 도구, Android 운영체제 및 Google Play에 제공합니다.
  + 모든 Android 엡 프로젝트의 루트 디렉터리에 반드시 존재해야 합니다.

- 주요 역할 및 내용
  + 애플리케이션 패키지 이름
    * 앱의 고유 식별자 입니다.
    * Java 또는 Kotlin의 패키지 이름 규칙을 따릅니다
    * Google Play Store에서 앱을 식별하는 데 사용되며, 한번 게시된 후에는 변경할 수 없습니다.
    * <manifest> 태그의 package 속성에 정의됩니다.
    
  + 애플리케이션 컴포넌트 선언
    * 앱을 구성하는 모든 핵심 컴포넌트(Activity, Service, BroadcastReceiver, ContentProvider)를 선언해야 합니다.
    * 각 컴포넌트는 <activity>, <service>, <receiver>, <provider> 태그를 사용하여 선언됩니다.
    * 컴포넌트의 이름, 속성 기능 등을 정의합니다.
    * ```xml
      <application>
            <activity android:name=".MainActivity" android:exported="true">
                <intent-filter>
                    <action android:name="android.intent.action.MAIN" />
                    <category android:name="android.intent.category.LAUNCHER" />
                </intent-filter>
            </activity>
            <service android:name=".MyBackgroundService" />
            <receiver android:name=".MyBroadcastReceiver" android:exported="false">
                <intent-filter>
                    <action android:name="android.intent.action.BOOT_COMPLETED"/>
                </intent-filter>
            </receiver>
            <provider
                android:name=".MyContentProvider"
                android:authorities="com.example.myapp.provider"
                android:exported="true" />
        </application>
      ```
      
  + 권한 선언
    * 앱이 시스템의 보호된 부분이나 다른 앱의 데이터에 접근하기 위해 필요한 권한을 선언합니다.
    * <uses-permission> 태그를 사용하여 요청합니다.
    * 인터넷 접근, 카메라 사용, 위치 정보 접근 등
    * ```xml
      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.CAMERA" />
      ```
      
  + 하드웨어 및 소프트웨어 기능 요구사항 선언
    * 앱이 정상적으로 작동하기 위해 필요한 하드웨어 또는 소프트웨어 기능을 명시합니다.
    * <users-feature>태그를 사용하여 선언 합니다.
    * 카메라 기능, 특정 센서 등
    * ```xml
      <uses-feature android:name="android.hardware.camera" android:required="true" />
      <uses-feature android:name="android.software.leanback" android:required="false" />
      ```
  
  + API 레벨 요구사항
    * 앱이 호환되는 최소 Android API 레벨, 타겟 API 레벨, 그리고 선택적으로 최대 API 레벨을 지정합니다.
    * <uses-sdk> 태그 내에 정의됩니다.
    * ```xml
      <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="33" />
      ```
      
  + 애플리케이션 메타데이터
    * 앱의 아이콘, 앱 이름, 테마 등 전역적인 애플리케이션 설정을 정의합니다.
    * <application> 태그의 속성으로 설정됩니다.
    * 추가적인 메타데이터를 <meta-data> 태그를 사용하여 키-값 쌍으로 제공할 수 있습니다.
    * 라이브러리나 특정 시스템 기능에 필요한 설정을 전달하는 데 사용됩니다.
    * ```xml
      <application
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
            <meta-data
                android:name="com.google.android.geo.API_KEY"
                android:value="YOUR_API_KEY"/>
        </application>
      ```

  + 인텐트 필터
    * 컴포넌트가 어떤 종류의 인텐트에 반응할 수 있는지 시스템에 알립니다.
    * <intent-filter> 태그 내에 <action>, <category>, <data> 요소를 사용하여 정의합니다.
    
- 면접 관련 질문
  + taskAffinity, launchMode 설정이 앱 동작에 어떤 영향을 주나요?
    * launchMode는 Activity의 인스턴스 생성 방식, 
    * taskAffinity는 액티비티가 어떤 태스크에 소속될지를 결정 합니다.
    
  + Android App Bundle 사용시 Manifest에서 주의할 점은 무엇인가요?
    * AAB는 기능 모듈별로 APK를 생성하는 방식이므로 Manifest도 각 모듈 별로 존재할 수도 있고, 병합 규칙에 따라 최종 Manifest가 결정됩니다.

  + allowBackup, fullBackupContent 설정의 보안적 의미는 무엇인가요?
    * android:allowBackup="true"가 설정되면 사용자가 adb로 앱 데이터를 백업/복원할 수 있고,
    * 민감한 데이터가 포함된 앱에서는 보안 위험이 될 수 있습니다.


---


### Context
- 개념 및 정의
  + Context = "현재 애플리케이션/컴포넌트가 어떤 환경에서 실행 중인지에 대한 핸들"
  + 즉, OS와 앱 사이의 다리 역할
    * 시스템 서비스 접근 (getSystemService)
    * 리소스 접근 (getResources)
    * 컴포넌트 실행 (startActivity, startService, sendBroadcast) 의 현재 상태와 환경에 대한 정보를 담고 있는 객체
  
- Context 종류
  + | Context 종류            | 설명                   | 대표 사용 예                                |
    | --------------------- | -------------------- | -------------------------------------- |
    | **Application**       | 앱 전역에서 하나            | DB, SharedPreferences, 전역 Toast        |
    | **Activity**          | UI 관련, 화면 단위         | 뷰 inflate, 다이얼로그, 테마 적용                |
    | **Service**           | 백그라운드 실행 환경          | 음악 재생, 위치 추적                           |
    | **BroadcastReceiver** | 브로드캐스트 수신 시 일시적으로 제공 | 알람 수신 후 동작 실행                          |
    | **ContextWrapper**    | Context 기능을 감싸서 확장   | `ContextThemeWrapper` (테마 적용된 Context) |
   
- Application vs Activity Context
  + Application Context
    * 전역적, 오래 살아남음
    * UI 관련 기능은 ❌ (예: Dialog 띄우기 불가 → Theme 없음)
  + Activity Context
    * 화면 단위로 UI 포함
    * 액티비티가 끝나면 함께 소멸해야 함
  + ❗실수 사례
    * 싱글톤 객체에 Activity Context를 저장하면 → Activity가 해제되지 않고 메모리 누수 발생

- context 사용
  + | 할 일            | 예시                                                         |
    |------------------|--------------------------------------------------------------|
    | 리소스 접근       | `getString(R.string.app_name)`                               |
    | 뷰 생성          | `LayoutInflater.from(context).inflate(...)`                  |
    | 파일 읽기/쓰기    | `context.openFileInput("file.txt")`                          |
    | 토스트 띄우기     | `Toast.makeText(context, "Hello", Toast.LENGTH_SHORT)`       |
    | 다른 액티비티 시작 | `context.startActivity(intent)`                              |
    | 서비스 시작       | `context.startService(...)`                                  |
 
- 면접 관련 질문
  + Application Context와 Activity Context 차이는?
    * Application: 앱 전역, UI 불가, 수명 길다
    * Activity: UI 포함, Activity 생명주기와 동일
  + 왜 Application Context로 Dialog를 띄우면 안 되나요?
    * Application Context에는 Theme/Window 정보 없음
    * UI 관련 기능은 반드시 Activity Context 사용해야 함
  + Context 메모리 누수는 언제 발생하나요?
    * Activity Context를 싱글톤/전역 static 변수에 저장했을 때
    * Activity 종료 후에도 GC가 수거하지 못해 메모리 누수 발생
  + getApplicationContext()와 this 차이는?
    * this: 현재 Activity Context
    * applicationContext: 앱 전역 Application Context  


---


### Weak/Strong Reference
- 개요
  + `WeakReference` 와 `StrongReference`는 GC(가비지 컬렉션) 과 객체 생명주기 관리에서 중요한 개념
  + Android에서는 메모리 누수 방지, 캐시 최적화, Context 참조 관리 등에서 많이 활용됨

- Strong Reference (강한 참조)
  + 일반적으로 사용하는 기본 참조 방식 default 
  + ```kotlin
    val activity = MainActivity()
    ```
  + 특징
    * 참조가 남아 있는 한, GC 대상이 아님
    * 명시적으로 null 처리하거나, 스코프에서 벗어나야 GC 가능
    * 객체 생명주기를 강하게 유지 → 일반적인 코드 대부분 Strong Reference 
  
  
- Weak Reference (약한 참조)
  + GC가 객체를 자유롭게 수거할 수 있는 참조
  + ```kotlin
    val activityRef = WeakReference(acitivty)
    ```
  + 특징
    * 참조는 유지하지만 GC가 필요시 수거 가능
    * 메모리 누수 방지, 캐시, Context 참조, Listener/Handler에서 활용 

- Java 4가지 참조 타입 비교
  + | 참조 타입       | GC 수거 여부                          | 특징/사용처                                 |
    | ----------- | --------------------------------- | -------------------------------------- |
    | **Strong**  | 참조 남아 있으면 절대 수거 안 함               | 일반적인 객체 참조 방식                          |
    | **Soft**    | 메모리 부족할 때만 GC가 수거                 | 캐시 구현 시 사용 (메모리가 충분하면 유지)              |
    | **Weak**    | 언제든 GC가 수거 가능                     | Handler → Activity 참조 방지, 메모리 민감 객체 참조 |
    | **Phantom** | 객체가 **finalize 후**, 수거 직전에만 참조 가능 | 리소스 해제 추적, 정리 작업 (거의 실무에서는 잘 안 씀)      |

- 비교
  + | 항목         | Strong Reference  | WeakReference                      |
    | ---------- | ----------------- | ---------------------------------- |
    | GC 대상 여부   | 참조가 존재하면 GC 대상 아님 | GC는 자유롭게 수거 가능                     |
    | 참조 유지      | 강하게 유지 (직접 소유)    | 약하게 유지 (GC 허용)                     |
    | 사용 예       | 대부분의 변수, 클래스 속성 등 | 캐시, Context 참조, 리스너, View 등        |
    | Android 활용 | 일반적인 코드           | Handler → Activity 참조 방지, LruCache |

- 실전 예제
  + 잘못된 사용 예제
    * ```kotlin
      class MyHandler(val activity: Activity) : Handler() {
          override fun handleMessage(msg: Message) {
              activity.doSomething()
          }
      }
      ```
    * Activity가 끝나도 Handler가 내부에서 강하게 참조 중이면 GC 불가 → 메모리 누수
    * ```kotlin
      class MyHandler(activity: Activity) : Handler() {
         private val activityRef = WeakReference(activity)

         override fun handleMessage(msg: Message) {
             val activity = activityRef.get()
             activity?.doSomething()
         }
      }
      ```
    * Activity가 끝나면 GC가 수거 가능하도록 WeakReference 로 해결

- 면접 관련 질문
  + WeakReference는 언제 사용하나요? 실무에서의 예시도 함께 설명해주세요.
    * WeakReference는 객체를 참조하면서도 GC가 해당 객체를 수거할 수 있도록 허용할 때 사용
    * Android에서는 Activity나 Context처럼 생명주기가 명확하고 무거운 객체를 오래 참조하면 메모리 누수 위험이 있음
    * Handler나 Listener 내부에서 Activity를 참조할 때 WeakReference를 사용
    * 오래 실행되는 타이머나 핸들러가 Activity를 직접 참조하면 화면이 닫힌 뒤에도 메모리가 회수되지 않기 때문에 WeakReference(activity)로 감싸서 GC가 수거할 수 있게 합니다."
  + WeakReference 객체에서 실제 객체를 사용할 때 주의해야 할 점은?
    * WeakReference.get()은 반환값이 nullable이기 때문에, 항상 null 체크 후 사용
    * 이 참조는 언제든 무효화될 수 있으므로, 장시간 신뢰 가능한 객체 상태가 필요하다면 별도의 강한 참조 유지가 필요할 수 있음
  + GC 캐시 정책 관련 참조 타입
    * Strong/Soft/Weak/Phantom 4가지의 참조타입이 존재
    * soft 는 캐시용으로 적합하며 메모리 부족할때만 수거 (LRU캐시처럼 동작)
    * phantom 객체가 finalize 된 이후 추적용으로 쓰임. 리소스 해제 후 처리 작업 추적 용도이나 복잡해서 거의 쓰이질 않음
  + Soft / Weak / Phantom 차이는?
    * Soft → 캐시용 (메모리 부족 시에만 정리)
    * Weak → 즉시 수거 가능 (안정적 누수 방지)
    * Phantom → 수거 이후 정리 추적 (거의 안 씀) 


---


### ANR(Application Not Responding)
- 정의
  + 안드로이드 앱에서 메인(UI) 스레드가 장시간(blocking) 멈춰 이벤트나 화면 갱신을 처리하지 못하면 발생
  + 시스템이 5초 이상 입력 이벤트 처리 못하면 → ANR 대화상자 표시

- ANR 발생 조건
  + | 유형                       | 제한 시간 | 설명                                                     |
    | ------------------------ | ----- | ------------------------------------------------------ |
    | 입력 이벤트 처리 지연             | 5초    | 터치, 키 입력 이벤트 처리 못할 때                                   |
    | Foreground Service 시작 지연 | 5초    | `startForegroundService()` 호출 후 `startForeground()` 지연 |
    | Service 실행 지연            | 수 초   | `onCreate()`, `onStartCommand()` 등 오래 걸릴 때             |
    | BroadcastReceiver 지연     | 10초 내 | `onReceive()`가 오래 걸릴 때                                 |
    | JobService 지연            | 수 초   | `onStartJob()`, `onStopJob()` 반환 지연                    |

- 주요 원인
  + 메인 스레드 블로킹
    * 네트워크 작업, 대용량 파일 I/O
    * 복잡한 연산 (긴 루프, 알고리즘)
    * 무거운 UI 갱신 (RecyclerView 과도한 bind 등)
  + 동기화 문제
    * 락 경쟁, 교착 상태(Deadlock)
  + 잘못된 컴포넌트 사용
    * BroadcastReceiver에서 긴 작업 수행
    * Service에서 무거운 로직 직접 처리
  + 시스템/기기 부하
    * 시스템 리소스 부족, 다른 프로세스 간섭  

- 예방 방법
  + 백그라운드 처리
    * 네트워크, 파일 I/O, DB → 별도 스레드 / Coroutine / WorkManager
  + StrictMode 활용
    * 개발 중 메인 스레드에서 잘못된 I/O 탐지
  + UI 최적화
    * RecyclerView, DiffUtil, 비동기 렌더링
  + 락/동기화 최소화
    * 락 범위 최소화, 순서 일관성 유지
  + 서비스/리시버 최적화
    * BroadcastReceiver에서는 빠른 처리만 → 나머지는 JobScheduler/WorkManager

- 면접 관련 질문
  + ANR이란 무엇인가요?
    * 메인 스레드가 입력 이벤트를 5초 이상 처리 못해 사용자에게 "응답 없음" 대화상자가 뜨는 현상
  + 주요 원인?
    * 메인 스레드에서 네트워크/IO 수행, 긴 루프, Deadlock, 무거운 UI 작업
  + 해결 방법?
    * 모든 무거운 작업은 백그라운드 스레드로
    * StrictMode, WorkManager, Coroutine, 최적화된 UI 사용
  + ANR 발생 시 분석 방법은?
    * /data/anr/traces.txt 확인
    * Perfetto/Systrace로 스레드 상태 및 부하 분석 


---


### 의존성 주입 (DI)
- 개념 및 정의
  + 객체 간의 의존성을 외부에서 주입하는 방식
  + 클래스 내부에서 직접 의존 객체를 생성하지 않고, 외부에서 전달(주입) 받아 사용
  + 주입 방식
    * 생성자 주입 → 가장 권장
    * 세터 주입 → 선택적 의존성일 때때
    * 인터페이스 주입 → 덜 쓰임

- 왜 필요한가?
  + | 이유      | 설명                                       |
    | ------- | ---------------------------------------- |
    | 유지보수 용이 | 구체 클래스가 아닌 추상(인터페이스)에 의존 → 교체 쉬움         |
    | 테스트 쉬움  | Mock 객체/Fake Repository 주입 가능            |
    | 확장성     | 기존 코드 수정 없이 새로운 구현체를 주입 가능               |
    | 결합도 낮춤  | 클래스 간 의존 관계 약화 → SRP, OCP 같은 SOLID 원칙 실현 |

- 예시
  + ❌ 잘못된 방식 (강한 결합)
    * ```kotlin
      class Car {
        private val engine = Engine() // 직접 생성 → Engine 교체 불가
      }
      ```
  + ✅ 좋은 방식 (생성자 주입)
    * ```kotlin
      class Car(private val engine: Engine) {
        fun drive() = engine.start()
      }

      // 외부에서 주입
      val car = Car(GasolineEngine())
      ```
  + ✅ 인터페이스 활용 (확장성↑)  
    * ```kotlin
      interface Engine {
        fun start(): String
      }

      class GasolineEngine : Engine {
        override fun start() = "Gasoline Engine Start"
      }

      class ElectricEngine : Engine {
        override fun start() = "Electric Engine Start"
      }

      class Car(private val engine: Engine) {
        fun drive() = engine.start()
      }

      // 주입 시점에서 엔진 교체 가능
      val car = Car(ElectricEngine()) // 쉽게 교체 가능
      ```

- 📌 안드로이드에서 DI가 중요한 이유
  + 안드로이드 객체 수명 주기(Activity, Fragment, ViewModel 등)가 복잡
  + Context, Repository, UseCase 등 의존성이 얽혀있음
  + 테스트 용이성: 예를 들어, UserRepository 대신 FakeUserRepository 주입 가능
  + 전역 싱글톤(Context, DB 등) 관리 편리 → ApplicationContext, Room DB, Retrofit 인스턴스

- 안드로이드에서 DI 도구
  + | DI 도구       | 특징                                |
    | ----------- | --------------------------------- |
    | **Hilt**    | 구글 공식 DI (Dagger 기반, 보일러플레이트 최소화) |
    | **Dagger2** | 성능 최상급, 복잡한 프로젝트에 강력, 단 설정 어려움    |
    | **Koin**    | 코틀린 DSL 기반, 설정 쉬움, 러닝커브 낮음        |
    | **수동 DI**   | 직접 객체 생성/팩토리, 소규모 프로젝트에 적합        |

- Hilt 예시
  + ```kotlin
    // Repository
    class UserRepository @Inject constructor() {
      fun getUser() = "철수"
    }

    // ViewModel
    @HiltViewModel
    class MainViewModel @Inject constructor(
      private val repository: UserRepository
    ) : ViewModel() {
      val user = repository.getUser()
    }

    // Application 설정
    @HiltAndroidApp
    class MyApp : Application()
    ```

- 면접 관련 질문
  + DI란 무엇이고, 왜 필요한가요?
    * 객체 간 결합도를 낮추고 유지보수/테스트를 쉽게 하기 위해 사용
  + 안드로이드에서 DI가 특히 중요한 이유는?
    * 복잡한 생명주기 관리, Context/Repository 의존성 관리, 테스트 용이성 확보
  + Hilt와 Koin의 차이는?
    * Hilt: 컴파일 타임 DI (안전하지만 설정 복잡)
    * Koin: 런타임 DI (코틀린 친화적, 간단하지만 성능 다소 떨어짐)
  + 싱글톤 패턴 대신 DI를 쓰는 이유는?
    * 싱글톤은 전역 상태 공유로 테스트/확장에 제약이 크지만, DI는 유연성과 확장성이 뛰어남


---


### WorkManager
- 정의
  + 안드로이드에서 백그라운드 작업을 보장하기 위한 Jetpack 라이브러리
  + 앱이 종료되거나 기기가 재부팅돼도 실행이 보장
  + 내부적으로 OS 버전에 맞게 `JobScheduler`, `AlarmManager`, `Firebase JobDispatcher` 등을 활용 → 앱 개발자는 API 하나만 사용하면 됨

- 사용 시점
  + | 상황            | 설명                         |
    | ------------- | -------------------------- |
    | **보장된 실행 필요** | 앱 종료, 재부팅 후에도 작업 실행 보장     |
    | **네트워크 필요**   | Wi-Fi / 데이터 연결 필요할 때       |
    | **주기적 동기화**   | 최소 15분 이상의 주기 작업           |
    | **조건 기반 실행**  | "충전 중일 때만", "네트워크 연결 시만" 등 |

- 특징
  + | 기능                                | 설명                     |
    | --------------------------------- | ---------------------- |
    | ✅ 작업 보장                           | 앱이 종료돼도, 재부팅돼도 실행됨     |
    | ✅ 제약 조건 지원                        | Wi-Fi, 충전 중일 때 등 설정 가능 |
    | ✅ JobScheduler, AlarmManager 등 통합 | OS 버전에 따라 자동 내부 분기     |
    | ✅ Chaining 지원                     | 작업 순서 정의 가능 (`then()`) |
    | ✅ Coroutine & RxJava 호환           | 코루틴 기반 작업 작성 가능        |

- Work 종류
  + OneTimeWorkRequest → 한 번 실행
  + PeriodicWorkRequest → 주기 실행 (최소 15분 이상, 정확한 시간 보장은 X → 최적화 고려)

- Worker 종류
  + | 클래스               | 특징                          |
    | ----------------- | --------------------------- |
    | `Worker`          | 기본 동기 실행                    |
    | `CoroutineWorker` | 코루틴 기반 (suspend fun doWork) |
    | `RxWorker`        | RxJava 기반                   |
  + ```kotlin
    class MyCoroutineWorker(appContext: Context, params: WorkerParameters) 
      : CoroutineWorker(appContext, params) {
        override suspend fun doWork(): Result {
            delay(1000) // 비동기 안전
            return Result.success()
      }
    }

    ```  

- Work 상태 관리
  + WorkInfo 객체로 상태 추적 가능
  + 상태: ENQUEUED, RUNNING, SUCCEEDED, FAILED, CANCELLED 
  + ```kotlin
    WorkManager.getInstance(context)
      .getWorkInfoByIdLiveData(workRequest.id)
      .observe(this) { workInfo ->
        if (workInfo.state == WorkInfo.State.SUCCEEDED) {
            Log.d("Work", "작업 완료")
        }
    }
    ```

- 데이터 전달
  + InputData → Worker에 값 전달
  + OutputData → Worker에서 결과 반환
  + ```kotlin
    // 전달
    val data = workDataOf("key" to "value")
    val work = OneTimeWorkRequestBuilder<MyWorker>()
      .setInputData(data)
      .build()

    // Worker 내부
    override fun doWork(): Result {
      val value = inputData.getString("key")
      val output = workDataOf("result" to "done")
      return Result.success(output)
    }
    ```

- WorkManager vs JobScheduler vs AlarmManager
  + | 항목       | WorkManager       | JobScheduler    | AlarmManager |
    | -------- | ----------------- | --------------- | ------------ |
    | 실행 보장    | ✅ 앱 종료/재부팅 후에도 실행 | ✅ (Lollipop 이상) | ❌ 보장 안 됨     |
    | 제약조건 지원  | ✅ 충전, 네트워크 등 가능   | ✅               | ❌ (시간만 가능)   |
    | 최소 주기    | 15분               | 15분             | 초 단위 가능      |
    | 사용 난이도   | 쉬움 (Jetpack 통합)   | 복잡              | 단순           |
    | 대표 사용 사례 | 데이터 동기화, 업로드      | 시스템 작업, 배치      | 알람, 푸시 시뮬레이션 |
  + 👉 즉, WorkManager는 JobScheduler + AlarmManager의 통합 고수준 API
 

- 면접 관련 질문
  + WorkManager를 언제 사용하나요?
    * 앱이 종료/재부팅돼도 반드시 실행되어야 하는 작업
    * 주기적 동기화, 파일 업로드/백업
    * 제약 조건 기반 작업
  + AlarmManager 대신 WorkManager를 써야 하는 이유는?
    * WorkManager는 작업 실행을 보장하지만 AlarmManager는 기기에 따라 누락될 수 있음
    * 다만 정확한 시간 알림은 AlarmManager가 필요
  + 주기 작업 최소 주기가 왜 15분인가요?
    * 배터리 절약을 위해 안드로이드 시스템 정책상 JobScheduler 기반 최소 간격이 15분
  + Worker / CoroutineWorker 차이는 무엇인가요?
    * Worker는 동기, CoroutineWorker는 suspend 기반 비동기 → 안드로이드에서는 CoroutineWorker 권장


---


### DiffUtil
- 개념 및 정의
  + RecyclerView에서 데이터가 변경될 때 변경된 부분만 효율적으로 업데이트하기 위한 유틸리티 클래스.
  + 전체 리사이클러뷰를 다시 그리는 notifyDataSetChanged() 대신, Diff 알고리즘을 통해 바뀐 아이템만 갱신 → 성능 최적화 + UX 개선

- 기본 사용법
  + ```kotlin
    class MyDiffCallback(
      private val oldList: List<User>,
      private val newList: List<User>
    ) : DiffUtil.Callback() {
    
      override fun getOldListSize() = oldList.size
    
      override fun getNewListSize() = newList.size
    
      // 아이템 자체가 같은지 (고유 ID 비교)
      override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition].id == newList[newItemPosition].id
      }

      // 아이템의 내용이 같은지 (equals 비교)
      override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition] == newList[newItemPosition]
      }
    }

    // 사용
    val diffResult = DiffUtil.calculateDiff(MyDiffCallback(oldList, newList))
    adapter.submitList(newList)  // ListAdapter면 이거만 호출하면 됨
    diffResult.dispatchUpdatesTo(adapter)
    ``` 

- 📌 ListAdapter와 DiffUtil
  + 안드로이드 Jetpack에서 제공하는 ListAdapter는 내부적으로 DiffUtil을 내장하고 있음
  + 따라서 DiffUtil.ItemCallback<T>만 구현하면 자동으로 DiffUtil 적용 가능 
  + ```kotlin
    data class User(val id: Int, val name: String)

    class UserAdapter : ListAdapter<User, UserAdapter.ViewHolder>(DIFF_CALLBACK) {
        
        companion object {
            private val DIFF_CALLBACK = object : DiffUtil.ItemCallback<User>() {
                override fun areItemsTheSame(oldItem: User, newItem: User) = oldItem.id == newItem.id
                override fun areContentsTheSame(oldItem: User, newItem: User) = oldItem == newItem
            }
        }

        class ViewHolder(val binding: ItemUserBinding) : RecyclerView.ViewHolder(binding.root)

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val binding = ItemUserBinding.inflate(LayoutInflater.from(parent.context), parent, false)
            return ViewHolder(binding)
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val user = getItem(position)
            holder.binding.textView.text = user.name
        }
    }
    ```

- DiffUtil vs notifyDataSetChanged
  + | 비교 항목      | notifyDataSetChanged() | DiffUtil           |
    | ---------- | ---------------------- | ------------------ |
    | 갱신 범위      | 전체 아이템 다시 그림           | 변경된 항목만 그림         |
    | 성능         | ❌ 비효율적                 | ✅ 효율적              |
    | UX (애니메이션) | ❌ 깜빡임, 스크롤 튐           | ✅ 자연스러운 애니메이션      |
    | 사용 편의성     | 간단하지만 성능 나쁨            | 초기 구현 필요하지만 재사용 가능 |
 

- 면접 예상 질문
  + DiffUtil을 직접 구현했을 때와 ListAdapter의 차이는?
    * DiffUtil은 직접 old/new 리스트를 비교하고 dispatchUpdatesTo() 호출 필요
    * ListAdapter는 DiffUtil을 내부적으로 관리하므로 submitList()만 호출하면 됨
  + DiffUtil의 내부 동작 원리 아나요?
    * 최소 편집 거리 알고리즘 (Myers’s diff algorithm) 기반으로 리스트 변경점 계산
    * O(n + m) 성능 (n = oldList, m = newList 크기)
  + DiffUtil에서 areItemsTheSame과 areContentsTheSame 차이는?
    * areItemsTheSame: 같은 객체(동일 ID)인지 확인 (ex: PK 비교)
    * areContentsTheSame: 같은 객체라도 내용이 바뀌었는지 확인 (equals 비교)


---


### 안드로이드 8.0 이후 브로드캐스트 제한사항
- 개요
  + Android 8.0부터는 배터리 수명과 시스템 성능을 향상시키기 위해 백그라운드 실행에 대한 제한이 강화되었습니다.
  + Android 8.0 이전에는 앱이 매니페스트에 브로드캐스트 리시버를 등록해두면, 앱이 실행 중이지 않더라도 시스템이나 다른 앱에서 발생하는 다양한 브로드캐스트를 수신할 수 있었습니다.
    이는 편리했지만, 많은 앱이 특정 시스템 이벤트에 응답하기 위해 백그라운드에서 깨어나 리소스를 소모하는 원인이 되었습니다.
    이로 인해 배터리가 빠르게 소모되고 시스템 성능이 저하될 수 있었습니다. 안드로이드 8.0에서는 이러한 문제를 해결하기 위해 매니페스트에 등록된 리시버가 수신할 수 있는 브로드캐스트의 종류를 제한했습니다.

- 핵심 변경 사항
  + 대부분 암시적 브로드캐스트 수신 불가 : 앱이 Android 8.0 이상을 타겟팅하는 경우, 매니페스트에 등록된 리시버는 더 이상 대부분의 암시적 브로드캐스트를 수신할 수 없습니다.
  + 명시적 브로드캐스트는 계속 수신 가능 : 매니페스트에 등록된 리시버는 여전히 명시적 브로드캐스트를 수신할 수 있습니다.
  + 컨텍스트에 등록된 리시버는 계속 동작 : Context.registerReceiver()를 사용하여 동적으로 등록된 리시버는 앱이 활성 상태일 때 암시적 브로드캐스트를 포함한 모든 브로드캐스트를 계속 수신할 수 있습니다.

- 자세한 설명
  + 제한의 대상
    * 브로드캐스트 Android 8.0에서 주로 제한되는 것은 암시적 브로드캐스트입니다.
  + 예시
    * android.net.conn.CONNECTIVITY_CHANGE (CONNECTIVITY_ACTION): 네트워크 연결 상태가 변경될 때 발생하는 브로드캐스트입니다.
      Android 7.0 (API 레벨 24)부터 이미 매니페스트 등록이 제한되었지만, Android 8.0에서는 이 제한이 더 광범위하게 적용됩니다.
    * android.hardware.action.NEW_PICTURE (ACTION_NEW_PICTURE): 새로운 사진이 찍혔을 때 발생하는 브로드캐스트입니다. 이 역시 Android 7.0부터 제한되었습니다.

- 예외적으로 허용되는 암시적 브로드캐스트
  + 모든 암시적 브로드캐스트가 제한되는 것은 아닙니다. 시스템 운영에 중요하거나, 앱이 특정 기능을 수행하기 위해 반드시 필요한 일부 브로드캐스트는 여전히 매니페스트에 등록된 리시버를 통해 수신할 수 있습니다.
  + 예외 브로드캐스트의 예
    * ACTION_BOOT_COMPLETED: 기기가 부팅을 완료했을 때 발생합니다.
    * ACTION_LOCKED_BOOT_COMPLETED: 직접 부팅(Direct Boot) 모드에서 사용자가 기기를 잠금 해제하기 전에 발생합니다.
    * SMS/MMS 관련 브로드캐스트 (예: SMS_RECEIVED_ACTION, WAP_PUSH_RECEIVED_ACTION)

- 컨텍스트 등록 리시버의 역할 강화
  + 매니페스트 등록 리시버의 기능이 제한됨에 따라, 컨텍스트 등록 리시버의 중요성이 커졌습니다.
  + Context.registerReceiver(): Activity나 Service와 같은 컴포넌트의 컨텍스트를 사용하여 코드 내에서 동적으로 브로드캐스트 리시버를 등록합니다.
  + 생명주기: 컨텍스트 등록 리시버는 해당 컨텍스트가 유효한 동안에만 브로드캐스트를 수신합니다.
    예를 들어, Activity 컨텍스트에 등록된 리시버는 Activity가 활성 상태일 때만 동작합니다.
  + 장점: 앱이 사용자에게 보여지거나 활발하게 사용될 때만 브로드캐스트를 수신하므로, 불필요한 백그라운드 작업을 줄일 수 있습니다.

- 앱 타겟 API 레벨의 중요성
  + 이러한 브로드캐스트 제한은 기본적으로 앱이 Android 8.0 (API 레벨 26) 이상을 타겟팅할 때 적용됩니다.

- 결론
  + Android 8.0의 브로드캐스트 제한은 앱의 백그라운드 동작을 최적화하여 사용자 경험을 향상시키기 위한 중요한 변경 사항입니다.
  + 개발자는 이러한 제한 사항을 이해하고, 앱이 최신 Android 버전에서도 안정적으로 동작하도록 JobScheduler, WorkManager, 컨텍스트 등록 리시버, 포그라운드 서비스 등의 대안을 적극적으로 활용해야 합니다.
  + 이는 배터리 효율성을 높이고 시스템 리소스 사용을 최적화하는 데 기여합니다.

- 면접 예상 질문
  + Android 8.0이상에서 브로드캐스트 관련 제한사항은 무엇인가요?
    * 암시적 브로드캐스트의 정적 등록제한
    * 백그라운드 앱은 브로드캐스트 수신 제한
    * 명시적 브로드캐스트나 런타임 등록 권장
    * 배터리 최적화 및 성능 향상을 위한 정책 변화
  + 암시적 브로드캐스트와 명시적 브로드캐스트의 차이점은?
    * 암시적 : 특정 수신자를 지정하지 않고 Intent 필터에 따라 시스템이 전달
    * 명시적 : setComponent() 또는 setPackage()로 특정 리시버 지정
    * Android 8.0부터는 명시적 브로드캐스트 권장


---


### SharedPreferences & DataStore
- SharedPreferences
  + 작은 양의 key-value 형태 데이터를 로컬에 저장할 수 있도록 제공되는 기본 저장 API
  + 주로 로그인 정보, 앱 설정, 토글 상태 등 간단한 데이터를 저장하는데 사용
  + 핵심특징
    * XML 파일 기반 저장
    * 동기적 or제한적인 비동기 방식 (apply())
    * 앱 재시작 후에도 데이터 유지됨 (영속성)
    * Thread-safe 보장 없기에 다중 접근 시 위험
    * 복잡한 데이터 구조 저장에 부적합
  + Sample Code
    * ```kotlin
      // 데이터 쓰기
      val prefs = context.getSharedPreferences("my_prefs", Context.MODE_PRIVATE)
      prefs.edit {
        putString("userName", "james")
        .putBoolean("isLogin", true)
      }
      
      // 데이터 읽기
      val username = prefs.getString("userName", null)
      val isLoggedIn = prefs.getBoolean("isLogin", false)
      ```
  + apply() vs commit()
    * `apply()` → 비동기, 결과 반환 없음 (권장)
    * `commit()` → 동기, boolean 반환 (성공 여부 확인 가능)
    * UI 스레드에서 commit() 호출 시 ANR 위험
  + Migration 이슈
    * 앱 데이터가 많거나 빈번히 쓰기 작업이 있을 경우 XML 파싱 지연 발생 가능
    * SharedPreferences는 내부적으로 하나의 파일(XML)에 모든 데이터를 저장하므로, 데이터 크기가 커질수록 성능 저하  

- DataStore
  + Jetpack에서 제공하는 최신 데이터 저장 솔루션으로, 비동기 처리와 타입 안전성을 갖춘 Key-Value 저장 방식
  + SharedPreferences의 단점을 보완하고, Flow 및 Coroutine을 기반으로 설계
  + 핵심 특징
    * 코루틴 기반 비동기 저장
    * Flow 통한 실시간 데이터를 감지 및 수집
    * 두 종류 제공
      1. `Preferences DataStore`: Key-Value 저장
      2. `Proto DataStroe`: 구조화된 커스텀 객체 저장 (protobuf 기반)
    * 스레드 안전 + ANR 방지
    * JetPack 공식 권장 방식
  + Sample Code
    * 의존성 추가 `implementation("androidx.datastore:datastore-preferences:1.0.0")`
    * ```kotlin
      class UserPreferencesRepository(private val context: Context) {
        // 초기 설정
        private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "user_settings")

        // 키 정의
        object UserPreferencesKeys {
          val USERNAME = stringPreferencesKey("userName")
          val IS_LOGIN = booleanPreferencesKey("isLogin")
        }

        val userData: Flow<Pair<String?, Boolean>> = context.dataStore.data
          .map { prefs ->
            val username = prefs[UserPreferencesKeys.USERNAME]
            val isLoggedIn = prefs[UserPreferencesKeys.IS_LOGIN] ?: false
            username to isLoggedIn
        }

        suspend fun saveUser(username: String, isLoggedIn: Boolean) {
          context.dataStore.edit { prefs ->
            prefs[UserPreferencesKeys.USERNAME] = username
            prefs[UserPreferencesKeys.IS_LOGIN] = isLoggedIn
          }
        }

        suspend fun clearUser() {
          context.dataStore.edit { prefs ->
            prefs.clear()
          }
        }
      }
      ```
  + Preferences vs Proto
    * Preferences DataStore → 단순 Key-Value (SharedPreferences 대체)
    * Proto DataStore → 구조화된 데이터 저장 (스키마 기반, 타입 안전 보장
  + Migration 지원
    * 기존 SharedPreferences 데이터를 DataStore로 마이그레이션 가능
    * SharedPreferencesMigration(context, "prefs_name")
  + 실시간 반응
    * Flow 기반 → 데이터 변경 시 UI 자동 업데이트 가능 (collectLatest) 

- 비교
  + | 항목         | SharedPreferences  | DataStore                            |
    | ---------- | ------------------ | ------------------------------------ |
    | 저장 구조      | XML 파일 (Key-Value) | Preferences (Key-Value) / Proto (객체) |
    | API 방식     | get/put 직접 호출      | Flow + suspend 함수                    |
    | 저장/읽기      | 동기 + 제한적 비동기       | 완전 비동기 (Coroutine 기반)                |
    | 스레드 안전성    | ❌ 보장 안 됨           | ✅ 보장됨                                |
    | 데이터 타입 안전성 | ❌ 없음               | ✅ (특히 ProtoDataStore)                |
    | 성능/확장성     | 데이터 많아지면 느려짐       | 대량 데이터에도 효율적                         |
    | 권장 여부      | 레거시 유지용            | ✅ Jetpack 공식 권장                      |


- 면접 관련 질문
  + SharedPreferences에서 ANR이 발생할 수 있는 상황은?
    * 메인 스레드에서 commit() 호출 시 발생
    * 큰 데이터를 자주 저장할 때 XML 파싱이 오래 걸리면 위험
  + DataStore에서 Flow를 쓰는 이유는?
    * 데이터 변경 시 자동으로 UI 업데이트 반영 가능
    * Reactive 패턴에 적합 → MVVM, LiveData/StateFlow와 궁합 좋음
  + DataStore Proto를 써야 하는 상황은?
    * 데이터 구조가 복잡하고 타입 안정성이 중요한 경우
    * 예: 사용자 설정(UserSettings), 앱 환경설정, 다중 필드 상태 저장


---


### okHttp와 Retrofit 차이
- 정의
  + Android 및 Java 애플리케이션에서 네트워킹을 처리하는 데 널리 사용되는 라이브러리이지만, 서로 다른 계층에서 작동하며 다른 목적을 가지고 있습니다.

- OkHttp : HTTP 클라이언트
  + 역할
    * OkHttp는 HTTP 및 HTTP/2 클라이언트 입니다.
    * 실제 네트워크 요청을 만들고 응답을 받는 저수준(low-level) 라이브러리입니다.
  + 주요 기능
    * HTTP GET, POST, PUT, DELETE 등 다양한 요청 메서드 지원
    * 요청/응답 헤더 및 본문 처리
    * 연결 풀링을 통한 성능 최적화
    * Gzip 압축을 통한 데이터 크기 감소
    * 응답 캐싱
    * 타임아웃 설정
    * TSL/SSL 보안 연결 지원
    * HTTP/2 및 SPDY 프로토콜 지원
    * 자동 재시도 및 리디렉션 처리
    * WebSocket 지원
  + 사용 방식 : 개발자가 직접 Request 객체를 만들고, OkHttpClient를 사용하여 이를 실행하고, Response 객체를 받아 처리해야 합니다.
    ```kotlin
    // OkHttp 사용 예시
    val client = OkHttpClient()

    val request = Request.Builder()
        .url("https://api.example.com/data")
        .header("Authorization", "Bearer your_token")
        .build()

    client.newCall(request).enqueue(object : Callback {
        override fun onFailure(call: Call, e: IOException) {
            // 오류 처리
        }

        override fun onResponse(call: Call, response: Response) {
            if (response.isSuccessful) {
                val responseBody = response.body?.string()
                // 응답 처리 (예: JSON 파싱)
            } else {
                // 서버 오류 처리
            }
        }
    })
    ```
  + 장점
    * 매우 효율적이고 강력한 HTTP 통신 기능 제공
    * 세밀한 요청/응답 제어 가능
    * 다양한 네트워크 최적화 기능 내장
  + 단점
    * 상대적으로 저수준 API이므로, 반복적인 코드 작성이 필요할 수 있음
    * API 엔드포인트가 많아질수록 코드 관리가 복잡해질 수 있음

- Retrofit : 타입-세이프 HTTP 클라이언트
  + 역할
    * Retrofit은 타입-세이프한 REST 클라이언트입니다. 즉, HTTP API를 Java/Kotiln 인터페이스로 변환하여 사용하기 쉽게 만들어주는 고수준 라이브러리입니다.
  + 기반
    * Retrofit은 내부적인 OkHttp를 HTTP 클라이언트로 사용합니다. 즉 Retrofit은 OkHttp 위에 구축된 추상화 계층입니다.
  + 주요 기능
    * 인터페이스 기반 API 정의 : 어노테이션을 사용하여 HTTP 요청을 인터페이스 메서드로 매핑합니다.
    * 자동 직렬화/역직렬화 : 요청 본문과 응답 본문을 Java/Kotlin 객체로 자동으로 변환해줍니다.
    * 동기/비동기 요청 지원 : Kotlin 코루틴, RxJava, Call 객체를 통한 비동기 처리 지원
    * URL 파라미터, 쿼리 파라미터, 요청 본문 등을 어노테이션으로 쉽게처리
    * 멀티파트 요청지원
  + 사용 방식 : 개발자는 API 엔드포인트를 정의하는 인터페이스를 만들고, Retrofit 빌더를 사용하여 해당 인터페이스의 구현체를 생성합니다.
    ```kotlin
    // Retrofit 인터페이스 정의 예시
    interface ApiService {
        @GET("data")
        suspend fun getData(@Header("Authorization") token: String): MyData // MyData는 데이터 모델 클래스
    }

    // Retrofit 사용 예시
    val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .client(OkHttpClient()) // 선택 사항: 커스텀 OkHttpClient 설정 가능
        .addConverterFactory(GsonConverterFactory.create()) // JSON <-> 객체 변환기
        .build()

    val apiService = retrofit.create(ApiService::class.java)

    // 코루틴 내에서 호출
    viewModelScope.launch {
        try {
            val myData = apiService.getData("Bearer your_token")
            // 성공 처리
        } catch (e: Exception) {
            // 오류 처리
        }
    }
    ```
  + 장점
    * API 호출 코드를 매우 간결하고 직관적으로 작성 가능
    * 타입-세이프 : 컴파일 시점에 API 정의 오류를 발견할 수 있음
    * JSON/XML 등의 데이터 변환을 자동으로 처리하여 보일러플레이트 코드 감소
    * OkHttp의 강력한 네트워킹 기능을 그대로 활용 가능
  + 단점
    * OkHttp에 대한 의존성을 가짐
    * 매우 세밀한 저수준 제어가 필요한 경우 OkHttp를 직접 사용하는 것보다 유연성이 떨어질 수 있음
- OkHttp와 Retrofit의 관계 및 차이점 요약
  + | 특징       | OkHttp                    | Retrofit              |
    |----------|---------------------------|-----------------------|
    | 계층       | 저수준 HTTP 클라이언트            | 고수준 REST 클라이언트        | 
    | 주요목적     | HTTP 요청/응답처리, 네트워크 최적화    | API엔드포인트를 인터페이스로 추상화  |
    | 사용방식     | Request.Response 객체 직접 처리 | 어노테이션 기반 인터페이스 정의, 자동 객체 매핑 |
    | 코드 복잡도   | 상대적으로 높음                  | 상대적으로 낮음 |
    | 데이터 변환   | 수동처리                      | 자동처리 |
    | 의존성      | 독립성                       | OkHttp에 의존 |

- 면접 관련 질문
  + Retrofit은 OkHttp와 어떤 관계를 가지고 있나요?
    * Retrofit은 OkHttp 위에 구축된 추상화 계층입니다.
      Retrofit은 내부적으로 OkHttp를 HTTP클라이언트로 사용합니다. Retrofit은 OkHttp의 강력한 네트워킹 기능을 활용하면서 개발자가 더 쉽고 간결하게 API를 호출할 수 있도록 도와줍니다.
  + OkHttp를 "저수준(low-level) HTTP 클라이언트"라고 하고 Retrofit을 "고수준(high-level) REST 클라이언트"라고 부르는 이유는?
    * 저수준 HTTP 클라이언트는 개발자가 직접 Request 객체를 만들고, OkHttpClient를 사용하여 이를 실행하고, Response 객체를 받아 헤더나 본문을 직접 파싱하는 등 HTTP 통신의 세부적인 부분을 다루기 때문입니다.
    * 고수준 HTTP 클라이언트는 HTTP API의 엔드포인트, 파라미터, 요청/응답 타입 등을 인터페이스와 어노테이션으로 추상화합니다. 개발자는 실제 HTTP 요청 생성이나 응답 파싱의 복잡한 과정을 직접 다루지 않고, 마치 일반적인 메소드를 호출하듯이 API를 사용할 수 있게 해주기 때문입니다.
  + Retrofit의 가장 큰 장점 중 하나인 "타입-세이프(type-safe)"하다는 것은 어떤 의미인가요?
    * Retrofit은 API 인터페이스를 정의할 때 요청 파라미터의 타입과 응답 객체의 타입을 명시합니다. 
      이를 통해 컴파일 시점에 파라미터 타입 불일치나 잘못된 응답 타입 사용 등의 오류를 미리 발견할 수 있다는 의미입니다.
    * 이로 인해 잘못된 타입으로 인한 런타임 예외 발생 가능성을 줄여주고, API 명세가 코드로 명확하게 표현되므로, API 사용 방법을 이해하기 쉽고 실수를 줄일 수 있습니다.


---


### Handler, Looper, MessageQueue
- 개념 및 정ㅢ
  + Handler: 메시지 전송자 → Runnable/Message를 Queue에 넣어 Looper에게 전달
  + MessageQueue: 메시지 저장소 (FIFO) → `enqueueMessage()`, `next()`로 꺼냄
  + Looper: 메시지 처리기 → Queue에서 메시지를 꺼내 `Handler.handleMessage()` 실행
  + 즉, Handler.post() → MessageQueue 저장 → Looper.loop() → Handler 실행 

- 비교표
  + | 구성요소         | 역할      | 설명                                   |
    |--------------|---------|--------------------------------------|
    | Handler      | 메시지 전송자 | 작업에서 메시지를 생성에서 Looper에게 전달           |
    | MessageQueue | 메시지 저장소 | 전송된 메시지를 시간순으로 저장 (FIFO 큐)           |
    | Looper       | 메시지 처리기 | MessageQueue를 무한히 돌면서 메시지를 하나씩 꺼내 실행 |

- 전체 구조 흐름
  + ```markdown
    - Thread
        └── Looper
              └── MessageQueue
                       ↑
                   Handler.post()
    ```
  + `Handler`가 `Message` 또는 `Runnable`을 생성하여 `MessageQueue`에 넣음
  + `Looper`는 `MessageQueue`를 계속 돌면서 메시지를 꺼냄
  + 해당 메시지를 처리 (`Handler.handleMessage()` 또는 `Runnable.run()` 호출)

- Handler
  + **Runnable** 또는 **Message**를 전달할 수 있는 인터페이스
  + `post(Runnable)` 또는 `sendMessage(Messgae)` 사용
  + `handlerMessage(Message msg)`를 override 하여 메시지 처리 가능

- Looper
  + `Thread`에 붙어서 해당 쓰레드가 이벤트 루프를 돌도록 만듬
  + 기본적으로 UI 메인 쓰레드에는 `Looper.getMainLooper()`가 존재
  + 새 쓰레드에서 쓰고 싶다면 `Looper.prepare()` -> `Looper.loop()` 호출 필요

- MessageQueue
  + 메시지들이 `enqueueMessage()`로 들어와 저장됨
  + `next()` 메서드로 메시지를 하나씩 꺼냄
  + `Looper.loop()`이 이 `next()`를 무한 반복

- 코드 예시
  + 메인 쓰레드에서 Handler 사용
    ```kotlin
        val handler = Handler(Looper.getMainLooper())
    
        handler.post {
            // UI 쓰레드에서 실행
            textView.text = "작업 완료"
        }
    ```
    * `post()`는 내부적으로 Message를 만들어 MessageQueue에 등록
    * Main Looper는 이걸 꺼내서 실행
  + 새 Thread에서 Handler 만들기
    ```kotlin
        class MyThread : Thread() {
            lateinit var handler : Handler
    
            override fun run() {
                Looper.prepare()    // Looper를 이 Thread에 바인딩
                handler = Handler(Looper.myLooper()!!) {
                    println("새 쓰레드에서 메시지 처리 됨")
                    true
                }
                Looper.loop()   // 메시지 루프 시작 
            }
        }
    
        val t = MyThread()
        t.start()
    
        // 나중에 다른 쓰레드에서 호출
        t.handler.post {
            println(" 이 코드는 MyThread 안에서 실행됨")
        }
    ```
  + 딜레이, 반복 작업 가능
    ```kotlin
        val handler = Handler(Looper.getMainLooper())
        val runnable = object : Runnable {
            override fun run() {
                println("1초마다 반복")
                handler.postDelayed(this, 1000)
            }
        }
        handler.post(runnable)
    ```
    - `postDelayed()`로 딜레이 실행 가능
    - `removeCallbacks(runnable)`로 중단 가능

- Handler 직접 써야하는 이유
  + | 상황                   | 이유                                    |
    |----------------------|---------------------------------------|
    | 다른 쓰레드에서 UI를 바꿔야 할 떄 | UI는 오직 메인 쓰레드에서만 수정 가능 -> Handler로 전달 |
    | 쓰레드 간 작업 분리          | 백그라운드 쓰레드에서 계산하고 -> 메인 쓰레드에서 결과 반영    |
    | 일정 시간 후 작업 실행        | `postDelayed`로 지연 실행 가능               |
    | 주기적 작업 처리            | `Runnable` 재전송 방식으로 반복 구현 가능          |

- 면접 관련 질문
  + Handler란 무엇인가요?
    * `Message` 또는 `Runnable`을 다른 스레드로 전달하고, Looper를 통해 큐에 등록된 작업을 실행하게 해주는 클래스.
    * UI 쓰레드로 작업을 전달하거나 딜레이/반복 작업에도 사용
  + Looper란 무엇인가요? 왜 필요한가요?
    * 해당 쓰레드에 연결되어 있는 이벤트 루프 처리기
    * `MessageQueue`를 반복해서 확인하며 메시지를 하나씩 꺼내서 실행
    * `UI 쓰레드`에는 기본적으로 `Looper`가 존재
  + Handler는 어떤 쓰레드에서 동작하나요?
    * 생성 시 지정한 Looper에 따라 동작하므로 메인 쓰레드에서 실행
    * 직접 만든 쓰레드에 사용하려면 Looper.prepare()로 초기화 해야 함
  + Handler 대신 Coroutines를 써도 되지 않나요?
    * Coroutine은 Handler보다 추상화된 방식으로 스레드 전환을 처리 가능
    * `withContext(Dispatchers.Main)`으로 메인쓰레드 전환이 가능하며, **코드 가독성과 에러 처리 측면에서 유리**
    * 하지만 내부적으로 Dispatchers는 `Handler`를 기반으로 동작하는 경우가 많다


---


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


### Interceptor
- 정의
  + retrofit2는 안드로이드에서 가장 많이 쓰이는 HTTP 클라이언트 라이브러리이며, 내부적으로 OkHttp를 사용하는데 OkHttp의 핵심 기능 중 하나가 바로 **Interceptor**이고 기능을 그대로 사용
  + Interceptor는 모든 네트워크 요청/응답을 **가로채서 가공하거나 검사할 수 있는 훅(Hook)** 이자 인터페이스**

- 특징
  + | 특징                        | 설명 |
    |-----------------------------|------|
    | 요청/응답 모두 개입 가능     | 헤더 추가, 응답 처리 등 |
    | 체인 구조                   | 여러 Interceptor를 연결 가능 (`chain.proceed()`) |
    | 순차 실행                  | 등록된 순서대로 실행됨 |
    | 인증 처리 용이              | JWT, OAuth 토큰 등 자동 삽입 가능 |
    | 로그, 디버깅에도 활용 가능   | OkHttpLoggingInterceptor |
    | 캐싱, 리트라이 커스터마이징 가능 | 고급 제어 가능 |

- 기본 구조
  + ```kotlin
    class AuthInterceptor(private val tokenProvider: () -> String) : Interceptor {
      override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
            .newBuilder()
            .addHeader("Authorization", "Bearer ${tokenProvider()}")
            .build()
        return chain.proceed(request)
      }
    }
    ```
  + 이 코드는 모든 요청에 Authorization 헤더를 자동으로 추가

- 샘플 코드
  + Retrofit + Interceptor 설정
    * ```kotlin
      val client = OkHttpClient.Builder()
        .addInterceptor(AuthInterceptor { getAccessToken() })
        .addInterceptor(HttpLoggingInterceptor().apply {
          level = HttpLoggingInterceptor.Level.BODY
        })
        .build()

      val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com")
        .client(client)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
      ```
  + 토큰 만료 시 자동 갱신
    * ```kotlin
      class TokenRefreshInterceptor(...) : Interceptor {
        override fun intercept(chain: Interceptor.Chain): Response {
            val request = chain.request()
            val response = chain.proceed(request)

            if (response.code == 401) {
                synchronized(this) {
                    val newToken = refreshAccessToken()
                    val newRequest = request.newBuilder()
                        .header("Authorization", "Bearer $newToken")
                        .build()
                    return chain.proceed(newRequest)
                } 
            }

           return response
        }
      }
      ```
    * ️만료된 토큰을 자동으로 갱신할 수 있음
    * 모든 API 호출이 통합된 방식으로 보안 처리됨

- 결론
  + | 용도          | Interceptor 활용 예                     |
    | ----------- | ------------------------------------ |
    | 인증 토큰 자동 추가 | 헤더에 Authorization 삽입                 |
    | 요청 공통 헤더    | User-Agent, Locale, App-Version 등 삽입 |
    | 응답 가공       | 응답 코드에 따른 처리                         |
    | 로그 기록       | `HttpLoggingInterceptor` 사용          |
    | 토큰 갱신       | `401` 응답 시 자동 재요청                    |
  + Interceptor는 네트워크의 관문이다. 요청과 응답 모두를 지켜보며 수정, 검사, 자동화할 수 있다. Retrofit을 쓰면 반드시 함께 써야 할 고급 툴

- 면접 관련 질문
  + Interceptor는 언제, 왜 사용하는가요?
    * Interceptor는 네트워크 요청이나 응답을 가로채서 공통 작업을 처리하거나 수정할 수 있도록 해주는 구조입니다.
      주로 다음과 같은 경우에 사용합니다:
      1. 모든 요청에 공통 헤더 추가 (예: Authorization, App-Version 등)
      2. 인증 토큰이 만료되었을 때 자동으로 갱신 처리
      3. API 요청/응답 로깅 및 디버깅
      4. 캐시 정책, 네트워크 리트라이 제어 등
    * 이런 공통 처리를 각 API마다 반복하지 않고 Interceptor로 한 곳에서 제어할 수 있어서, 유지보수성이 뛰어납니다.
  + addInterceptor()와 addNetworkInterceptor()의 차이는 무엇인가요?
    * addInterceptor()는 전체 요청/응답을 가로챌 수 있는 애플리케이션 레벨의 인터셉터입니다.
      1. 요청이 캐시에서 처리되든 네트워크로 가든 모두 동작합니다.
    * addNetworkInterceptor()는 오직 네트워크를 통해 실제 서버와 통신할 때만 동작합니다.
      1. 캐시에서 응답이 반환될 경우, 호출되지 않습니다.
    * 실무에서는 대부분 addInterceptor()를 사용하며, 네트워크 상세 조작이 필요할 때만 addNetworkInterceptor()를 사용합니다.
  + 토큰 인증이 필요한 앱에서 Interceptor를 어떻게 활용해봤나요?
    * Interceptor를 활용해서 모든 API 요청에 자동으로 Bearer 토큰을 추가했습니다.
    * 또한, 서버에서 401 응답을 받았을 때는 Interceptor 내에서 토큰을 자동 갱신하고 원래 요청을 재시도하는 로직도 구현했습니다.
    * 이 구조 덕분에 API 호출부에서는 인증 처리 코드를 신경 쓸 필요가 없었고, 보안 로직을 중앙 집중화할 수 있었습니다.
  + Interceptor에서 무한 루프나 요청 중복 문제가 발생할 수 있는 경우는 언제인가요?
    * Interceptor에서 401 처리 후 토큰을 갱신하고 요청을 다시 보낼 때,
      → 재시도 로직이 반복적으로 호출되면 무한 루프가 발생할 수 있습니다.
    * 이걸 방지하려면 다음과 같은 방어 로직을 넣습니다:
      1. 새로 보낸 요청에는 토큰 갱신 플래그를 붙이거나,
      2. 재귀 요청 횟수를 제한하거나,
      3. chain.request()의 URL이나 Tag로 이미 재시도했는지 확인
    * 이런 방어 코드를 통해 네트워크 폭주나 앱 크래시를 막을 수 있습니다.
  + Retrofit과 OkHttp의 관계는 무엇인가요?
    * Retrofit은 고수준 HTTP 클라이언트이며, 내부적으로 OkHttp를 기반으로 동작합니다.
      1. 요청 전송, 응답 수신, 캐싱, 연결 풀링 등 낮은 수준의 네트워크 처리는 모두 OkHttp가 수행하고,
      2. Retrofit은 이를 래핑해서 인터페이스 기반 API 호출, Gson 파싱, 코루틴 연동 같은 기능을 제공합니다.
    * Interceptor는 Retrofit이 아니라 OkHttp의 기능이며, Retrofit에서 OkHttpClient를 설정해줄 때 함께 구성합니다.


---


### Notification
- 개요
  + 안드로이드 알림은 앱이 사용자에게 중요하거나 시기적절한 정보를 제공하는 방법 중 하나입니다.
  + 앱이 현재 실행 중이지 않거나 백그라운드에 있을 때도 사용자에게 알림을 표시하여 상호작용을 유도할 수 있습니다.

- 알림의 목적 및 중요성
  + 시기적절한 정보 제공 : 사용자에게 새로운 메시지, 예정된 이벤트, 다운로드 완료 등과 같은 중요한 정보를 즉시 알립니다.
  + 사용자 참여 유도 : 알림을 통해 사용자가 앱을 다시 실행하거나 특정 작업을 수행하도록 유도할 수 있습니다.
  + 백그라운드 작업 알림 : 음악 재생, 파일 다운로드, 위치 추적과 같은 백그라운드 작업을 사용자에게 알리고 제어할 수 있는 인터페이스를 제공합니다.

- 알림의 기본 구성 요소
  + 알림은 시스템 템플릿에 의해 디자인되며, 앱은 템플릿의 각 부분에 콘텐츠를 정의합니다.

- 알림 채널 - Android 8.0 이상 필수
  + 안드로이드 8.0부터 모든 알림은 반드시 채널을 통해 게시되어야합니다.
  + 채널 생성 : 앱에서 보내는 알림의 종류별로 채널을 만들어야 합니다.
  + 사용자 제어 : 사용자는 시스템 설정에서 각 채널별로 알림 동작을 직접 제어할 수 있습니다.
  + 중요도 : 채널을 만들 때 중요도를 설정해야 하며, 이는 알림이 사용자에게 얼마나 방해가 될지를 결정합니다.

- 알림 생성 및 표시 단계
  + AndroidX Core 라이브러리 추가 : NotificationCompat API를 사용하기 위해 build.gradle 파일에 의존성을 추가합니다
  + 런타임 권한 요청 : Android 13 이상을 타겟팅하는 앱은 알림을 게시하기 위해 POST_NOTIFCATIONS 런타임 권한을 사용자로부터 받아야합니다.
  + 알림 콘텐츠 설정 : NotificationCompat.Builder를 사용하여 알림의 내용을 구성합니다.
  + 알림 채널 생성 및 중요도 설정 : 위에서 설명한 대로 채널을 생성하고 NotificationManager에 등록합니다.
  + 알림의 탭 동작 설정 : 사용자가 알림을 탭했을때 실행될 작업을 PeddingIntant를 사용하여 설정합니다.
  + 알림 표시 : NotificationManagerCompat를 사용하여 알림을 시스템에 게시합니다.

- 알림의 추가 기능
  + 확장 가능한 알림 : 더 많은 정보를 표시하기 위해 알림을 확장할 수 있습니다.
  + 알림 작업 : 알림에 직접 버튼을 추가하여 사용자가 앱을 열지 않고도 빠른 작업을 수행할 수 있도록 합니다.
  + 진행률 표기시 : 파일 다운로드와 같이 시간이 오래걸리는 작업의 진행 상태를 알림에 표시할 수 있습니다.
  + 알림 그룹 : 여러 알림을 하나의 그룹으로 묶어 표시할 수 있습니다.

- 면접 질문
  + Notification을 구현하는 기본적인 단계는 무엇인가요?
    * NotificationCompat.Builder 사용, 채널 생성, 콘텐츠 설정, 탭 액션(PendingIntent), NotificationManagerCompat.notify().
  + Foreground Service에서 Notification을 사용하는 이유는 무엇이며, 어떻게 구현해야 하나요? 일반 Notification과의 차이점은 무엇인가요?
    + 백그라운드 작업이 진행 중임을 사용자에게 명확히 알리고, 시스템에 의해 작업이 중단되는 것을 방지 합니다. startForeground() 호출 시 Notification 필수로 노출되어야 하며, 사용자가 직접 해제하기 전까지 사라지지 않아야 함.


---


### GSON & Moshi
- 직렬화(Serialization)
  + `직렬화`는 객체를 JSON, XML 등 문자열 형태로 변환하는 과정
  + `역직렬화(Deserialization)`는 문자열(JSON 등)을 다시 객체로 복원하는 과정
  + Android 앱에서는 API 통신 시 데이터를 객체 <-> JSON으로 변환할 일이 많은데 이럴 때 주로 쓰이는 것이 `Gson`, `Moshi` 같은 JSON 파서다.

- 공통점
  + | 항목 | 설명 |
    |------|------|
    | JSON <-> 객체 매핑 | 자동 지원 (Serialize/Deserialize) |
    | 어노테이션 지원 | `@SerializedName`, `@Json` 등 |
    | Null-safe 처리 가능 | 옵션 또는 어댑터로 제어 가능 |
    | Kotlin + Java 호환 | 모두 가능 |
    | Retrofit 연동 가능 | `converter-gson`, `converter-moshi`|

- 차이점
  + | 항목           | **Gson**                           | **Moshi**                           |
    |----------------|------------------------------------|-------------------------------------|
    | 개발 주체       | Google                             | Square (Retrofit 만든 회사)         |
    | Kotlin 지원     | 제한적 (Kotlin에 최적화 X)         | 우수함 (`KotlinJsonAdapterFactory`) |
    | 속도/성능       | 느린 편                            | 빠르고 가벼움                        |
    | Null 처리       | 더 관대함                          | 더 엄격함                            |
    | 타입 어댑터     | 복잡함 (`TypeToken` 필요)          | 심플하고 타입 안전함                |
    | 확장성          | 풍부한 기능, 오래된 레거시 지원    | 모던 설계, 가볍고 간결함            |
    | 빌드 사이즈     | 크다                               | 작다                                 |

- 샘플 코드 비교
  + GSON
    * ```kotlin
      data class User(
        @SerializedName("user_name") val name: String,
        val age: Int
      )

      val gson = Gson()
      val json = """{"user_name": "Lee", "age": 28}"""
      val user = gson.fromJson(json, User::class.java)
      val jsonOut = gson.toJson(user)
      ```
  + Moshi
    * ```kotlin
      @JsonClass(generateAdapter = true)
      data class User(
        @Json(name = "user_name") val name: String,
        val age: Int
      )

      val moshi = Moshi.Builder()
        .add(KotlinJsonAdapterFactory())
        .build()

      val adapter = moshi.adapter(User::class.java)
      val user = adapter.fromJson("""{"user_name": "Lee", "age": 28}""")
      val jsonOut = adapter.toJson(user)
      ```

- 선택 기준
  + | 상황                          | 추천                              |
    | --------------------------- | ------------------------------- |
    | Kotlin 프로젝트 (특히 data class) | ✅ Moshi                         |
    | 레거시 Java 프로젝트               | ✅ Gson                          |
    | 가볍고 빠른 파싱이 중요한 경우           | ✅ Moshi                         |
    | 기존 Gson 기반 코드 유지            | ✅ Gson (단, 필요시 Moshi 마이그레이션 고려) |

- 면접 질문
  + @SerializedName (GSON) / @Json (Moshi) 어노테이션은 어떤 역할을 하나요?
    * 이 어노테이션들은 JSON 데이터의 키 이름과 Kotlin/Java 객체의 필드(프로퍼티) 이름이 다를 경우 이를 매핑해주는 역할을 합니다.
  + JSON 데이터의 특정 필드가 null일 경우, GSON/Moshi는 어떻게 처리하나요? 기본값을 설정하는 방법이 있나요?
    * GSON은 기본적으로 필드 타입이 객체(Object) 타입이면 null로 설정합니다.
    * Moshi는 Kotlin data class의 프로퍼티에 기본값을 지정해두면, JSON에 해당 키가 없거나 값이 null일 때 그 기본값을 사용합니다.


---


### View & ViewGroup
- 개요
  + 안드로이드 UI를 구성하는 가장 기본적인 빌딩 블록은 View와 ViewGroup입니다. 이 둘의 관계와 역할을 이해하는 것은 Android UI 개발의 핵심입니다.

- View
  + 개요
    * View클래스는 사용자 인터페이스의 기본 구성요소입니다.
    * 화면에서 사용자와 상호작용할 수 있는 모든 것의 기본 클래스입니다.
    * View는 화면의 특정 직사각형 영역을 차지하며, 해당 영역의 그리기와 이벤트 처리를 담당합니다.
  + 주요 특징 및 역할
    * 화면의 직사각형 영역 차지: 모든 View는 화면에 자신만의 공간을 가집니다. 이 공간의 크기와 위치는 레이아웃 과정에서 결정됩니다.
    * 그리기 (Drawing): View는 자신의 모양과 내용을 화면에 그립니다. onDraw(Canvas canvas) 메서드를 오버라이드하여 커스텀 그리기를 구현할 수 있습니다. 시스템은 필요할 때 이 메서드를 호출하여 View를 다시 그리도록 요청합니다.
    * 이벤트 처리 (Event Handling): 사용자의 터치, 키 입력 등과 같은 이벤트를 감지하고 반응합니다. onTouchEvent(MotionEvent event), onClickListener, onKeyListener 등을 통해 이벤트를 처리합니다.
    * 속성 (Properties): View는 다양한 속성을 가집니다. (예: ID, 배경색, 패딩, 가시성 등). 이러한 속성은 XML 레이아웃 파일이나 Kotlin/Java 코드에서 설정할 수 있습니다.
    * 측정 (Measuring): onMeasure(int widthMeasureSpec, int heightMeasureSpec) 메서드를 통해 자신의 크기를 결정합니다. 부모 ViewGroup으로부터 크기 제약 조건을 전달받아 자신의 콘텐츠와 제약 조건에 맞는 크기를 계산합니다.
    * 레이아웃 (Layout): onLayout(boolean changed, int left, int top, int right, int bottom) 메서드를 통해 화면 내 자신의 위치와 크기를 최종적으로 할당받습니다. 이 과정은 부모 ViewGroup에 의해 관리됩니다.

- ViewGroup
  + 개요
    * ViewGroup은 View 클래스와 특별한 서브클래스입니다.
    * ViewGroup의 주요 역할은 다른 View 객체들을 담는 컨테이너 역할을 하는 것입니다.
    * 즉, ViewGroup은 UI 요소들을 그룹화하고 이들의 레이아웃을 관리합니다.
  + 주요 특징 및 역할
    * 자식 View 포함 (Container): 여러 개의 View 또는 다른 ViewGroup을 자식으로 가질 수 있습니다. 이를 통해 복잡한 UI 계층 구조를 만들 수 있습니다.
    * 레이아웃 관리 (Layout Management): ViewGroup은 자식 View들의 크기를 측정하고 화면에 배치하는 방법을 결정합니다. 각 ViewGroup의 서브클래스는 자신만의 레이아웃 정책을 가집니다.
    * 이벤트 전달 (Event Dispatching): ViewGroup은 터치 이벤트와 같은 사용자 이벤트를 받으면, 해당 이벤트가 발생한 위치에 있는 적절한 자식 View에게 이벤트를 전달할지, 아니면 자신이 직접 처리할지를 결정합니다. (onInterceptTouchEvent(), dispatchTouchEvent())
    * 계층 구조 (Hierarchy): ViewGroup은 다른 ViewGroup을 자식으로 가질 수 있으므로, UI 요소들은 트리(tree)와 같은 계층 구조를 형성합니다. 이 구조의 최상위에는 일반적으로 DecorView라는 특별한 ViewGroup이 있으며, 그 아래에 개발자가 정의한 레이아웃이 배치됩니다.

- View와 ViewGroup의 관계 요약
  + View는 개별 UI 요소입니다.
  + ViewGroup은 이러한 View들을 담는 컨테이너이며, 이들의 배치 방식을 결정합니다.
  + ViewGroup은 View를 상속받으므로, ViewGroup 자체도 하나의 View로 취급될 수 있습니다. 
  + UI는 View와 ViewGroup의 계층적인 트리 구조로 구성됩니다. ViewGroup은 다른 View나 ViewGroup을 자식으로 가질 수 있습니다.

- 면접 질문
  + View와 ViewGroup의 차이점은 무엇인가요?
    * View는 화면에 그려지고 사용자와 상호작용하는 기본적인 UI 구성요소 입니다. 예를들어 TextView, Button 등이 있습니다. 
      반면 ViewGroup은 다른 View들을 담는 컨테이너 역할을 하는 특별한 종류의 View 입니다. 예를들어 LinearLayout, RelativeLayout 등이 있습니다.
  + onMeasure() 메서드에서 MeasureSpec은 무엇이며 어떻게 사용되나요?
    * MeasureSpec은 부모 ViewGroup이 자식 View에게 전달하는 크기 제약 조건입니다. 정수 값으로 인코딩되며, 모드(Mode)와 크기(Size) 두 가지 정보를 담고 있습니다.


---


### jar, aar, apk
- JAR (Java ARchive)
  + 정의
    *  Java 클래스와 메타데이터를 압축한 라이브러리 패키지
  + 구성
    * .class 파일 (컴파일된 Java 바이트코드)
    * META-INF/MANIFEST.MF
    * 일반 리소스 파일(.properties, .xml 등) 포함 가능
    * ❌ Android 리소스(res/, AndroidManifest.xml) 포함 불가
  + 사용처
    * 순수 Java 코드 기반 라이브러리 (ex: Gson, Apache Commons, OkHttp JAR 버전)

- AAR (Android ARchive)
  + 정의
    *  안드로이드 라이브러리 모듈을 패키징한 파일 (JAR의 Android 확장 버전)
  + 구성
    * classes.jar (Java/Kotlin 코드)
    * res/, assets/, jni/
    * AndroidManifest.xml
    * proguard.txt 등 추가 설정 파일
  + 사용처
    * Android 전용 모듈을 라이브러리 형태로 배포
    * 다른 프로젝트에서 공통 UI, SDK, 커스텀 컴포넌트 재사용
    
- APK (Android Package)
  + 정의
    * 안드로이드 앱 최종 패키지 (사용자 단말기에 설치 가능한 실행 파일)
  + 구성
    * classes.dex (Dalvik/ART 바이트코드)
    * res/, assets/, AndroidManifest.xml
    * META-INF/ (서명 정보)
    * lib/ (네이티브 라이브러리)
  + 사용처
    * 최종적으로 배포되는 앱 파일 (Google Play 스토어 업로드 및 단말기 설치)
    
- 비교표
  + | 항목             | JAR (Java Archive) | AAR (Android Archive) | APK (Android Package) |
    | -------------- | ------------------ | --------------------- | --------------------- |
    | 목적             | Java 라이브러리         | Android 라이브러리         | 앱 최종 실행 패키지           |
    | 포함 가능 요소       | Java 코드만           | 코드 + 리소스 + Manifest   | 코드 + 리소스 + 서명         |
    | 설치 가능 여부       | ❌                  | ❌                     | ✅                     |
    | 리소스 포함 여부      | ❌                  | ✅                     | ✅                     |
    | Manifest 포함 여부 | ❌                  | ✅                     | ✅                     |
    | 사용 대상          | Java 애플리케이션        | Android 프로젝트 공유 모듈    | 사용자 단말기 설치 앱          |

- APK 빌드 과정
  + ```text
    .java / .kt (소스 코드)
        │
        ▼
    .class (Java 바이트코드)     ← javac / kotlinc 컴파일러
        │
        ▼
    .jar (Java Archive)        ← 클래스 + 메타데이터 묶음
        │
        ▼
    .dex (Dalvik Executable)   ← D8/R8 (ART/Dalvik VM용 변환)
        │
        ▼
    APK (Android Package)       ← .dex + res/ + Manifest + assets/ + lib/ + META-INF
          │
          ▼
    (서명 & zipalign)           ← 서명(Signature) + 최적화
          │
          ▼
    최종 설치 파일 (.apk)         ← 사용자 단말기에 설치 가능
    ```
  + 소스 코드 (.java / .kt)
    * 개발자가 작성한 코드
  + .class (바이트코드)
    * javac (Java), kotlinc (Kotlin) 컴파일러가 바이트코드로 변환
  + .jar (Java Archive)
    * 여러 .class 파일과 메타데이터를 묶은 라이브러리 형태
  + .dex (Dalvik Executable)
    * D8 (또는 R8: 최적화/압축 포함)이 .class → .dex로 변환
    * 안드로이드 런타임(ART/Dalvik VM)에서 실행 가능한 코드
  + APK 패키징
    * classes.dex + res/(리소스) + AndroidManifest.xml + assets/ + lib/
    * 모든 앱 구성 요소를 하나로 묶음
  + 서명(Signing) & zipalign 최적화
    * Google Play에 배포하거나 단말기에 설치하려면 반드시 앱에 서명(Signature) 필요
    * zipalign은 압축을 정렬해 런타임 성능 최적화
  + 최종 APK
    * 사용자가 설치할 수 있는 실제 앱 파일

- 면접 질문
  + JAR, ARR, APK의 차이가 무엇일까요?
    * JAR : Java 아카이브로, 컴파일된 .class 파일과 메타정보만 포함하며, 안드로이드 리소스는 포함할 수 없습니다. 주로 Java 라이브러리에서 사용됩니다.
    * AAR: Android 전용 아카이브로, classes.jar 외에도 res/, AndroidManifest.xml, assets/ 등을 포함할 수 있어 Android 라이브러리 모듈 배포에 사용됩니다.
    * APK: Android Package로, 실제 앱을 설치할 수 있는 최종 패키지입니다. 실행 파일(.dex), 리소스, 서명 정보까지 모두 포함합니다.
  + AAR 파일은 어떤 경우에 사용하나요?
    * 안드로이드 앱에서 재사용 가능한 모듈을 분리해 라이브러리로 만들고, 이를 다른 프로젝트에 배포하거나 의존성으로 추가할 때 사용됩니다.
    * ex) 커스텀 버튼 UI, 공통 ViewModel, 인증 SDK 등을 AAR로 만들어 다른 앱에서 활용할 수 있습니다.
  + AAR과 JAR의 주요 차이점은 무엇인가요?
    * AAR은 Android 라이브러리 전용 형식으로, 코드뿐 아니라 리소스(res/), AndroidManifest, assets 등을 포함할 수 있습니다.
    * 반면 JAR은 Java 코드만 포함하므로, Android 앱에서 UI 구성이나 리소스를 사용하는 경우 적합하지 않습니다.


---


### 안드로이드 앱 서명 방식 v1, v2, v3 비교
- 개요
  + 안드로이드 앱 서명 방식은 앱의 무결성을 보장하고 안전하게 업데이트할 수 있도록 하는 중요한 메커니즘입니다.

- JAR 서명(v1 스킴)
  + 방식
    * APK 파일 내의 META-INF 디렉터리에 있는 파일들(MANIFEST.MF, CERT.SF, CERT.RSA)을 통해 서명합니다.
    * APK 내의 각 파일에 대한 헤시값을 MANIFEST.MF에 저장하고, 이 MANIFEST.MF파일 자체와 다른 메타데이터를 CERT.SF에 해시하여 저장합니다.
    * 마지막으로 CERT.SF 파일과 공개키 인증서를 CERT.RSA 파일에 저장하여 서명합니다.
  + 특징
    * 전체 APK파일이 아닌, 압축 해제된 파일들의 내용을 기반으로 서명합니다.
    * APK 파일의 일부만 변경되어도 서명이 유효하지 않게 됩니다.
    * Android 초기버전부터 사용된 방식입니다.
  + 단점
    * 서명 확인 속도가 느립니다. APK 전체를 압축 해제하고 각 파일의 해시를 검증해야 하기 때문입니다.
    * APK 파일 자체의 무결성을 완벽하게 보장하지 못합니다. 예를 들어, ZIP파일 구조의 특정 영역을 수정해도 서명 검증을 통과할 수 있는 취약점이 있었습니다.

- APK 서명 스킴 v2(v2 스킴)
  + 도입시기 : 안드로이드 7.0(Nougat)
  + 방식
    * APK 파일 전체의 바이너리 내용에 대해 해시를 계산하고 서명합니다.
    * 서명 정보는 APK 파일 내의 "APK 서명 블록"이라는 별도의 섹션에 저장됩니다.
    * 이 블록은 ZIP 파일의 Central Directory 바로 앞에 위치합니다.
  + 특징
    * 전체 파일 무결성 : APK 파일의 어떤 부분이든 변경되면 서명이 무효화됩니다. 이 과정을 통해 v1의 취약점을 해결했습니다.
    * 검증 속도 향상 : APK 전체를 압축 해제할 필요 없이 APK 서명 블록만 읽어 서명을 빠르게 검증할 수 있습니다.
    * v1 서명과 함께 사용될 수 있습니다.
  + 장점
    * 보안 강화 : APK 변조 방지 능력이 향상되었습니다.
    * 설치 시간 단축 : 서명 검증 속도가 빨라져 앱 설치 시간이 줄어듭니다.

- APK 서명 스킴 v3(v3 스킴)
  + 도입시기 : 안드로이드 9.0(Pie)
  + 방식
    * v2 스킴과 유사하게 APK 서명 블록을 사용하지만, 추가적인 정보를 포함할 수 있도록 확장되었습니다.
    * 가장 큰 특징은 APK 키 회전 지원입니다.
  + 특징
    * APK 키 회전 지원 : 앱 업데이트 시 서명 키를 변경할 수 있는 기능을 제공합니다. 이전 키와 새 키 간의 신뢰 관계를 서명 블록 내에 기록하여, 키가 변경되어도 OS가 동일한 앱의 업데이트로 인식할 수 있게 합니다.
    * v3 서명 블록 내에는 지원되는 SDK 버전 범위와 각 버전에 대한 서명 정보가 포함될 수 있습니다.
    * v1, v2 서명과 함께 사용될 수 있습니다.
  + 장점
    * 보안 유연성 증대 : 서명 키가 유출되거나 만료될 경우, 앱의 연속성을 유지하면서 새 키로 안전하게 전환할 수 있습니다.
    * v2의 장점을 그대로 유지합니다.

- 비교요약
  + | 특징        | JAR 서명 (v1)                  | APK 서명 스킴 v2         | APK 서명 스킴 v3                  | 
    |-----------|------------------------------|----------------------|-------------------------------| 
    | 도입 시기     | Android 초기 버전                | Android 7.0 (Nougat) | Android 9 (Pie)               | 
    | 서명 대상     | APK 내 압축 해제된 파일들             | APK 파일 전체 바이너리       | APK 파일 전체 바이너리                | 
    | 서명 위치     | META-INF 디렉터리 내 파일들          | APK 서명 블록            | APK 서명 블록 (v2와 동일 위치, 확장된 구조) | 
    | 무결성 보장    | 파일 내용 기반 (ZIP 구조 일부 변경에 취약)  | APK 파일 전체 (변조 방지 강화) | APK 파일 전체 (변조 방지 강화)          | 
    | 검증 속도     | 느림 (전체 압축 해제 및 파일 검증 필요)     | 빠름 (APK 서명 블록만 확인)   | 빠름 (APK 서명 블록만 확인)            | 
    | 주요 기능 추가  | -                            | 전체 파일 무결성, 검증 속도 향상  | APK 키 회전 지원                   | 
    | 하위 호환성    | -                            | v1과 함께 사용 가능         | v1, v2와 함께 사용 가능              |

- 면접 질문
  + 안드로이드 앱 서명이 왜 중요한가요?
    * 안드로이드 앱 서명은 크게 두 가지 중요한 목적을 가집니다.
    * 앱 무결성 보장과 안전한 업데이트로 나눕니다.
    * 앱 무결성은 앱이 배포된 후 악의적으로 변조되지 않았음을 보장합니다. 서명을 통해 사용자는 다운로드한 앱이 개발자가 배포한 원본 그대로임을 신뢰할 수 있습니다.
    * 안전한 업데이트는 동일한 개발자가 만든 앱의 업데이트임을 시스템이 확인할 수 있게 합니다. 서명 키가 일치해야만 앱 업데이트가 가능하므로, 다른 개발자가 만든 악성 앱으로 대체되는 것을 방지합니다.
  + v1 서명 방식과 v2 서명 방식의 가장 큰 차이점은 무엇이며, v2 방식이 가지는 이점은 무엇인가요?
    * v1과 v2 서명 방식의 가장 큰 차이점은 서명 대상과 범위입니다.
    * v1은 APK 파일 내부에 압축된 개별 파일들 각각에 대해 서명을 진행하고, 이 정보를 META-INF 폴더에 저장합니다.
    * v2는 APK 파일 전체의 바이너리 내용에 대해 해시를 계산하고 서명합니다. 이 서명 정보는 APK 파일 내 별도의 "APK 서명 블록"에 저장됩니다.
  + APK 서명 스킴 v3에서 추가된 핵심 기능은 무엇이며, 이 기능이 왜 필요한가요?
    * APK 서명 스킴 v3에서 추가된 핵심 기능은 APK 키 회전 지원입니다.
    * 첫째 서명 키 보안 강화 입니다. 만약 앱 서명에 사용된 개인 키가 유출되거나 손상될 경우, 기존에는 해당 앱의 업데이트를 안전하게 배포하기 매우 어려웠습니다. 
      키 회전 기능을 사용하면, 이전 키와의 신뢰 관계를 증명하면서 새로운 키로 앱을 서명하여 업데이트를 계속할 수 있습니다.
    * 둘째 장기적인 앱 관리는 시간이 지남에 따라 보안 표준이 강화되거나, 회사 정책 변경 등으로 인해 서명 키를 변경해야 할 필요가 생길 수 있습니다. 
      키 회전은 이러한 상황에서 앱의 연속성을 유지하며 안전하게 키를 전환할 수 있는 방법을 제공합니다.


---


### 안드로이드 성능 최적화 방법
- 🧵 메인 스레드(UI Thread) 최적화
  + 규칙: 메인 스레드에서 오래 걸리는 작업 금지 (ANR 방지)
  + ❌ 피해야 할 작업
    * 네트워크 요청 (HTTP, 소켓 등)
    * DB I/O (Room, SQLite)
    * 대용량 파일 I/O
    * 복잡한 연산 (JSON 파싱, 이미지 처리 등)
  + ✅ 해결 방법
    * Coroutines (Dispatchers.IO/Main)
    * RxJava, WorkManager
    * Handler/Looper/Executor
  + ```kotlin
    fun fetchData() {
      val url = URL("https://example.com")
      val response = url.readText() // ⚠️ 메인 스레드 차단 → ANR 위험
    }

    fun fetchData() {
      lifecycleScope.launch(Dispatchers.IO) {
        val url = URL("https://example.com")
        val response = url.readText()

        withContext(Dispatchers.Main) {
            textView.text = response
        }
      }
    }
    ``` 

- 🎨 UI / 렌더링 최적화
  + 렌더링 속도 유지 (16ms 이내, 60fps)
  + ✅ 최적화 방법
    * 불필요한 Overdraw 방지 (배경 중첩 줄이기)
      1. Android Studio → Debug GPU Overdraw 확인
    * ConstraintLayout / MotionLayout 적극 활용 → 뷰 계층 단순화
    * RecyclerView → ViewHolder 재활용 / DiffUtil 사용
    * 큰 이미지 → Glide/Picasso/Fresco 같은 라이브러리 활용
    * ViewStub / include / merge 태그로 뷰 성능 개선
    * 애니메이션 → Lottie (JSON 기반), GPU 최적화 활용
  + ```kotlin
    // diffUtil 사용
    class MyAdapter : ListAdapter<User, MyViewHolder>(DiffCallback()) {
      class DiffCallback : DiffUtil.ItemCallback<User>() {
        override fun areItemsTheSame(old: User, new: User) = old.id == new.id
        override fun areContentsTheSame(old: User, new: User) = old == new
      }
    }

    // Overdraw 줄이기 -> merge 태그
    <merge xmlns:android="http://schemas.android.com/apk/res/android">
      <TextView android:id="@+id/title" ... />
      <ImageView android:id="@+id/icon" ... />
    </merge>
    ```  

- 🧠 메모리 최적화
  + ✅ 방법
    * 메모리 누수 방지
      1. static Context 보관 금지
      2. Handler → WeakReference 사용
      3. 리스너/콜백 해제 잊지 않기
      4. Fragment → ViewBinding 해제 (onDestroyView)
    * Bitmap 최적화
      1. inSampleSize 로 다운샘플링
      2. Glide → placeholder / thumbnail / cache 전략
    * LruCache 사용 (메모리 캐싱)
    * LeakCanary 같은 툴로 메모리 누수 탐지
  + ```kotlin
    // Bitmap Downsampling
    val options = BitmapFactory.Options().apply { inSampleSize = 4 }
    val bitmap = BitmapFactory.decodeResource(resources, R.drawable.large_img, options)

    // Leak 방지
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
      _binding = FragmentHomeBinding.inflate(inflater, container, false)
      return binding.root
    }

    override fun onDestroyView() {
      super.onDestroyView()
      _binding = null // 메모리 누수 방지
    }
    ``` 

- 🔋 배터리 최적화
  + 배터리는 앱 성능 평판에 직결
  + ✅ 방법
    * 불필요한 WakeLock 사용 금지
    * GPS / 센서 → 필요한 시간에만 등록/해제
    * WorkManager, JobScheduler 사용 (백그라운드 반복 작업 최적화)
    * Foreground Service → 꼭 필요한 경우만 사용
    * 네트워크 요청 → 배치 처리, 압축, 캐싱 활용
  + ```kotlin 
    val constraints = Constraints.Builder()
      .setRequiredNetworkType(NetworkType.UNMETERED)
      .setRequiresCharging(true)
      .build()

    val workRequest = PeriodicWorkRequestBuilder<SyncWorker>(15, TimeUnit.MINUTES)
      .setConstraints(constraints)
      .build()

    WorkManager.getInstance(context).enqueue(workRequest)
    ``` 

- 🌐 네트워크 최적화
  + ✅ 방법
    * Retrofit + OkHttp → Connection Pool / Caching / GZIP 압축
    * JSON 파싱 → Gson보다 Moshi / Kotlin Serialization (빠르고 가벼움)
    * 불필요한 중복 요청 방지 (Debounce/Throttle)
    * 이미지 → CDN 사용, WebP 변환, Cache-Control 헤더 적용
    * 서버와의 주기적 Sync → FCM(푸시) 활용 (폴링 최소화)
  + ```kotlin
    val cacheSize = (10 * 1024 * 1024).toLong()
    val cache = Cache(File(context.cacheDir, "http_cache"), cacheSize)

    val client = OkHttpClient.Builder()
      .cache(cache)
      .addInterceptor { chain ->
          val response = chain.proceed(chain.request())
          response.newBuilder()
              .header("Cache-Control", "public, max-age=60")
              .build()
      }
      .build()
    ``` 

- ⚡ 빌드 & 코드 최적화
  + ✅ 방법
    * ProGuard / R8 → 코드 난독화 + 불필요한 코드 제거 + 사이즈 축소
    * 모듈화 / Dynamic Delivery → 필요한 기능만 설치
    * Reified 타입, Inline 함수 사용 (Kotlin)
    * 불필요한 Reflection / 리플렉션 기반 DI 줄이기 (Hilt, Koin 최적 활용)
  + ```gradle
    minifyEnabled true
    shrinkResources true
    proguardFiles getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro"
    ```
  + ```kotlin
    inline fun <reified T> Gson.fromJson(json: String): T =
      this.fromJson(json, object : TypeToken<T>() {}.type)
    ```

- 면접 관련 질문
  + 앱 성능 최적화에서 가장 중요한 포인트는 무엇인가요?
    * 메인 스레드를 가볍게 유지하는 것 (UI 블록 방지)
    * 네트워크/DB/파일 I/O는 백그라운드에서 처리
  + 메모리 누수를 어떻게 방지할 수 있나요?
    * Context/Activity 참조 관리, WeakReference, 리스너 해제, LeakCanary 활용
  + RecyclerView 성능 최적화 방법은?
    * ViewHolder 패턴, DiffUtil, ListAdapter, setHasFixedSize(true), itemView 재활용


---


### AAB vs APK
- AAB(안드로이드 앱 번들) — 개념 & 특징
  + 정의  
    * 개발자가 빌드한 모든 코드·리소스·구성(모듈)을 묶은 번들 파일(.aab).
  + 핵심 아이디어
    * Play가 번들을 받아 기기(ABI, 언어, 화면밀도 등)별로 최적화된 APK(또는 split APK 세트) 를 생성해 제공 → 사용자는 필요한 리소스/코드만 다운로드.
  + 지원 기능
    * Dynamic Feature Modules: 기능을 install-time 또는 on-demand로 분리 가능
    * Configuration splits: ABI, density, language에 따른 분할 배포.
    * Play App Signing: Play가 최종 APK 서명(대부분의 경우) — 개발자는 업로드 키를 사용.
  + 흔히 사용하는 워크플로: `./gradlew bundleRelease` → `app-release.aab` 생성 → Play Console 업로드

- APK — 개념 & 특징
  + 정의
    * 설치 가능한 표준 안드로이드 패키지(.apk).
  + 구성  
    * `classes.dex`, `res/`, `AndroidManifest.xml`, `lib/`, `assets/`, `META-INF(서명)` 등.
  + 특징
    * 바로 adb install로 단말에 설치 가능.
    * Play로 업로드할 수도 있고, 다른 스토어/사이드로드용으로 배포 가능.
    * 단일 파일로 모든 리소스 포함 → 사용자에게 불필요한 리소스까지 전달될 가능성.

- 비교
  + | 항목       |                                          AAB (Android App Bundle) | APK (Android Package)                            |
    | -------- | ----------------------------------------------------------------: | ------------------------------------------------ |
    | 목적       |                              Play에서 앱을 **기기 맞춤형으로 생성·배포**하기 위한 번들 | 단말에 **설치 가능한 최종 패키지**                            |
    | 포함 요소    |   code (classes.jar), res, manifest, native libs, dynamic modules | classes.dex, res, manifest, native libs, 서명 정보 등 |
    | 배포 방식    | Google Play(또는 bundletool 이용)에서 기기별 APK(또는 split APK)를 생성·서명 후 배포 | 개발자가 직접 서명해서 배포(스토어/사이드로드)                       |
    | 설치 가능 여부 |               직접 설치 불가 — bundletool이나 Play Console을 통해 APK로 변환/설치 | 바로 설치 가능 (`adb install`)                         |
    | 장점       |          사용자 다운로드/설치 크기 감소, Dynamic Feature(온디맨드 모듈) 지원, Play 최적화 | 단순함, Play 외 배포(사이드로드, 다른 스토어)에 유리                |
    | 단점       |                                로컬 설치/테스트가 번거로움(번들툴 필요), Play에 의존적 | 사용자에게 전체 패키지 전달 → 큰 용량, 불필요한 리소스 포함              |
 
- Play Console과 App Signing
  + Play App Signing: AAB를 업로드하면 Play가 기기별 APK를 빌드하고 Play의 앱 서명 키로 최종 APK에 서명합니다.
    * 개발자는 upload key로 AAB를 업로드(서명)하고, Play가 App Signing Key로 재서명.
    * 장점: 키 관리 편리, 향후 키 교체 지원 등.
  + 테스트 방법
    * Internal testing / Closed testing 트랙을 통해 실제 Play에서 기기별 APK 생성 후 검증.
    * 로컬 테스트는 bundletool 사용.

- AAB의 장점 / 단점 — 실무 요약
  + 장점  
    * 다운로드/설치 크기 감소: 기기별로 필요한 리소스만 전달 → 사용자 경험 개선(데이터/용량 절약)
    * Dynamic Delivery: 기능을 모듈화해 on-demand로 제공 → 초기 설치 용량 축소
    * Play 최적화: Play가 기기 맞춤 APK를 생성·최적화
    * 보안/키 관리: Play App Signing으로 키 관리를 쉽게
  + 단점 / 고려사항
    * 직접 설치 불가: 로컬 테스트/사이드로드 시 번들툴 필요 → 개발/테스트 흐름 변경 필요
    * Play 의존성: AAB의 가장 큰 장점은 Play에서 발휘 → 다른 스토어 배포 시 추가 작업 필요
    * 테스트 복잡성: 다양한 split 조합·Dynamic feature 테스트 필요
    * 구현·운영 복잡성: 모듈화 설계·버전 관리·on-demand API 사용 등의 추가 비용

- AAB로 전환할 때 체크리스트 & 모범 사례
  + ./gradlew bundleRelease가 정상 빌드되는지 확인
  + Play App Signing에 등록(필요한 경우 upload key 준비)
  + 중요 라이브러리(native .so 등)가 ABI별로 잘 분리되는지 확인
  + Dynamic Feature 도입 시: 기능 경계, 의존성, 런타임 로드/언로드 시나리오 설계
  + 자동화(CI): 번들 빌드 및 Play Upload(Gradle Play Publisher, Fastlane) 도입
  + 테스트: internal testing 트랙 + bundletool 로컬 테스트
  + 리소스/문자열 다국어 분리 → language split으로 사이즈 절약 가능
  + ProGuard/R8로 코드 축소 & 리소스 shrink 활성화

- 면접 관련 질문
  + AAB와 APK의 가장 큰 차이는 무엇인가요?
    * APK는 설치 가능한 단일 패키지이고, AAB는 앱의 모든 코드·리소스를 묶은 번들입니다. 
    * Play가 AAB를 받아서 기기별로 최적화된 APK(또는 split APK)를 생성·서명해 배포한다는 점이 핵심 차이입니다.
  + AAB를 업로드하면 누가 서명하나요? 개발자는 추가로 할 일이 있나요?
    * 보통 Google Play가 최종 APK에 Play App Signing 키로 서명합니다. 
    * 개발자는 로컬에서 upload key로 AAB를 서명해 업로드하고, Play에서 재서명합니다. (Play App Signing을 사용하지 않는 옵션도 있지만 Play에서 서명 관리하는 것이 권장됩니다.)
  + AAB로 전환했을 때 테스트/로컬 설치는 어떻게 하나요?
    * AAB는 직접 adb install 불가합니다.
    * 로컬 테스트는 bundletool을 사용해 .apks를 생성하고 bundletool install-apks로 설치하거나 unzip 후 universal.apk를 추출해 설치합니다.
    * 또 Play Console의 internal testing 트랙을 활용하면 Play에서 기기별 APK를 생성·배포해서 테스트할 수 있습니다. 