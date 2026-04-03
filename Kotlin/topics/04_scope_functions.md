### 스코프 함수(let, run with, apply, also)
- 개념 및 정의
  + 객체 컨텍스트 내부에서 블록 코드를 실행할 수 있도록 도와주는 함수
  + 간결하고, 읽기 쉽고, null-safe 만들어주며 객체 초기화, 설정, 연산, 로깅 등에 널리 사용
  + let, run, with, apply, also 등 주로 쓰임

- 컨텍스트 참조 방식 & 반환값 기준
  + | 함수   | 객체 참조 방식 | 반환 값          | 사용 목적 요약                           |
    |--------|----------------|------------------|------------------------------------------|
    | `let`  | `it`           | 람다 결과         | null-safe 연산, 결과 변환                |
    | `run`  | `this`         | 람다 결과         | 설정 + 결과 반환                         |
    | `with` | `this`         | 람다 결과         | 멤버 접근 중심 연산                      |
    | `apply`| `this`         | 객체 자체 반환     | 설정 후 객체 반환 (빌더 패턴처럼)        |
    | `also` | `it`           | 객체 자체 반환     | 부가 작업 (로깅, 디버깅, 체이닝 등)      |

- let
  + 객체 참조 `it`
  + 반환 값 : 람다 마짐가 표현식 결과
  + 사용하는 경우
    * Null 체크와 함께 코드 실행
    * 연산 결과를 변수에 할당
    * 메서드 체이닝
  + ```kotlin
    val result = nullableObj?.let {
      println("not null: $it")
      it.length
    }
    ``` 

- run
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 사용하는 경우
    * 여러 프로퍼티/메서드를 한 블록에서 접근
    * 블록 실행 결과를 반환하고 싶을 때
    * 객체 초기화 + 연산
  + ```kotlin
    val result = person.run {
     println(name)
      age + 5
    }
    ``` 

- with
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 사용하는 경우
    * 확장 함수가 아닌 일반 함수
    * 객체에 여러 번 접근할 때 가독성 향상
    * 주로 수동 DSL 문법에서 자주 사용
  + ```kotlin
    val result = with(person) {
      println(name)
      age + 10
    }
    ``` 

- apply
  + 컨택스트 객체 : this
  + 반환 값 : 객체 자체
  + 주요 사용 경우
    * 객체 초기화 후 그 객체를 그대로 리턴하고 싶을 때
    * 빌더 패턴 대체
  + ```kotlin
    val person = Person().apply {
      name = "Alice"
      age = 30
    }
    ``` 

- also
  + 컨텍스트 객체 : it
  + 반환 값 : 객체 자체
  + 주요 사용 경우
    * 로깅, 디버깅 등 부가 작업
    * 체이닝 내에서 원본 객체 유지하면서 추가 작업 수행
  + ```kotlin
    val person = Person("Alice", 30).also {
      println("Creating person: $it")
    }
    ``` 

- 선택 기준 요약
  + | 사용 상황                 | 추천 함수   |
    | --------------------- | ------- |
    | null 체크 + 결과 변환       | `let`   |
    | 객체 설정 후 다른 결과 반환      | `run`   |
    | 객체 설정 후 그 객체 반환       | `apply` |
    | 부가 작업 후 객체 반환 (예: 로깅) | `also`  |
    | 여러 멤버에 반복 접근 + 결과 반환  | `with`  |

- 추가 팁
  + with 는 확장 함수가 아님
    * `with(obj) { ... }` 형태로 사용 (run/apply 등은 obj.run {...} 식)
  + let + Elvis
    * null-safe 처리에 가장 많이 쓰이는 조합
    * val result = input?.let { it.trim() } ?: "default"
  + apply vs also
    * 결과는 같지만, `this` vs `it` 문맥 차이가 존재
    * `apply` 설정용, `also` 부가행위

- 면접 관련 질문
  + apply와 also의 차이는?
    * 둘 다 객체를 반환하지만, apply는 블록 안에서 this로 객체를 참조하며 설정 목적에 적합하고,also는 it으로 객체를 참조하여 로깅/검증/추가 작업 등에 적합합니다.
  + run과 with의 차이는?
    * run은 확장 함수로 객체에 바로 붙여 쓸 수 있고, with는 일반 함수이므로 객체를 인자로 전달해야 합니다.
    * 용도는 유사하지만 호출 형태에 차이가 있습니다.
  + 스코프 함수 선택 기준은 어떻게 되나요?
    * 연산 결과 반환이 중요하면 let, run, with
    * 객체 자체 반환이 필요하면 apply, also
    * null-safe 연산엔 let, 초기화에는 apply, 부가작업엔 also가 적합합니다.
 

---
