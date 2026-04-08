### 상속 제어 (open, final, abstract)
- 정의
  + 코틀린에서 상속 제어 키워드는 클래스나 함수, 프로퍼티가 상속/재정의 가능한지 명확하게 제어하는 기능
  + Java 와 기본값이 다르며 코틀린의 안정성과 설계 철학을 보여주는 부분

- 핵심 개념
  + | 키워드        | 의미                             | 기본값 여부            |
    | ---------- | ------------------------------ | ----------------- |
    | `final`    | **상속 불가**, 오버라이드 금지            | ✅ Kotlin의 기본값     |
    | `open`     | **상속 가능**, 오버라이드 허용            | ❌ 명시적으로 선언해야 함    |
    | `abstract` | 추상 멤버 또는 클래스, **무조건 오버라이드 필요** | ❌ 추상 클래스에서만 사용 가능 |

- 상세 설명
  + final
    * 코틀린에서는 별도 지정이 없으면 `final`
    * 오버라이드/상속 하려면 명시적으로 `open` or `abstract` 로 풀어야 함
    * ```kotlin
      class Animal {
         fun sound() { println("Animal sound") } // final이 기본
      }

      // class Dog : Animal() { ... } // ❌ 컴파일 에러 (speak 오버라이드 불가)
      ```
  + open
    * 해당 클래스 or 멤버를 상속/재정의 가능하게 만듦
    * 용도: 다형성, 테스트용 mock 클래스 등에 사용
    * ```kotlin
      open class Animal {
         open fun sound() { println("Animal sound") }
      }

      class Dog : Animal() {
         override fun sound() { println("Bark!") }
      }
      ```
  + abstract
    * 클래스 또는 멤버가 추상적이며 하위 클래스에서 반드시 구현해야 함
    * 인스턴스화 불가, 일부만 구현해도 가능 (abstract + concrete), 메서드에도 abstract 가능
    * ```kotlin
      abstract class Animal {
        abstract fun speak()
      }

      class Cat : Animal() {
        override fun speak() = println("Meow")
      }
      ```

- 실전 주의사항
  + `data class`는 `final` 고정 (→ 상속 불가)
  + `abstract`에는 반드시 open이 내포되어 있음 (굳이 `open abstract` 쓸 필요 없음)
  + `open` 없이 오버라이드 시도하면 컴파일 에러
  + 실무에서는 의도하지 않은 상속을 막기 위해 대부분 `final`을 유지하고 필요한 경우만 `open` 지정 

- 면접 관련 질문
  + Kotlin은 왜 모든 클래스가 기본적으로 final인가요?
    * 의도하지 않은 상속이나 오버라이드를 방지하고 안정성과 예측 가능성을 높이기 위해서입니다.
    * Java는 반대로 기본적으로 상속 가능하지만, Kotlin은 명시적 설계를 권장합니다.
  + abstract 클래스와 interface의 차이는?
    * abstract 클래스는 상태(필드) + 구현 가능 / 단일 상속
    * interface는 필드 없음(기본은), 구현 가능 / 다중 구현 가능
  + data class는 왜 상속이 불가능한가요?
    * data class는 equals(), hashCode(), copy() 등을 자동 생성하며, 상속 시 이 동작이 깨질 수 있기 때문입니다.
    * 명확한 데이터 표현용으로 final이 기본입니다.


---
