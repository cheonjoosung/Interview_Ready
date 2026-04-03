### init 블록 & constructor()
- `init` 블록
  + | 특징             | 설명                                                                 |
    |------------------|----------------------------------------------------------------------|
    | 실행 시점         | 객체가 생성될 때 **자동으로 실행됨**                                           |
    | 위치             | 클래스 본문 내부, 여러 개 작성 가능                                            |
    | 용도             | 초기값 검증, 로그 출력, 계산 등 **공통 초기화 로직 처리**                           |
    | 실행 순서         | 생성자 호출 후 → `init` 블록 실행                                         |
  + ```kotlin
    class User(val name: String) {        
        init {
            println("init name is $name")
        }
    }
    ```
    
- `constructor()`
  + | 구분     | 설명                                                           |
    | ------ | ------------------------------------------------------------ |
    | 주 생성자  | 클래스 헤더에 정의, **가장 기본적인 생성자**                                  |
    | 보조 생성자 | `constructor` 키워드로 클래스 본문 안에 정의, **다양한 인자 조합을 지원**           |
    | 호출 순서  | 보조 생성자 → 주 생성자 호출 (`: this(...)`) → `init` 블록 → 보조 생성자 블록 실행 |
  + ```kotlin
    class User(val name: String) {
        constructor(name: String, age: Int) : this(name) {
            println("constructor name is $name age is $age")
        }
    }
    ```
    
- 실행 흐름 예시
  + ```kotlin
    class Book(val title: String) {
      init {
          println("📘 Book title: $title") // init 블록
      }

      constructor(title: String, author: String) : this(title) {
          println("✍️ Author: $author") // 보조 생성자
      }

      constructor(title: String, author: String, price: Int) : this(title) {
          println("💰 Author: $author, Price: $price")
      }
    }

    fun main() {
      Book("해리포터")
      Book("해리포터2", "롤링")
      Book("해리포터3", "롤링", 10000)
    }
    /*
    📘 Book title: 해리포터
    📘 Book title: 해리포터2
    ✍️ Author: 롤링
    📘 Book title: 해리포터3
    💰 Author: 롤링, Price: 10000
    */
    ``` 

- 요약
  + | 항목    | init 블록             | constructor() (보조 생성자)             |
    | ----- | ------------------- | ---------------------------------- |
    | 실행 시점 | 생성 시 자동 실행          | 명시적으로 호출해야 실행됨                     |
    | 주 용도  | **공통 초기화** 로직 처리    | **다양한 생성 방식**을 허용                  |
    | 개수 제한 | 여러 개 가능             | 여러 개 가능 (`오버로드` 형태)                |
    | 실행 순서 | 주 생성자 → `init` 실행 순 | 보조 생성자 → 주 생성자 → `init` → 보조 블록 실행 |

- 면접 관련 질문
  + init 블록과 constructor의 차이는?
    * `init`은 공통 초기화 코드
    * `constructor`는 인자 조합을 달리해 다양한 방식으로 인스턴스를 생성할 수 있도록 함
  + init이 constructor보다 먼저 실행되나요?
    * 보조 생성자가 먼저 호출되지만, 실행은 주 생성자의 필드 초기화 → init → 보조 생성자 본문 순서로 진행됨
  + 생성자 파라미터에 검증 로직을 넣고 싶다면?
    * init 블록에서 검증하는 것이 좋음
    * 생성자에선 값만 받고, 검증/로깅/비즈니스 로직은 분리하는 게 유지보수에 유리


---
