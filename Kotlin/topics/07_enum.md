### enum 열거형 클래스
- 개념 및 정의
  + 미리 정의된 고정된 상수 집합을 표현할 때 사용하는 열거형 클래스
  + 방향, 상태 코드, 명령어 등 한정된 선택지가 있을  때 적합
  + 각 열거 상수는 싱글턴 객체이며 필요 시 생성자와 메서드 정의 가능

- 기본 사용법
  + ```kotlin
    enum class Direction {
        NORTH, SOUTH, EAST, WEST
    }
    
    val dir = Direction.NORTH
    print(dir.name) // "NORTH"
    print(dir.ordinal) // 0
    ```
  + `name`: 문자열
  + `ordinal`: 순서(0부터 시작)
    
- 예제
  + 생성자와 프로퍼티 사용
    * ```kotlin
      enum class HttpsStatus(val code: Int, val desc: String) {
        OK(200, "Success"),
        NOT_FOUND(404, "Not Found"),
        SERVER_ERROR(500, "Internal Server Error")
      }
      
      val status = HttpsStatus.NOT_FOUND
      println(status.code) // 404
      println(status.desc) // Not Found
      ```
  + enum 별로 다른 동작 구현 (함수 정의)
    * ```kotlin
      enum class Operation {
        PLUS {
           override fun apply(a: Int, b: Int) = a + b
        },
        MINUS {
           override fun apply(a: Int, b: Int) = a - b
        };

        abstract fun apply(a: Int, b: Int): Int
      }
      
      Operation.PLUS.apply(5,3).also { println(it) } // 8
      Operation.MINUS.apply(5,3).also { println(it) } // 2
      ```
    * 각 enum 상수가 자신만의 메서드 구현을 가질 수 있음
    * 추상 메서드를 정의하고 override 가능
  + `when` 과 함께 쓰임
    * ```kotlin
      fun handleDirection(dir: Direction) = when(dir) {
        Direction.NORTH -> "Going up"
        Direction.SOUTH -> "Going down"
        Direction.EAST  -> "Going right"
        Direction.WEST  -> "Going left"
      }
      ```
    * `when`과 함께 사용 시 모든 enum을 다 다루면 `else` 생략 가능
    * enum 타입에 대해 `exhaustive`(가능한 모든 경우를 빠짐없이 다룸) 체크 수행
  + `values()` / `valueOf()`
    * ```kotlin
      enum class Direction {
        NORTH, SOUTH, EAST, WEST
      }
      
      Direction.values().forEach { println(it) } // 전체 enum 출력
      Direction.valueOf("EAST").also { println(it) } // 문자열 -> enum
      ```
  
- enum vs sealed class
  + | 항목        | enum class           | sealed class                  |
    | --------- | -------------------- | ----------------------------- |
    | 목적        | 고정된 상수 집합            | 계층적인 타입 제한                    |
    | 런타임 확장    | ❌ 불가능 (상수 고정)        | ✅ 하위 클래스 자유롭게 정의 가능           |
    | 인스턴스 속성   | 제한적 (상수당 고정 생성자)     | 유연하게 정의 가능                    |
    | 상태 표현     | 간단한 상태 (예: 방향, 상태코드) | 복잡한 구조적 상태 표현에 유리             |
    | `when` 사용 | 자동 exhaustive 체크     | exhaustive 하려면 else 필요할 수도 있음 |

- 면접 관련 질문
  + enum class는 어떤 상황에서 적합한가?
    * enum은 가능한 값이 정해진 범위 내에 있고, 각 값이 하나의 고유한 상태나 명령을 의미할 때 적합
    * 예: 방향(Direction), HTTP 상태 코드, 정렬 기준 등
  + enum과 sealed class의 근본적인 차이?
    * enum은 고정된 상수 집합만 정의할 수 있고, 각 상수는 동일한 타입
    * 반면 sealed class는 서로 다른 구조나 속성을 가진 하위 타입을 정의할 수 있어 더 유연
    * 복잡한 상태 분기나 타입 분기에는 sealed class가 더 적합
  + enum 클래스에서 동작을 다르게 구현할 수 있는 이유는?
    * Kotlin의 enum 클래스는 각 상수를 객체처럼 취급하므로, 각 상수가 자체적으로 함수를 오버라이드할 수 있음
    * 이를 통해 Operation.PLUS, Operation.MINUS처럼 각 상수마다 다른 동작을 지정할 수 있음


---
