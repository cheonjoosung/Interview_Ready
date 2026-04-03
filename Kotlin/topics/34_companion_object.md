### companion object
- 개념 정의
  + 클래스의 인스턴스를 생성하지 않고도 접근 가능한 정적(static) 멤버를 선언할 수 있게 해주는 기능
  + 코틀린은 Java의 `static` 키워드가 없기 때문에 이를 객체 기반으로 우회 구현
  + Java의 `static`과 유사한 역할을 하지만 객체처럼 취급할 수 있다는 점에서 더 유연
  + 기본 사용 예시
    * ```kotlin
      class MyClass {
        companion object {
          const val PI = 3.14

          fun create(): MyClass = MyClass()
        }
      }

      // 사용
      val obj = MyClass.create()
      println(MyClass.PI)
      ```
    * `MyClass`를 new 없이 바로 접근 가능
    * PI, create() 모두 정적처럼 사용됨

- 주요 특징 요약
  + | 항목          | 설명                                                                                     |
    | ----------- | -------------------------------------------------------------------------------------- |
    | 정적 접근       | 클래스명으로 직접 접근 가능: `MyClass.PI`, `MyClass.create()`                                      |
    | 싱글턴         | 클래스 당 오직 **1개**의 companion object만 존재                                                  |
    | 이름 지정 가능    | `companion object Factory { ... }`처럼 이름 부여 가능                                          |
    | 인터페이스 구현 가능 | 일반 객체처럼 인터페이스 구현 가능                                                                    |
    | 자바 호환성      | Java에서는 `MyClass.Companion.method()`로 접근<br> → `@JvmStatic` 사용 시 `MyClass.method()` 가능 |
 
- 다양한 사용법
  + 이름 있는 companion object 예시
    * ```kotlin
      class Logger {
        companion object Factory {
          fun create(): Logger = Logger()
        }
      }

      // 사용
      val logger = Logger.create()          // 기본 방식
      val logger2 = Logger.Factory.create() // 이름 지정 시 가능
      ```
  + 팩토리 메서드 + 상수 
    * ```kotlin
      class User private constructor(val name: String) {
        companion object {
          const val DEFAULT_NAME = "Guest"

          fun create(name: String?): User {
            return User(name ?: DEFAULT_NAME)
          }
        }
      }

      // 사용
      val user1 = User.create("Bae")
      val user2 = User.create(null) // → "Guest" 사용
      ```

- companion object 사용 예시
  + | 상황               | 설명                              |
    | ---------------- | ------------------------------- |
    | 팩토리 메서드          | `User.create(...)` 등으로 객체 생성 제어 |
    | 정적 상수 정의         | `const val VERSION = "1.0"` 등   |
    | 자바 static과 호환    | `@JvmStatic` 어노테이션 활용 가능        |
    | 클래스 내부에서 공유 함수/값 | 유틸리티 함수나 상수, 내부 공유 자원 관리 등에 사용  |

- 주의 사항
  + | 항목                  | 설명                                                                                               |
    | ------------------- | ------------------------------------------------------------------------------------------------ |
    | Kotlin에 `static` 없음 | 반드시 `companion object`로 정적 멤버를 구현해야 함                                                            |
    | `const val` 제한      | `const val`은 오직 `top-level` 또는 `companion object` 안에서만 선언 가능                                     |
    | 상태 저장 시 동기화 고려 필요   | `companion object`는 싱글턴 → 상태 공유 시 race condition 발생 위험 있음<br>→ `@Synchronized`, `volatile` 사용 고려 |

- 면접 관련 질문 
  + companion object는 어떤 특징을 가지고 있나요?
    * 클래스에 1개만 선언 가능 (싱글턴 객체)
    * 클래스명으로 정적 접근 가능
    * 이름 부여 가능 (companion object Factory)
    * 일반 객체처럼 인터페이스 구현 가능
  + companion object `내부에서 생성자(private constructor)`를 사용할수 있는 이유는 무엇인가요?
    * `companion object`는 클래스의 내부 스코프이기 때문에 `private constructor`에 접근 가능 → 외부에는 생성자를 숨기고, 정적 팩토리 메서드만 제공하는 구조 구현 가능
  + companion object에 상태(state)를 저장해도 되나요?
    * 가능하지만 companion object는 기본적으로 싱글턴이기 때문에 **여러 스레드에서 동시에 접근하면 race condition이 발생합니다.**
    * 따라서 공유상태가 있다면 적절한 동기화 (ex) @Synchronized, volatile)가 필요합니다.


---
