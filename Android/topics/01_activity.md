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
