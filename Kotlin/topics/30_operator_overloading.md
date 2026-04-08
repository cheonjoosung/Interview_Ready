### Operator Overloading
- 개념 및 정의
  + operator 키워드를 사용해, 클래스의 메서드에 특정 연산자 기능을 부여할 수 있음
  + 예를 들어 `plus()` 메서드에 `operator`를 붙이면 `+` 연산자로 사용할 수 있게 됩니다.
  + ```kotlin
    operator fun plus(other: T): T

    data class Point(val x: Int, val y: Int) {
      operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
      }
    }

    val p1 = Point(1, 2)
    val p2 = Point(3, 4)
    val p3 = p1 + p2      // 내부적으로 p1.plus(p2)
    println(p3)           // 출력: Point(x=4, y=6)
    ```

- 오버로딩 가능한 연산자
  + | 연산자  | 함수 이름        | 예시 시그니처                                          |
    | ---- | ------------ | ------------------------------------------------ |
    | `+`  | `plus`       | `operator fun plus(other: T): T`                 |
    | `-`  | `minus`      | `operator fun minus(other: T): T`                |
    | `*`  | `times`      | `operator fun times(other: T): T`                |
    | `/`  | `div`        | `operator fun div(other: T): T`                  |
    | `%`  | `rem`        | `operator fun rem(other: T): T`                  |
    | `[]` | `get`, `set` | `operator fun get(index: Int): T`                |
    | `==` | `equals`     | `operator fun equals(other: Any?): Boolean`      |
    | `!=` | `equals` 사용  | 위와 동일                                            |
    | `++` | `inc`        | `operator fun inc(): T`                          |
    | `--` | `dec`        | `operator fun dec(): T`                          |
    | `()` | `invoke`     | `operator fun invoke(): T`                       |
    | `in` | `contains`   | `operator fun contains(value: T): Boolean`       |
    | `..` | `rangeTo`    | `operator fun rangeTo(other: T): ClosedRange<T>` |

- 주요 예시
  + `[]` 오버로딩 : `get`, `set`
    * ```kotlin
      class MyList {
        private val data = mutableListOf(1, 2, 3)

        operator fun get(index: Int): Int = data[index]

        operator fun set(index: Int, value: Int) {
          data[index] = value
        }
      }

      val list = MyList()
      println(list[0])    // 1 → get 호출
      list[0] = 10        // set 호출
      ```
  + `==` 오버로딩 : `equals`
    * 💡 equals()를 오버라이드할 경우 반드시 hashCode()도 재정의해야 Map, Set 등의 자료구조에서 일관된 동작을 보장합니다. 
    * ```kotlin
      data class User(val name: String) {
        override operator fun equals(other: Any?): Boolean {
          return (other is User) && other.name == name
        }

        override fun hashCode(): Int = name.hashCode()
      }

      val u1 = User("Tom")
      val u2 = User("Tom")
      println(u1 == u2)    // true → equals 호출됨
      ```
  + `()` 오버로딩 : `invoke`
    * ```kotlin
      class Greeter(val message: String) {
        operator fun invoke(name: String) {
          println("$message, $name!")
        }
      }

      val g = Greeter("Hello")
      g("Bae")   // invoke 호출 → Hello, Bae!
      ``` 
   
- 주의사항
  + | 항목                                            | 설명                       |
    | --------------------------------------------- | ------------------------ |
    | `operator`는 제한된 함수 이름에만 사용 가능                 | 정해진 메서드 시그니처 외에는 오버로딩 불가 |
    | `equals()`를 오버라이드하면 반드시 `hashCode()`도 재정의해야 함 | 객체의 동등성 비교시 오류 방지        |
    | 연산자 오버로딩은 **가독성**에 해로울 수 있음                   | 의미가 불분명한 오버로딩은 피할 것      |

- 면접 관련 질문
  + Kotlin에서 `+` 연산자를 오버로딩하려면 어떤 조건을 만족해야 하나요?
    * `plus()` 함수에 `operator` 키워드를 붙여야 하며, 해당 클래스의 멤버 함수이거나 확장 함수여야 합니다.
    * 시그니처는 `operator fun plus(other: T): T` 형태여야 하며, T는 원하는 타입으로 지정할 수 있습니다.
  + `==` 연산자와 `equals()`는 어떤 관계인가요?
    * `==` 연산자는 내부적으로 `equals()`를 호출합니다. 따라서 `==`를 오버로딩하고 싶다면 `equals()` 메서드를 오버라이드해야 합니다. 
    * 또한 `equals()`를 수정한 경우 `hashCode()`도 반드시 일치하는 방식으로 재정의해야 합니다.
  + operator 오버로딩을 남용하면 어떤 문제가 생길 수 있나요?
    * 코드 가독성이 떨어집니다.
    * 예를 들어 *, [], () 같은 연산자를 비표준 용도로 오버로딩하면, 함수 호출 의도가 불분명해져 유지보수가 어려워질 수 있습니다. 
    * 오버로딩은 자연스러운 의미를 가진 연산자만 선택적으로 사용하는 것이 좋습니다.


---
