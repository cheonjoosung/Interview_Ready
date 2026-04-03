### Unit
- 개념
  + Kotlin에서 Unit은 값을 반환하지 않는 함수의 반환 타입을 명시할 때 사용되는 타입입니다.
  + 자바의 void와 유사한 역할을 하지만, Unit은 실제 존재하는 객체입니다.

- Unit의 주요특징
  + | 특징              | 설명                                           |
    | --------------- | -------------------------------------------- |
    | 반환 없는 함수의 반환 타입 | 함수가 명시적으로 값을 반환하지 않으면 `Unit` 타입              |
    | 실제 객체           | `Unit`은 `object`로 정의된 싱글톤 (Java의 `void`와 다름) |
    | 생략 가능           | `: Unit`은 생략 가능 – 컴파일러가 자동 추론                |
    | 제네릭에 사용 가능      | `Unit`은 타입이므로 제네릭 함수의 타입 파라미터로 사용 가능         |
    | 명시적 `return` 지원 | `return` 또는 `return Unit` 모두 가능, 관용적으로는 생략   |

- 사용법
  + 기본
    * ```kotlin
      fun printMessage(message: String): Unit {
        println(message)
      }

      fun greet() { // Unit 생략, 컴파일러가 추론
        println("Hello!")
      }
      ``` 
  + Unit은 실제 싱글톤 객체다
    * Kotlin의 `Unit`은 다음처럼 선언된 실제 객체입니다
    * ```kotlin
      object Unit
      ```
    * 즉, `Unit` 타입의 값은 단 하나 – `Unit` 객체 그 자체입니다.
    * Java의 `void`는 그 자체로 타입이 아니므로, 제네릭에 쓸 수 없고, 객체로 다룰 수도 없습니다.  
  + 제네릭에서의 활용
    * ```kotlin
      fun <T> doSomething(block: () -> T) {
        val result = block()
        println("Done")
      }

      fun main() {
        doSomething<Unit> {
          println("작업 실행 중")
        }
      }
      ```  
    * Unit은 void와 달리 제네릭 함수의 타입 인자로 사용 가능
    * 함수형 프로그래밍 스타일에서 특히 유용
  + 명시적인 `return Unit`?
    * ```kotlin
      fun checkAge(age: Int): Unit {
        if (age < 18) {
          println("Minor")
          return // 또는 return Unit
        }
        println("Adult")
      }
      ``` 
    * `return`만 사용해도 Unit 반환과 동일
    * `return Unit`은 가능하지만 권장되지 않음

- void vs Unit 비교
  + | 항목            | Kotlin `Unit`      | Java `void`    |
    | ------------- | ------------------ | -------------- |
    | 타입인가?         | ✅ 실제 타입 (`object`) | ❌ 타입이 아님 (키워드) |
    | 값이 존재하는가?     | ✅ `Unit` 싱글톤 객체    | ❌ 값 자체가 없음     |
    | 제네릭에 사용 가능한가? | ✅ 사용 가능            | ❌ 불가능          |
    | 반환 타입 생략 가능?  | ✅ 생략 시 자동 추론       | ❌ 명시적 void 필요  |
  

- 왜 void 대신 Unit을 사용할까?
  + 일관된 타입 시스템: Kotlin은 모든 것을 객체로 취급하려는 경향이 있습니다. Unit을 객체로 만듦으로써 "값이 없음"조차도 타입 시스템 내에서 일관되게 다룰 수 있습니다.
  + 제네릭 활용 가: 위에서 언급했듯이, 제네릭 타입 파라미터로 Unit을 사용할 수 있게 되어 함수형 프로그래밍 패턴 등에서 유용합니다.
  + 함수형 프로그래밍 적합: 함수형 프로그래밍에서는 모든 함수가 값을 반환하는 것을 선호합니다. Unit은 "의미 있는 값은 없지만, 작업은 완료되었음"을 나타내는 일종의 신호로 볼 수 있습니다. 

- 면접 관련 질문
  + Kotlin의 `Unit`과 Java의 `void`는 어떤 차이가 있나요?
    * `Unit`은 실제 객체이며 타입이 존재합니다.
    * `void`는 단지 "값 없음"을 나타내는 키워드입니다.
    * Kotlin에서는 `Unit`을 통해 값이 없더라도 타입 시스템 내에서 일관되게 처리할 수 있습니다.
    * 또한 `Unit`은 제네릭에 사용할 수 있는 반면, `void`는 불가능합니다.
  + 반환 타입이 `Unit`인 경우, 함수 선언에서 생략해도 괜찮나요?
    * Kotlin 컴파일러가 자동으로 `Unit`을 추론합니다.
    * `fun myFunc()`처럼 선언하면, 내부적으로는 `fun myFunc(): Unit`과 동일하게 처리됩니다.
  + `Unit`을 반환하는 함수에서 `return`은 꼭 써야 하나요?
    * 아니요. `return`은 생략 가능합니다.
    * 단, 함수 중간에서 빠져나가고 싶다면 `return`을 사용할 수 있습니다.
    * `return Unit`처럼 명시하는 것도 가능하지만 관용적으로는 단순히 `return`만 사용합니다. 


---
