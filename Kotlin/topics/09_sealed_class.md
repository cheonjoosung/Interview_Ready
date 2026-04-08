### sealed class
- 개념 및 정의
  + 하위 클래스의 정의를 같은 파일로 제한한 제한적 상속 클래스
  + 컴파일러가 하위 타입을 모두 알 수 있기 때문에, `when` 문에서 안전하고 완전한 분기가 가능
  + `enum class`보다 더 유연하게 각 하위 타입에 **서로 다른 속성과 동작**을 부여할 수 있어 복잡한 상태 모델링에 적합

- 기본 문법
  + ```kotlin
    sealed class Result

    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
    object Loading : Result()

    // when 과 함께 사용
    fun handle(result: Result) {
      when (result) {
          is Success -> println("성공: ${result.data}")
          is Error   -> println("에러: ${result.message}")
          is Loading -> println("로딩 중")
      }
    }
    ```
  + `when` 블록에서 `else` 생략 가능 - 컴파일러가 모든 하위 타입을 알고 있어서 exhaustive check 가능

- 특징 요약
  + | 항목                | 설명                                                 |
    | ----------------- | -------------------------------------------------- |
    | **상속 제한**         | 하위 클래스는 **같은 파일 내에서만 정의 가능** (컴파일러가 전체 타입 인지)      |
    | **직접 인스턴스화 불가**   | 추상 클래스처럼 동작. `sealed` 자체는 객체 생성 불가                 |
    | **다양한 하위 클래스 가능** | `data class`, `object`, `class` 등 다양한 형태로 하위 정의 가능 |
    | **when 최적화**      | `when` 사용 시 `else` 생략 가능 (모든 타입 처리 확인됨)            |
    | **enum보다 유연함**    | enum은 상수만, sealed는 **속성/동작 커스터마이징** 가능             |

- enum class vs sealed class
  + | 비교 항목     | `enum class`            | `sealed class`                        |
    | --------- | ----------------------- | ------------------------------------- |
    | 사용 목적     | 제한된 **고정 상수 집합**        | **타입 계층 구조**, 다양한 상태 표현에 적합           |
    | 런타임 확장    | ❌ 불가능 (컴파일 시 고정)        | ✅ 가능 (같은 파일 내에서 자유롭게 정의 가능)           |
    | 속성 커스터마이징 | 제한적 (모든 상수는 동일 구조)      | 각 하위 클래스마다 **속성, 메서드 다르게 정의 가능**      |
    | when 처리   | 컴파일러가 자동 exhaustive 확인  | 동일 (else 생략 가능)                       |
    | 실무 활용     | 간단한 상수 분기 (정렬 기준, 방향 등) | 복잡한 상태 모델링 (API 응답, UI 상태, Command 등) |
 
- 실무 사용 예시
  + API 응답 모델
    * ```kotlin
      sealed class ApiResult<out T> {
        data class Success<T>(val data: T): ApiResult<T>()
        data class Failure(val error: Throwable): ApiResult<Nothing>()
        object Loading: ApiResult<Nothing>()
      }
      ``` 
  + UI 상태 표현 
    * ```kotlin
      sealed class LoginState {
        object LoggedOut : LoginState()
        data class LoggedIn(val user: User) : LoginState()
        object Loading : LoginState()
      }
      ``` 

- 면접 관련 질문
  + `sealed class`는 왜 `enum class`보다 더 유연한가요?
    * enum은 고정된 값만 표현할 수 있고, 각 상수가 동일한 구조를 가져야 합니다.
    * 반면, sealed class는 하위 클래스마다 서로 다른 속성과 동작을 정의할 수 있어 복잡한 상태나 데이터를 표현하는 데 더 적합합니다.
  + sealed class를 사용했을 때 when에서 else 없이도 안전한 이유는?
    * sealed class는 같은 파일 내에 정의된 모든 하위 타입을 컴파일러가 알 수 있어서, when 문이 exhaustive check를 수행할 수 있습니다.
    * 따라서 모든 하위 타입을 다 처리하면 else 없이도 컴파일 오류가 발생하지 않습니다.
  + sealed class와 abstract class의 차이는?
    * abstract class는 상속에 제한이 없고, 어디서든 하위 클래스 정의가 가능합니다.
    * 반면 sealed class는 하위 타입을 같은 파일에만 정의할 수 있어, 컴파일러가 전체 타입을 추론할 수 있다는 점이 다릅니다.
    * 이 덕분에 when 문에서 안전하고 명확한 분기 처리가 가능합니다.


---
