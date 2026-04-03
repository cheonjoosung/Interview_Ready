### 접근제어자
- 개념 및 정의
  + 클래스, 함수, 변수 등이 어디에서 접근 가능한지 제한하는 키워드
  + 코틀린은 기본적으로 안정성과 명확성을 중시하기 때문에 접근 범위 명시하는 것이 중요함

- 접근 제어자 비교
  + | 접근 제어자   | 클래스 외부 접근 | 상속 클래스 접근 | 같은 파일 내 접근 | 같은 모듈 접근 | 다른 모듈 접근 |
    |---------------|------------------|------------------|-------------------|----------------|----------------|
    | `public`      | ✅               | ✅               | ✅                | ✅             | ✅             |
    | `internal`    | ✅               | ✅               | ✅                | ✅             | ❌             |
    | `protected`   | ❌               | ✅               | ✅ (클래스 내)     | ✅             | ❌             |
    | `private`     | ❌               | ❌               | ✅ (같은 클래스/파일) | ❌             | ❌             |

- public (기본값, 어디서나 접근 가능)
  + **어디서든 접근 가능**하며, 특별한 키워드 없이 선언하면 기본값으로 적용됨
    ```kotlin
    // 파일: Car.kt
    package com.example

    class Car {
        fun drive() = println("Car driving")
    }

    // 다른 패키지
    val car = Car()
    car.drive() // ✅ 정상 접근
    ```
- `internal` (같은 모듈 내에서만 접근 가능)
  + 모듈 단위로 접근 범위 제한
  + 모듈 기준이기 때문에 앱 내부용 코드, 라이브러리 구현 등에서 활용 높음
    ```kotlin
    internal class Engine {
      fun start() = println("Engine start")
    }

    // 같은 모듈
    val e = Engine() // ✅ 가능

    // 다른 모듈
    val e = Engine() // ❌ 컴파일 에러
    ``` 

- protected (상속 관계 내부에서만 접근 가능)
  + 외부에서는 보이지 않지만, 하위 클래스에서는 사용 가능
    ```kotlin
    open class Animal {
      protected fun breathe() = println("숨 쉬는 중")
    }

    class Dog : Animal() {
        fun bark() {
            breathe() // ✅ 가능
        }
    }

    val d = Dog()
    // d.breathe() // ❌ 외부에서는 접근 불가
    ```

- `private` (같은 클래스 or 파일 내에서만 접근 가능)
  + 캡슐화에 최적, 외부에 절대 노출되지 않도록 제한
    ```kotlin
    class Secret {
      private val code = "1234"
      private fun unlock() = println("Unlocked with $code")

      fun tryUnlock() = unlock() // ✅ 내부에서 호출 가능
    }

    // 파일 수준
    // File: Util.kt
    private fun helper() = println("File-local only")

    fun api() = helper() // ✅ 가능

    // 다른 파일에서 helper() 호출 시 컴파일 에러
    ```  

- 실무 활용 팁
  + | 상황                     | 권장 접근 제어자   | 이유           |
    | ---------------------- | ----------- | ------------ |
    | 유틸/헬퍼 함수, internal API | `internal`  | SDK 외부 노출 방지 |
    | 상속 구조 내 공통 로직          | `protected` | 하위 클래스 전용    |
    | 민감한 로직, 상태 캡슐화         | `private`   | 완전 은닉화       |
    | 외부에서 호출 가능한 API        | `public`    | API 명확히 노출   |

- 면접 관련 질문
  + Kotlin에만 있는 접근제어자는?
    * internal. 모듈 단위 접근 제어로 Java에는 존재하지 않음.
  + Java의 default 접근제어자와 Kotlin의 internal 차이?
    * Java의 default는 패키지 단위이고, Kotlin의 internal은 모듈 단위임.
    * internal은 multi-module 프로젝트에서 코드 은닉에 효과적
  + Kotlin에서는 top-level 함수에도 접근제어자를 붙일 수 있는가?
    * 가능. 함수, 클래스, 변수 모두 top-level일 경우에도 private, internal 등 설정 가능.
 

---
