### Safe Call / Elvis 연산자
- 개념/정의
  + NPE(Null Pointer Exception) 방지하기 위해 nullable 타입과 null-safe 연산자 지원
  + `?.` safe call 과 `?:` Elvis 연산자 2가지를 주로 사용

- Safe Call 연산자 (?.)
  + 객체가 null 아닐 경우에만 메서드나 프로퍼티에 접근
  + 객체가 null 이면 연산을 건너뛰고 결과로 null 반환
  + NPE(Null Pointer Exception) 방지

- 사용법
  + ```kotlin
    val result = someObject?.someMethod()

    data class User(val name: String?)

    val user: User? = null
    val nameLength = user?.name?.length
    println(nameLength) // 출력: null
    ```
  
- Elvis 연산자 (?:)
  + 좌변이 null 일 경우 우변 값 사용
  + 기본값, 예외처리, 빠른 리턴 등에 활용
  
- 사용법
  + ```kotlin
    val value = nullableValue ?: defaultValue

    // Safe Call 연산자 + Elvis 연산자 함께 쓰는 예
    val user: User? = null
    val nameLength = user?.name?.length ?: 0
    println(nameLength) // 출력: 0
  
    // Android 에서 자주 쓰는 패턴
    // null이면 return 처리
    val token = prefs?.getString("token", null) ?: return

    // 예외 처리
    val user = findUser() ?: throw IllegalStateException("User not found")
    ```

- 요약
  + | 연산자  | 역할                 | 반환 결과               |
    | ---- | ------------------ | ------------------- |
    | `?.` | null이면 호출 or 접근 생략 | `null`              |
    | `?:` | null이면 우측 기본값 사용   | non-null 값 또는 throw |

- 면접 관련 질문
  + Safe Call 연산자와 Null 체크의 차이는?
    * Safe Call 연산자(?.)는 null 여부를 명시적 조건 없이 간결하게 처리할 수 있어 가독성이 높고, 체이닝에도 적합합니다. (예: a?.b?.c?.d) 기존의 if (a != null) a.b 구조보다 훨씬 직관적입니다.   
  + Elvis 연산자와 기본값 할당의 차이는?
    * Elvis 연산자(?:)는 null일 경우에만 우측 값을 사용하므로, 기본값이 조건적으로만 적용 된다. 단순한 기본값 할당이 아니라 null 체크와 동시에 fallback을 처리하는 연산자임
  + Elvis 연산자로 예외 처리가 가능한 이유는? 
    * ?: 우측은 표현식이므로, 값을 반환하거나 예외를 던지는 것도 허용되기에 이를 통해 null이 발생할 경우 한 줄로 throw 처리가 가능해지고, 코드가 더 간결하면서도 의도를 분명하게 표현할 수 있음


---


## var vs val
- 개념
  + var 변경 가능한 변수 가변(mutable)
  + val 변경 불가능한 변수 불변(immutable)

- 공통점
  + | 항목                     | 내용                                    |
    | ---------------------- | ------------------------------------- |
    | **둘 다 변수 선언에 사용**      | Kotlin에서 변수를 선언할 때 `var` 또는 `val`을 사용 |
    | **타입 추론 가능**           | `var name = "John"`처럼 타입 명시 없이 선언 가능  |
    | **nullable 허용 가능** | 둘 다 `?`로 nullable 여부 설정 가능 |
    | **스코프 규칙 동일** | 블록 스코프, 지역 변수, 전역 변수 등의 범위 규칙은 동일함 |

- 차이점
  + | 항목             | `var`          | `val`  |
    | -------------- | ---------------------------------- | ------------------------------------------------------------------ |
    | **가변성**        | 값을 변경할 수 있음                 | 최초 할당 이후 값 변경 불가                                                   |
    | **재할당**        | 가능                   | 불가능                          |
    | **사용 예시**      | 루프 인덱스, 상태 변화 변수 | 상수, 변경되지 않는 참조값        |
    | **컴파일러 처리**    | 매번 수정 가능성 고려  | 불변성 보장 → 최적화 유리 |
    
- 예시
  + ```kotlin
    // var: 값 변경 가능
    var name = "Alice"
    name = "Bob" // ✅ 가능

    // val: 값 변경 불가능
    val age = 30
    // age = 31 // ❌ 컴파일 오류

    // val로 선언된 불편 컬렉션의 내부는 변경 가능
    val list = listOf(1,2,3)
    // list 에는 객체를 변경할 수 있는 메소드가 제공되지 않음
    list = listOf(4,5,6) // ❌ 재할당 불가능
    
    // val로 선언된 가변 컬렉션의 내부는 변경 가능
    val numbers = mutableListOf(1, 2, 3)
    numbers.add(4) // ✅ 가능
    // numbers = mutableListOf(5, 6) // ❌ 재할당 불가능

    class MyViewModel : ViewModel() {
      private val _uiState = MutableLiveData<UiState>()
      val uiState: LiveData<UiState> = _uiState
    }
    ```
  + `val`은 "참조를 바꿀 수 없다"는 의미이지, 참조 대상의 내부가 불변이라는 의미는 아님. (읽기 전용과 불변 객체는 다른 뜻)
  + `val`은 외부에 읽기 전용으로 상태를 노출하기 위해 사용함. 내부에서는 `_uiState`를 통해 실제 값 변경
  
- 결론
  + | 전략                | 이유                                |
    | ----------------- | --------------------------------- |
    | 기본은 `val` 사용      | 코드 안정성 향상, 예측 가능성 높음              |
    | `var`은 필요한 경우만 사용 | 상태 변화가 필요하고, 제어 가능한 범위 내에서만 사용할 것 |
    | `val` + 가변 컬렉션    | 구조는 고정하고 내부 상태만 바꾸는 경우에 적합        |
    | `val` 많을수록 장점     | 스레드 안정성, 버그 감소, 가독성 및 유지보수 유리     |

- 면접 관련 질문
  + val과 var의 실질적인 차이는 메모리나 성능에도 영향이 있나요?
    * val은 참조 변경이 불가능하므로 컴파일러가 더 많은 최적화 기회를 가질 수 있습니다. 
    * var는 값이 언제든 바뀔 수 있으므로 더 조심해서 처리해야 하고, 멀티스레드 환경에선 동기화도 고려됩니다.
  + val로 선언한 컬렉션도 내부 요소는 변경 가능한 이유는?
    * val은 변수가 참조하는 객체 자체를 바꿀 수 없다는 뜻이지, 그 객체의 내부 구조까지 불변이라는 의미는 아닙니다. 
    * 예: val list = mutableListOf(...)는 참조는 고정되지만 요소는 변경 가능합니다.
  + 어떤 상황에서 var를 꼭 써야 하나요?
    * 값이 계속 변하는 상태값을 관리할 때는 var가 필요합니다. 
    * 예: 루프 인덱스, 스크롤 위치 추적, 카운터 등 다만 이런 경우도 가능한 한 범위를 좁히고, 외부에서 변경 불가능하게 캡슐화해야 안전합니다.


---
