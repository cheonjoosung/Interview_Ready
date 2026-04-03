### 타입 캐스팅 
- 정의
  + 하나의 객체를 다른 타입으로 변환하는 작업
  + 코틀린은 안전성과 명확성 중심으로 캐스팅을 설계하여 `ClassCastException`과 같은 런타입 에러 최소화 함

- 타입 캐스팅 방식 비교
  + | 분류            | 문법           | 설명                                         |
    |-----------------|----------------|----------------------------------------------|
    | **강제 캐스팅**   | `as`           | 타입이 맞지 않으면 **`ClassCastException` 발생**         |
    | **안전한 캐스팅** | `as?`          | 캐스팅 실패 시 `null` 반환 → **NPE 방지**            |
    | **스마트 캐스트** | `is` / `!is`   | 타입 체크 후 자동 캐스팅 (조건문 내에서만 유효)         |

- 사용 예시
  + `as` 강제 캐스팅
    * ```kotlin
      val any: Any = "Hello"
      val str: String = any as String  // ✅ 성공

      val num: Any = 123
      val str = num as String  // ❌ ClassCastException
      ```
    * 명시적으로 타입 변환하며 실패시 예외 발생
    * 반드시 타입이 정확히 일치하는지 확신이 있는 경우에만 사여ㅛㅇ  
  + `as?` 안전한 캐스팅
    * ```kotlin
      val any: Any = "Kotlin"
      val str: String? = any as? String  // ✅ "Kotlin"

      val num: Any = 123
      val str: String? = num as? String  // ✅ null 반환 (예외 없음)
      ```
    * 실패해도 예외 대신 null 반환
    * 코틀린의 null-safety 와 잘 어울리며, `?.`, `?:` 함께 스이면 더 안전  
  + `is` 타입 검사 (스마트 캐스트)
    * ```kotlin
      fun printLength(obj: Any) {
        if (obj is String) {
           println(obj.length)  // obj가 자동으로 String으로 캐스팅됨
        }
      }

      fun handleInput(input: Any) {
        when (input) {
            is String -> println("String length: ${input.length}")
            is Int -> println("Double: ${input * 2}")
            else -> println("Unknown type")
        }
      }
      ```
    * `is`로 체크하면 블록 안에서 자동으로 스마트 캐스팅됨
    * 해당 블록 안에서는 별도 캐스팅 없이 해당 타입처럼 사용 가능
    * `when`은 `is`를 내장적으로 사용하여 스마트 캐스트를 자동 처리함

- 면접 관련 질문
  + Kotlin의 as?가 Java의 캐스팅과 다른 점은?
    * Java에서는 잘못된 캐스팅 시 무조건 ClassCastException이 발생하지만, Kotlin의 as?는 실패할 경우 예외 대신 null을 반환하여 NPE나 런타임 크래시 위험을 줄여준다.
    * 즉, as?는 안전한 캐스팅 방식이다.
  + 스마트 캐스팅이 작동하지 않는 경우도 있나요?
    * 스마트 캐스트는 val이면서 non-null일 경우에만 안정적으로 작동한다.
    * 만약 var이거나, **nullable 타입(val str: String?)**이면 중간에 값이 바뀔 가능성이 있다고 판단되어 자동 캐스팅이 되지 않는다.
  + is와 as는 어떤 상황에서 어떻게 다르게 쓰이나요?
    * is는 타입을 체크하고, 체크된 블록 안에서 자동으로 스마트 캐스팅을 해준다.
    * as는 타입을 강제로 변환하며, 실패 시 예외가 발생한다.
    * 따라서 안정성이 중요할 땐 is + 조건문, 또는 as?를 활용하는 것이 좋다. 


---
