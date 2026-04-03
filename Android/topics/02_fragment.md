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
