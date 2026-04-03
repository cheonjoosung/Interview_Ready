### 가변인자(vararg)
- 정의
  + `vararg` 코틀린 함수에서 같은 타입의 인자를 0개 이상 받을 수 있도록 해주는 기능
  + 호출 시 인자 개수를 유동적으로 전달할 수 있어 함수의 유연성과 간결성 높여줌
  + 기본사용법
    * ```kotlin
      fun printNumbers(vararg numbers: Int) {
        for (number in numbers) {
            print("$number ")
        }
        println()
      }

      fun main() {
        printNumbers(1, 2, 3)       // 출력: 1 2 3
        printNumbers(4, 5, 6, 7, 8) // 출력: 4 5 6 7 8
        printNumbers()              // 출력: (아무것도 출력 안 함, 빈 줄)
      }
      ```

- 주요 특징 및 설명
  + | 항목          | 설명                                                                         |
    | ----------- | -------------------------------------------------------------------------- |
    | `vararg` 위치 | 함수 파라미터 선언 시 타입 앞에 `vararg` 키워드 사용                                         |
    | 내부 처리 방식    | 인자들은 내부적으로 배열(`Array<T>`)로 처리됨                                             |
    | 인자 전달 방식    | - 개별 값: `funA(1, 2, 3)` <br> - 배열 사용 시: `funA(*intArray)` (스프레드 연산자 사용 필수) |
    | 제약 사항       | - 함수 당 하나의 `vararg`만 사용 가능 <br> - 가급적 파라미터 목록의 마지막에 위치시켜야 함                |
    | 제네릭과의 결합    | 제네릭 타입으로도 사용 가능 (`fun <T> example(vararg items: T)`)                       |

- 활용 방법
  + 스프레드 연산자 `*`
    * 배열을 가변인자 함수에 전달할 경우 배열 앞에 * 사용
    * `printValues(messages)`처럼 `*` 없이 배열을 직접 전달하면 컴파일 오류 발생
    * ```kotlin
      fun printValues(vararg values: String) {
        values.forEach { println(it) }
      }

      fun main() {
        val messages = arrayOf("Hello", "World")
        printValues(*messages) // "Hello", "World"가 개별 인자로 전달됨
      }
      ```
  + 위치 제약과 Named Argument 사용  
    * vararg 뒤에 다른 파라미터가 있다면, 해당 파라미터는 이름을 명시해서 전달해야 합니다. 
    * ```kotlin
      fun processData(vararg values: Int, multiplier: Int) {
        for (value in values) {
          println(value * multiplier)
        }
      }

      fun main() {
        processData(1, 2, 3, multiplier = 10) // OK
        // processData(1, 2, 3, 10)         // 오류! multiplier인지 구분 불가
      }
      ```
  + 제네릭과 vararg
    * 제네릭과 함께 쓰면 다양한 타입의 가변 인자를 받아들일 수 있음 
    * ```kotlin
      fun <T> printAll(vararg items: T) {
        for (item in items) {
          println(item)
        }
      }

      fun main() {
        printAll("apple", "banana")
        printAll(1, 2, 3)
        printAll(true, false)
      }
      ``` 

- vararg 유용한 경우
  + 로깅 함수: log("message1", "message2", "message3")
  + 문자열 포맷팅: String.format("%s %s", "Hello", "World")
  + 빌더 패턴: listOf(vararg elements: T)
  + 테스트 코드에서 복수의 테스트 데이터 전달

- 장점 & 주의 사항
  + | 장점                        | 주의사항                                               |
    | ------------------------- | -------------------------------------------------- |
    | 다양한 인자 수를 유연하게 처리 가능      | 호출 시마다 배열이 생성되어 성능에 약간의 영향                         |
    | 코드 간결성 – 여러 개의 오버로딩 필요 없음 | Java와 상호 운용 시 `@JvmOverloads`, `@JvmVarargs` 고려 필요 |

- 면접 관련 질문
  + vararg 파라미터 뒤에 일반 파라미터를 둘 수 있나요?
    * 가능은 하지만, 반드시 이름 있는 인자(named argument) 로 호출해야 합니다.
    * 그렇지 않으면 어떤 인자가 어디에 매핑되는지 모호해져 컴파일 오류가 발생합니다.
  + vararg와 스프레드 연산자의 차이점은 무엇인가요?
    * vararg: 함수 정의 시, 가변 인자 선언에 사용
    * `*` (스프레드 연산자): 배열을 함수에 개별 인자로 펼쳐서 전달할 때 사용
  + vararg는 내부적으로 어떻게 처리되며, 성능상 단점은 없나요?
    * vararg 파라미터는 내부적으로 배열로 변환되어 처리됩니다. 
    * 그 때문에 함수가 자주 호출되는 상황에서는 불필요한 배열 생성으로 인한 약간의 오버헤드가 발생할 수 있지만, 대부분의 경우 실질적인 성능 저하는 크지 않습니다. 


___
