### 예외 처리
- 예외란
  + 실행 중 발생하는 오류 상황을 **객체로 표현**
  + 모든 예외는 `Throwable`을 상속
  + ```text
    Throwable
    ├── Error               // 시스템 레벨 심각 오류 (복구 불가)
    └── Exception           // 애플리케이션 수준 예외
        ├── RuntimeException // 주로 프로그래밍 오류
        └── IOException      // 입출력 중 발생
    ```

- 코틀린 예외 처리 특징
  + | 특징                  | 설명                                          |
    | ------------------- | ------------------------------------------- |
    | Unchecked Exception | Java처럼 강제 예외 선언 없음                          |
    | throw는 표현식          | `throw`는 값처럼 동작 가능 (`Nothing` 반환)           |
    | try도 표현식            | `val result = try { ... } catch { ... }` 가능 |

- 예외 발생
  + ```kotlin
    fun checkAge(age: Int) {
      if (age < 0) throw IllegalArgumentException("Age cannot be negative")
    }
    ``` 

- 전제 조건 함수
  + ```kotlin
    fun process(input: String?) {
      require(!input.isNullOrEmpty()) { "Input must not be null or empty" } // IllegalArgumentException
    }

    class User(val id: Int, var name: String) {
      fun rename(newName: String) {
          check(newName.length > 1) { "Name too short" } // IllegalStateException
      }
    }

    fun fail(message: String): Nothing = throw IllegalArgumentException(message)
    val user = name ?: fail("No name!")
    ```
  + | 함수        | 예외 타입                      | 사용 용도                 |
    | --------- | -------------------------- | --------------------- |
    | `require` | `IllegalArgumentException` | 함수 인자 검증              |
    | `check`   | `IllegalStateException`    | 객체 상태 검증              |
    | `error`   | `IllegalStateException`    | 무조건 예외 발생 (불가능한 분기 등) |

- 예외 잡기 (try-catch-finally)
  + ```kotlin
    fun readFile(path: String) {
      var reader: BufferedReader? = null
      
      try {
          reader = File(path).bufferedReader()
          println(reader.readLine())
      } catch (e: FileNotFoundException) {
          println("File not found: $path")
      } catch (e: IOException) {
          println("I/O error: ${e.message}")
      } finally {
          reader?.close()
          println("Always runs!")
      }
    }
    ``` 

- try 표현식
  + ```kotlin
    val message = try {
      "Success"
    } catch (e: Exception) {
      "Failure: ${e.message}"
    }
    ``` 

- 핵심 요약
  + | 개념              | 요약 설명                                        |
    | --------------- | -------------------------------------------- |
    | 예외 구조           | `Throwable` → `Exception` / `Error`          |
    | `throw`         | 표현식 (리턴 값처럼 사용됨) → 보통 `Nothing` 리턴 함수와 함께 사용 |
    | `require/check` | 조건 위반 시 예외 → 명확한 계약(Contract) 문서화 가능         |
    | try-catch       | 예외 발생 가능 코드 감싸기 + 다양한 예외 분기 + 자원 정리까지        |
 
- 면접 관련 질문
  + 코틀린은 Checked Exception을 지원하나요?
    * 아니요. Kotlin은 모든 예외가 Unchecked입니다.
    * Java의 throws와 같은 선언이 필요 없습니다.
  + `require`, `check` 차이점
    * | 함수      | 목적               | 예외 타입                    |
      | ------- | ---------------- | ------------------------ |
      | require | 함수에 넘긴 **인자 검증** | IllegalArgumentException |
      | check   | 객체의 **상태 검증**    | IllegalStateException    |
  + throw는 왜 표현식인가요?
    * throw는 Nothing 타입을 반환하므로, 삼항 연산자나 Elvis (?:) 등과 함께 사용할 수 있어 제어 흐름을 깔끔하게 구성할 수 있습니다.
  + 예외가 발생한 이후 finally는 반드시 실행되나요?
    * 네, finally는 예외 발생 여부와 관계없이 항상 실행됩니다. 
    * 단, System.exit()나 JVM 자체 종료 같은 상황은 예외입니다.


---
