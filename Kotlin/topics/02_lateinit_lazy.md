### lateinit & lazy
- 정의
  + 코틀린에서 non-null 타입 프로퍼티를 나중에 초기화할 수 있도록 지원하는 두 가지 메커니즘
  + `lateinit` & `lazy` 2가지 방식 제공

- lateinit
  + 선언
    ```kotlin
    lateinit var myProperty : String
    ```
  + 목적
    * 선언 시 즉시 초기화할 수 없는 non-null var 프로퍼티를 나중에 명확히 초기화할 필요 있을 때 사용
  + 제약사항
    * var 프로퍼티에만 사용할 수 있습니다.
    * Nullable 타입에는 사용할 수 없습니다.
    * 기본 자료형(Int, Boolean 등) 사용할 수 없음 (박싱된 타입 가능)\
    * 커스텀 getter/setter를 불가
    * 클래스 내부에서만 사용 가능 (top-levl 사용 불가)
  + 주의점
    * lateinit 프로퍼티는 사용하기 전에 반드시 초기화 필요
    * ::myProperty.isInitialized 로 체크 가능

- lazy
  + 선언
    ```kotlin
    val myLazyProperty : String by lazy { 
      println("초기화 수행") 
      "Hello Lazy" 
    }
    ```
  + 목적
    * 초기화 비용이 큰 객체를 최초 사용 시점까지 늦추고 싶을 때
    * 객체가 한 번만 초기화되고 이후 절대 변경되지 않아야 할 때
  + 제약사항
    * val 프로퍼티에만 사용 가능
    * 초기화 블록은 단 한 번만 실행
  + 특징
    * 필요에 따라 `PUBLICATION`, `NON`E 등의 스레드 안전 모드로 변경 가능
    * 초기화 블록은 최초 접근 시에만 실행
    * 기본적으로 LazyThreadSafetyMode.SYNCHRONIZED로 동작하기에 멀티스레드 환경에서도 안전하게 한 번만 초기화 됨

- 비교
  + | 조건                            | 추천 방식      | 설명                          |
    | ----------------------------- | ---------- | --------------------------- |
    | 생성자에서 초기화 불가능, 나중에 수동으로 넣어야 함 | `lateinit` | 주로 의존성 주입(DI), 뷰 바인딩 등에서 사용 |
    | 초기화 비용이 크고, 최초 접근 시 자동 초기화    | `lazy`     | 싱글톤, 캐시, 설정값 로딩 등에서 적합      |
    | 값이 변경될 수 있음                   | `lateinit` | `var` 기반                    |
    | 값이 한 번만 설정되고, 불변              | `lazy`     | `val` 기반                    |

- 면접 관련 질문
  + `lateinit`과 `lazy`는 각각 어떤 상황에 쓰이나요?
    * lateinit은 초기화 시점을 개발자가 직접 제어해야 하는 경우 사용하며 주로 DI, Android view 바인딩 등에서 유용
    * lazy는 초기화가 자동으로 이루어지고 한 번만 실행되며, 초기화 비용이 큰 객체나 싱글톤 객체에 적합
  + `lateinit` 사용 시 주의할 점은?
    * 초기화 전에 접근하면 UninitializedPropertyAccessException 예외가 발생
    * val이나 기본 타입에는 사용할 수 없고, 반드시 non-null 참조 타입 + var 사용
  + `lazy`의 스레드 안전성은 어떻게 보장되나요?
    * 기본적으로 lazy는 LazyThreadSafetyMode.SYNCHRONIZED 모드로 작동함. 이는 여러 스레드에서 동시에 접근하더라도 초기화 블록이 한 번만 실행되도록 보장함. 필요에 따라 다른 모드로 변경할 수 있음 (PUBLICATION, NONE)


---
