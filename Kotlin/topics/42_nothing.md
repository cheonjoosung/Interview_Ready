### Nothing 타입
- 개념 및 정의
  + `Nothing`은 **"값이 절대 존재하지 않음"**을 명시적으로 표현하는 특수 타입입니다.
  + 주로 정상적인 반환이 절대 없는 함수의 반환 타입으로 사용합니다.
  + 즉, 이 함수 이후의 코드는 절대 실행되지 않음을 보장합니다.
  
- 왜 필요한가?
  + | 목적        | 이유                                    |
    | --------- | ------------------------------------- |
    | 코드 가독성    | "이 함수는 절대 끝까지 반환하지 않는다"를 타입으로 명확하게 표시 |
    | 타입 안전성 강화 | 타입 추론 시 `Nothing` 덕분에 안전하게 컴파일 가능     |
    | 예외 상황 표현  | 예외 발생, 무한 루프 등 프로그램 흐름이 종료되는 구문에 사용   |

- 주요 특징
  + | 특징               | 설명                                          |
    | ---------------- | ------------------------------------------- |
    | **모든 타입의 하위 타입** | `String`, `List<Int>` 등 어떤 타입 자리에도 들어갈 수 있음 |
    | **값 없음**         | 인스턴스를 가질 수 없음                               |
    | **종료 보장**        | `throw`, `return` 등 코드 흐름을 끝내는 구문에서 사용      |
    | **주요 용도**        | 예외 발생 함수, 무한 루프, 실패 케이스 처리                  |

- 예시
  + 예외 던지는 함수에 사용
    * ```kotlin
      fun fail(message: String): Nothing {
        throw IllegalArgumentException(message)
      }
      ```
    * `Nothing` 반환 타입은 "이 함수는 절대 값을 반환하지 않는다"는 것을 의미합니다.
  + 엘비스 연산자(`?:`)와 결합
    * ```kotlin
      val name:String? = null
      val result = name ?: fail("name cannot be null")
      // result String 으로 타입 추론
      ```
    * `fail()`이 `Nothing`을 반환하므로, `result`는 `String` 타입으로 유지됩니다.
  + when exhaustive
    * ```kotlin
      sealed class Result
      object Success : Result()
      object Failure : Result()
      
      fun handle(result: Result): String = when(result) {
        is Success -> "OK"
        is Failure -> "Fail"
        else -> error("Unknown result") // error() 의 반환 Nothing
      }
      ```
    * `Nothing` 덕분에 `else` 분기에서 반환 타입을 깨뜨리지 않음

- 대표적으로 `Nothing`을 반환하는 표준 함수
  + | 함수 이름            | 설명                       |
    | ---------------- | ------------------------ |
    | `error(message)` | 예외 발생 후 프로그램 종료          |
    | `fail(message)`  | 커스텀 예외 발생                |
    | `TODO()`         | 아직 구현되지 않은 부분 표시 후 예외 발생 |
    | `throw` 표현식      | 예외 던짐                    |
    | `while(true)` 형태 | 무한 루프                    |
 
- 면접 관련 질문
  + Nothing 타입은 어떤 상황에서 사용하나요?
    * 정상적으로 종료되지 않는 함수(예외 발생, 무한 루프)나 실패 케이스를 명확히 표현하기 위해 사용합니다.
    * 타입 시스템에 "여기서 절대 값이 반환되지 않는다"를 알릴 수 있습니다.
  + Nothing은 어떤 타입의 하위 타입인가요?
    * 모든 타입의 하위 타입(subtype)입니다.
    * 예: String, Int, List<Double> 어디든 들어갈 수 있습니다.
  + 실무에서 Nothing 타입이 쓰이는 구체적인 예시는 무엇인가요?
    * 예외를 던지는 함수(fail, error, require)
    * ?: 엘비스 연산자와 조합해 타입 안정성 유지
    * when 표현식에서 모든 분기 처리가 되었음을 보장
    * 테스트 코드에서 TODO() 사용
  + Nothing과 Unit의 차이는 무엇인가요?
    * Unit: "값은 없지만 정상적으로 종료됨"
    * Nothing: "정상적으로 종료되지 않음 (반환 불가능)"
    * 즉, Unit 함수 뒤의 코드는 실행되지만, Nothing 함수 뒤의 코드는 절대 실행되지 않습니다.


---
