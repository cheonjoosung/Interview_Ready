### typealias
- 정의
  + `typealias`는 기준 타입에 새로운 이름(별칭) 부여하는 문법
  + 새 타입을 만드는 것이 아니라 기존 타입을 가리키는 별명(alias) 역할

- 기본 사용법
  + typealias 키워드를 사용하여 새로운 이름과 기존 타입을 연결합니다.
  + ```kotlin
    typealias Name = String
    typealias UserList = List<User>
    typealias ClickListener = (View) -> Unit
    ```
    
- 사용하는 이유 및 장점
  + 가독성향상
    * 복잡하거나 긴 타입 이름을 의미 있는 이름으로 대체하여 코드를 더 쉽게 이해할 수 있도록 합니다.
    * ```kotlin
      val handler: (Int, String, Boolean) -> Unit // ❌ 직관성 낮음
      typealias ErrorHandler = (Int, String, Boolean) -> Unit
      val handler: ErrorHandler                    // ✅ 의미 명확
      ```
  + 복잡한 제네릭 타입 간소화
    * 제네릭을 사용하는 복잡한 타입을 더 짧고 관리하기 쉬운 이름으로 만들 수 있습니다.
    + ```kotlin
      // BEFORE
      val processor: (List<Map<String, Set<Int>>>, (String) -> Boolean) -> List<String>

      // AFTER
      typealias DataFilter = (String) -> Boolean
      typealias ComplexData = List<Map<String, Set<Int>>>
      typealias DataProcessor = (ComplexData, DataFilter) -> List<String>

      val processor: DataProcessor = { data, filter -> emptyList() }\
      ```
  + 콜백, 함수 타입 정리
    * ```kotlin
      typealias LoginCallback = (Boolean, String?) -> Unit

      fun login(id: String, pw: String, callback: LoginCallback) { ... }
      ```
  + 제네릭 타입
    * ```kotlin
      typealias StringList<T> = List<T>
      val users: StringList<String> = listOf("A", "B")

      typealias IntList = List<Int>
      val numbers: IntList = listOf(1, 2, 3)
      ```
  + 내부 클래스, 객체에도 사용 가능
    * ```kotlin
      class Outer {
        inner class Inner
        object NestedObject
      }

      typealias InnerClass = Outer.Inner
      typealias NestedObj = Outer.NestedObject
      ``` 
    
- 주의사항
  + | 항목        | 설명                                         |
    | --------- | ------------------------------------------ |
    | 새로운 타입 아님 | `typealias`는 진짜 새로운 타입이 아니며, 컴파일 타임에 치환될 뿐 |
    | 생성자 없음    | `typealias` 자체에 생성자, 프로퍼티를 추가할 수 없음        |
    | 상속 불가     | `open`, `abstract`, `sealed` 등 사용 불가       |
    | 충돌 주의     | 동일한 타입에 너무 많은 별칭을 만들면 오히려 혼란을 초래할 수 있음     |

- 언제 typealias를 사용하면 좋을까?
  + | 상황               | 이유                                             |
    | ---------------- | ---------------------------------------------- |
    | 복잡한 타입 반복 사용     | 의미 있는 이름으로 정의하여 재사용 및 간결화                      |
    | 함수 타입 콜백 명확화     | 매개변수/반환값 구조를 명확하게 표현 가능                        |
    | 도메인에 맞는 타입 의미 부여 | String → `UserName`, Int → `UserId` 등으로 표현력 향상 |

- 면접 관련 질문
  + typealias는 어떤 역할을 하나요?
    * 기존 타입의 별칭을 제공해 복잡한 타입을 간결하고 명확하게 표현할 수 있게 해줍니다.
    * 실제 타입은 변하지 않고, 컴파일 타임에 단순히 치환됩니다.
  + typealias는 새 타입을 만드는 건가요?
    * 아니요. typealias는 새 타입이 아닙니다. 동일한 타입에 다른 이름을 부여하는 것뿐입니다.
    * 예: typealias Email = String → 여전히 String으로 동작합니다.
  + 언제 typealias를 적극 사용하는 것이 좋을까요?
    * 복잡한 함수 타입, 제네릭 타입을 반복해서 사용할 때
    * 도메인 개념에 맞춰 코드의 표현력과 명확성을 높이고 싶을 때
    * 공통 콜백 타입이나 인터페이스 구조를 읽기 쉬운 코드로 만들고 싶을 때 


---
