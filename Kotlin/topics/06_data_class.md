### data class
- 개념/정의
  + 데이터를 담기 위한 클래스
  + `equals()`, `hashCode()`, `toString()`, `copy()` 등 데이터 처리에 필요한 함수들을 자동 생성
  + 코드를 더 간결하게 효율적으로 작성 가능
  
- 자동으로 생성되는 함수들
  + | 함수          | 역할                                               |
    |---------------|----------------------------------------------------|
    | `toString()`  | `"User(name=철수, age=25)"` 형태 문자열 반환           |
    | `equals()`    | 내용 기반 비교 (`==` 연산자 동작 정의)                  |
    | `hashCode()`  | `equals()` 기반 해시값 생성                            |
    | `copy()`      | 일부 속성만 바꾼 새 객체 생성                          |
    | `componentN()`| 구조 분해 선언 지원 (`val (a, b) = obj`)                |

- 사용 예시
  + ```kotlin
    data class User(val name: String, val age: Int)

    fun main() {
      val user1 = User("철수", 25)
      val user2 = user1.copy(age = 30)

      println(user1)  // User(name=철수, age=25)
      println(user2)  // User(name=철수, age=30)

      val (name, age) = user2
      println(name)   // 철수
      println(age)    // 30
    }
    ``` 

- 제약사항
  + primary constructor에 최소 하나 이상의 프로퍼티 필수
  + `open`, `abstract`, `sealed`, `inner` 클래스로 선언할 수 없음
  + 상속 불가 (`final`로 선언 됨)

- 활용 사례
  + API 응답 데이터 모델
  + 로컬 캐시 객체
  + ViewModel의 UI 상태 모델 (ex. data class UiState)
  + 구조 분해 기반 이벤트 전달 (val (type, message) = result)

- 면접 관련 질문
  + 일반 클래스와 data class의 차이점은?
    * 일반 클래스는 toString(), equals(), hashCode() 등을 직접 구현해야 하지만, data class는 주 생성자 프로퍼티 기반으로 이들 메서드를 자동 생성해준다.
    * 결과적으로 더 간결하고 실용적인 데이터 구조를 만들 수 있다.
  + copy() 함수의 장점은?
    * copy()는 기존 객체의 상태를 유지하면서, 일부 프로퍼티만 변경한 새 인스턴스를 쉽게 만들 수 있다.
    * 불변 객체 패턴에서 매우 유용하며, 상태 관리나 UI 변경 감지 로직 등에 자주 쓰인다.
  + data class를 상속할 수 없는 이유는?
    * Kotlin은 data class를 값 객체로 간주하므로, 동등성, 해시값, 복사 등에 대한 일관된 동작이 필요하다.
    * 상속을 허용하면 이런 계약이 깨질 수 있어 자동으로 final로 선언된다. 


---
