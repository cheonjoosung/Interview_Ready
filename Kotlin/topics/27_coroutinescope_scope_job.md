### CoroutineScope & Scope & Job 관리
- CoroutineScope 정의
  + 코틀린 코루틴에서 구조화된 동시성을 구현하는 핵심요소 입니다.
  + CoroutineScope는 새로운 코루틴을 시작하고 이들의 생명주기를 관리하는 범위를 정의합니다.

- CoroutineScope 개념
  + CoroutineScope는 CoroutineContext의 한 종류로, 특히 Job을 포함하여 코루틴의 생명주기를 제어합니다. 
  + CoroutineScope 내에서 시작된 모든 코루틴은 해당 스코프의 Job의 자식으로 관리됩니다.

- CoroutineScope 주요역할
  + 코루틴의 생명주기 관리
    * CoroutineScope는 자신이 관리하는 모든 자식 코루틴의 생명주기를 추적합니다.
    * 스코프가 취소되면(scope.cancel()), 해당 스코프 내에서 실행 중이던 모든 자식 코루틴도 함께 취소됩니다. 
      이는 리소스 누수를 방지하고 코루틴을 깔끔하게 정리하는 데 매우 중요합니다. 
    * Android의 ViewModel에서 viewModelScope를 사용하면, ViewModel이 소멸될 때 viewModelScope도 취소되어 관련된 모든 코루틴이 자동으로 정리됩니다.
  + 구조화된 동시성 제공
    * "구조화된 동시성"은 코루틴이 명확한 부모-자식 관계를 가지며, 부모 코루틴(또는 스코프)이 모든 자식 코루틴의 완료를 기다리거나 함께 취소될 수 있도록 하는 프로그래밍 모델입니다.
    * CoroutineScope는 이러한 구조를 만들어, 코루틴이 "어디선가 백그라운드에서 떠도는" 것을 방지하고, 예측 가능하며 관리하기 쉬운 동시성 코드를 작성할 수 있게 합니다.
    * 부모 스코프의 Job이 완료되기를 기다리면(scope.coroutineContext[Job]?.join()), 모든 자식 코루틴이 완료될 때까지 기다리게 됩니다.
  + 코루틴 컨텍스트 상속 및 재정의
    * CoroutineScope는 자신만의 CoroutineContext를 가집니다. 이 컨텍스트에는 Job, CoroutineDispatcher, CoroutineName, CoroutineExceptionHandler 등이 포함될 수 있습니다.
    * 스코프 내에서 새로운 코루틴을 시작할 때 (launch, async 등), 자식 코루틴은 기본적으로 부모 스코프의 컨텍스트를 상속받습니다.
    * 필요에 따라 코루틴 빌더에 특정 컨텍스트 요소를 전달하여 부모의 컨텍스트를 재정의하거나 추가할 수 있습니다.
  + 코루틴 빌더의 수신 객체
    * launch, async와 같은 코루틴 빌더는 CoroutineScope의 확장 함수로 정의되어 있습니다. 따라서 이러한 빌더는 CoroutineScope 인스턴스 내에서 호출되어야 합니다.

- CoroutineScope 생성 방법
  + CoroutineScope() 팩토리 함수
    * CoroutineScope(coroutineContext: CoroutineContext): 주어진 CoroutineContext를 사용하여 새로운 스코프를 만듭니다. 일반적으로 Job()과 Dispatcher를 결합하여 컨텍스트를 구성합니다.
    ```kotlin
        val scope1 = CoroutineScope(Job() + Dispatchers.Main)
    ```
  + MainScope() 팩토리 함수
    * UI 관련 작업에 적합한 스코프를 만듭니다. 내부적으로 SupervisorJob()과 Dispatchers.Main을 사용합니다.
    ```kotlin
        val mainUiScope = MainScope() // SupervisorJob() + Dispatchers.Main
    ```
  + Android Jetpack의 생명주기 스코프
    * viewModelScope: ViewModel 클래스의 확장 프로퍼티로, ViewModel이 소멸될 때 자동으로 취소됩니다. 
    * lifecycleScope: LifecycleOwner (예: Activity, Fragment)의 확장 프로퍼티로, 해당 생명주기 객체가 소멸될 때 자동으로 취소됩니다.
  + GlobalScope
    * 애플리케이션 전체 생명주기를 가지는 전역 스코프입니다. 
    * GlobalScope에서 시작된 코루틴은 애플리케이션이 종료될 때까지 계속 실행될 수 있으며, 구조화된 동시성의 이점을 제공하지 않아 리소스 누수나 관리의 어려움을 초래할 수 있습니다. 
    * 특별한 경우가 아니면 사용을 지양해야 합니다.
  + coroutineScope { ... } 빌더
    * 이것은 새로운 CoroutineScope 인스턴스를 직접 만드는 것과는 약간 다릅니다. coroutineScope 빌더는 새로운 자식 스코프를 생성하고, 그 안의 모든 자식 코루틴이 완료될 때까지 현재 코루틴을 일시 중단합니다.
    * 주로 여러 코루틴을 병렬로 실행하고 모두 완료되기를 기다리거나, 특정 작업 그룹에 대한 예외 처리를 통합하려는 경우에 사용됩니다. 
    * 부모 코루틴의 컨텍스트를 상속받지만, Job은 새로 생성하여 자식 코루틴들을 관리합니다. 
    * 만약 이 내부 스코프에서 예외가 발생하면 해당 스코프와 그 자식들만 취소되고, 외부 스코프에는 영향을 미치지 않거나 예외를 전파할 수 있습니다.

- CoroutineScope 사용 시 주의사항
  + **적절한 생명주기에 맞는 스코프 사용**: Android에서는 viewModelScope나 lifecycleScope처럼 컴포넌트의 생명주기와 연동된 스코프를 사용하는 것이 중요합니다.
  + **스코프 취소**: 더 이상 필요하지 않은 스코프는 반드시 cancel()을 호출하여 관련된 모든 코루틴을 정리해야 합니다. (Jetpack 스코프는 자동으로 처리됨)
  + **Job과 SupervisorJob**: 일반 Job은 자식 코루틴 중 하나라도 실패하면 다른 모든 자식과 부모 Job까지 취소시킵니다. 
    반면 SupervisorJob은 자식 코루틴의 실패가 다른 자식이나 부모에게 영향을 주지 않도록 합니다. 
    UI 관련 작업이나 독립적인 여러 작업을 관리할 때 SupervisorJob이 유용할 수 있습니다. 
    MainScope()나 viewModelScope는 내부적으로 SupervisorJob을 사용합니다.

- Scope 정의
  + 다양한 컨텍스트에서 사용될 수 있지만, 일반적으로 변수, 함수, 클래스 등 식별자(identifier)가 유효하게 참조될 수 있는 코드상의 범위 또는 영역을 의미합니다.

- Scope의 주요역할
  + 이름 충돌 방지
    * 서로 다른 스코프에서는 동일한 이름의 변수나 함수를 선언할 수 있게 해줍니다.
    * 각 스코프는 독립적인 이름 공간을 가지므로, 한 스코프의 변수가 다른 스코프의 동일한 이름의 변수와 충돌하지 않습니다.
  + 가시성 및 접근 제어
    * 스코프는 특정 식별자가 코드의 어느 부분에서 보이고 접근 가능한지를 결정합니다.
    * 일반적으로 내부 스코프에서는 외부 스코프의 식별자에 접근할 수 있지만, 외부 스코프에서는 내부 스코프의 식별자에 직접 접근할 수 없습니다.
    * 이는 정보 은닉의 기초가 되며, 코드의 모듈성과 안정성을 높입니다.
  + 메모리 관리
    * 많은 프로그래밍 언어에서 변수의 생명주기는 해당 변수가 선언된 스코프와 밀접하게 관련됩니다.
    * 변수는 스코프에 진입할 때 메모리에 할당되고, 스코프를 벗어날 때 메모리에서 해제될 수 있습니다.
  + 코드의 구조화 및 가독성 향상
    * 스코프를 통해 코드를 논리적인 블록으로 나눌 수 있으며, 이는 코드의 구조를 명확하게 하고 이해하기 쉽게 만듭니다.
    * 변수나 함수의 영향 범위를 제한함으로써 코드의 특정 부분에 집중하여 분석하고 수정하기 용이해집니다.

- Job 정의
  + 실행 중인 코루틴의 핸들 또는 생명주기를 나타내는 객체입니다.
  + 코루틴이 시작될 때마다 해당 코루틴을 대표하는 Job인스턴스가 생성되며, 이를 통해 코루틴의 상태를 추적하고 제어할 수 있습니다.

- Job의 주요역할
  + 생명주기 관리
    * New (생성됨): Job이 생성되었지만 아직 활성화되지 않은 상태 (예: CoroutineStart.LAZY로 시작된 경우). 
    * Active (활성): 코루틴이 현재 실행 중이거나 일시 중단된 상태.
    * Completing (완료 중): Job의 모든 자식 코루틴이 완료되었지만, Job 자체의 완료 작업이 아직 진행 중인 상태.
    * Completed (완료됨): Job이 성공적으로 완료된 상태.
    * Cancelling (취소 중): Job에 취소 요청이 들어왔고, 취소 작업이 진행 중인 상태.
    * Cancelled (취소됨): Job이 취소된 상태.
  + 취소
    * cancel(cause: CancellationException? = null): 해당 Job과 이 Job에 속한 모든 자식 Job들에게 취소 요청을 보냅니다.
  + 대기
    * join(): Job이 완료될 때까지 (성공, 실패, 취소 모두 포함) 현재 코루틴을 일시 중단합니다. 이는 특정 코루틴의 작업이 끝날 때까지 기다려야 하는 경우에 유용합니다.
  + 부모-자식 관계 및 구조화된 동시성
    * Job은 계층 구조를 가질 수 있습니다. 한 코루틴(부모 Job) 내에서 다른 코루틴(자식 Job)을 시작하면, 자식 Job은 부모 Job의 하위 항목이 됩니다.
    * 부모 Job이 취소되면 모든 자식 Job도 재귀적으로 취소됩니다. 이는 구조화된 동시성의 핵심 원칙 중 하나로, 리소스 누수를 방지하고 코루틴의 생명주기를 예측 가능하게 만듭니다.
    * 부모 Job은 기본적으로 모든 자식 Job이 완료될 때까지 완료되지 않습니다.
  + 예외 처리
    * 자식 코루틴에서 처리되지 않은 예외가 발생하면, 해당 예외는 부모 Job으로 전파됩니다.
    * 기본적으로 부모 Job은 예외가 발생하면 자신과 다른 모든 자식 코루틴을 취소시킵니다.
    * CoroutineExceptionHandler를 CoroutineContext에 등록하여 이러한 예외를 처리할 수 있습니다.
  + 자식 Job 접근 및 관리
    * children: Sequence<Job>: 해당 Job의 직접적인 자식 Job들의 시퀀스를 반환합니다. 이를 통해 자식들의 상태를 확인하거나 개별적으로 제어할 수 있습니다. 
      
- | 특징       | 일반 스코프 (Scope)                          | Job (Kotlin Coroutines)                 | CoroutineScope (Kotlin Coroutin   | 
  |----------|-----------------------------------------|-----------------------------------------|-----------------------------------| 
  | 핵심 정의    | 식별자(변수, 함수 등)가 유효하게 참조될 수 있는 코드상의 영역/범위 | 개별 코루틴의 핸들 또는 생명주기를 나타내는 객체             | 새로운 코루틴을 시작하고 이들의 생명주기를 관리하는 범위   | 
  | 주요 관심사   | 식별자의 가시성, 생명주기, 이름 충돌 방지                | 단일 코루틴의 상태 추적, 실행 제어(취소, 대기), 부모-자식 관계  | 여러 코루틴의 그룹화, 일괄적인 생명주기 관리, 컨텍스트 제공 | 
  | 목적       | 코드 구조화, 메모리 관리, 접근 제어                   | 개별 코루틴의 제어 및 상태 확인                      | 구조화된 동시성, 리소스 누수 방지, 코루틴 실행 환경 제공 | 
  | 생성/형성 방식 | 언어의 문법적 구조 (블록 {} , 함수, 클래스 등)          | launch, async 등 코루틴 빌더 호출 시 반환          | CoroutineScope() 팩토리 함수, viewModelScope, lifecycleScope, coroutineScope { } 빌더 등 |
  | "취소" 개념  | 직접적인 "취소" 개념 없음 (스코프를 벗어나면 식별자 해제)      | cancel(): 해당 Job 및 그 자식 Job들에게 취소 요청    | cancel(): 스코프 내 모든 코루틴(Job) 일괄 취소 | 
  | 생명주기     | 식별자가 선언된 영역의 실행 흐름에 따름                  | New, Active, Completing, Completed, Cancelling, Cancelled | 스코프 자체의 생명주기 (예: ViewModel 소멸 시 viewModelScope 취소) | 
  | 주요 기능/연산 | - (언어 규칙에 의해 정의됨)                       | join(), cancel(), isActive, isCompleted, isCancelled | launch(), async(), cancel() (스코프의 Job을 통해) | 
  | 비유       | 도구나 재료를 사용할 수 있는 "작업 공간" (방, 책상)        | 특정 작업을 수행하는 "개별 작업자" 또는 "작업 지시서" | 여러 작업자(코루틴)를 관리하는 "프로젝트 관리 사무실"   |

- SupervisorJob vs 일반 Job 예제
  + ```kotlin
    val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default)

    scope.launch {
      launch {
          throw RuntimeException("이 코루틴만 죽고")
      }
      launch {
          delay(1000)
          println("이 코루틴은 살아 있음") // 실행됨
      }
    }
    ```
  + 일반 Job()에서는 첫 번째 코루틴 예외가 발생하면 두 번째 코루틴도 취소됨
  + | 항목            | `coroutineScope {}`  | `supervisorScope {}` |
    | ------------- | -------------------- | -------------------- |
    | 예외 전파         | 자식 중 하나 실패 → 전부 취소   | 자식 중 하나 실패해도 나머지 생존  |
    | 사용 목적         | 실패를 함께 처리하는 단위 작업 그룹 | 실패와 무관한 독립 작업 그룹 구성  |
    | Dispatcher 상속 | 상속함                  | 상속함                  |


---
