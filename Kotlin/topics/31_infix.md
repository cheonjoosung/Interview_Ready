### infix 함수
- 개념/정의
  + 점(`.`) or 괄호 없이 중위 표기법(infix notation)으로 호출할 수 있는 함수
  + 일반 함수 호출 방식인 `a.method(b)`를 `a method b` 형태로 더 자연스럽게 사용 가능
  + ```kotlin
    a.plus(b)        // 일반 호출
    a plus b         // infix 호출 (infix fun plus 정의되어 있다면)
    ```

- 사용 조건
  + | 조건                 | 설명                       |
    | ------------------ | ------------------------ |
    | `infix` 키워드 필수     | 함수 정의 앞에 `infix`를 붙여야 함  |
    | **멤버 함수 or 확장 함수** | 둘 중 하나여야 함               |
    | **파라미터는 하나만 허용**   | `vararg`, 디폴트 파라미터 모두 불가 |

- 예제
  + ```kotlin
    infix fun Int.multiply(x:Int): Int {
        return this * x
    }
    println(5 multiply 3) // 15
    ```

- 특징
  + 괄호/점 생략 → 코드 가독성 향상
  + Kotlin DSL 작성에 자주 사용됨 (예: 안드로이드 build.gradle.kts)
  + 연산자처럼 사용할 수 있어 의도 전달이 쉬움

- 면접 관련 질문
  + infix 함수는 왜 필요한가요?
    * 함수 호출을 자연어처럼 읽을 수 있게 해줘 가독성이 좋아집니다.
    * 특히 DSL(Domain Specific Language)이나 수학적 표현에 적합합니다.
  + infix 함수의 사용 조건은?
    * infix 키워드를 붙여야 하며, 클래스의 멤버 함수 또는 확장 함수여야 하고, 하나의 인자만 받을 수 있으며, vararg, 디폴트 파라미터는 사용할 수 없습니다.
  + infix 함수와 operator 함수의 차이점은?
    * | 항목    | infix                    | operator                  |
      | ----- | ------------------------ | ------------------------- |
      | 목적    | 중위 표기 지원                 | 연산자 오버로딩 지원               |
      | 표기 방식 | `a myFunc b`             | `a + b`, `a * b` 등        |
      | 키워드   | `infix`                  | `operator`                |
      | 사용 예시 | `a to b`, `5 multiply 3` | `Point(1,2) + Point(3,4)` |


---
