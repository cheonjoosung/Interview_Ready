### const
- 개념 및 정의
  + Kotlin에서 const는 **컴파일 타임 상수(compile-time constant)**를 선언할 때 사용하는 키워드입니다.
  + 자바의 public static final과 유사하며, 값이 프로그램 실행 전에 결정되어야 합니다.

- 기본 사용법
  + ```kotlin
    const val PI = 3.14
    const val APP_NAME = "My App"
    ```
  + 위 값들은 컴파일 시점에 확정되며, 변경 불가합니다.

- const & val 비교
  + | 항목          | `const val`                                    | `val`                            |
    | ----------- | ---------------------------------------------- | -------------------------------- |
    | 초기화 시점      | **컴파일 타임**                                     | **런타임**                          |
    | 사용 가능 타입    | 기본 타입만 가능 (`Int`, `String`, `Double` 등)        | 모든 타입 가능 (`List`, `Object` 등 포함) |
    | 선언 가능한 위치   | **top-level**, `object`, `companion object` 내부 | 어디든 가능                           |
    | 목적          | 상수 값(리터럴)을 정의                                  | 변경 불가능한 값 정의 (지연 초기화 가능)         |
    | Java에서 사용 시 | `public static final`처럼 노출                     | 일반 필드처럼 접근됨 (getter 필요)          |


- 선언 가능한 위치
  + | 위치                 | 설명               | 예시                                               |
    | ------------------ | ---------------- | ------------------------------------------------ |
    | ✅ top-level        | Kotlin 파일 최상단    | `const val BASE_URL = "https://api.com"`         |
    | ✅ object 내부        | 싱글톤 객체 내부        | `object Config { const val TOKEN = "abc" }`      |
    | ✅ companion object | 클래스의 companion 내 | `companion object { const val VERSION = "1.0" }` |
    | ❌ 함수 내부            | 함수 블록 내 선언 불가    | `fun test() { const val X = 1 } // 오류!`          |


- const가 필요한 이유
  + | 상황                  | 설명                                              |
    | ------------------- | ----------------------------------------------- |
    | **Annotation 파라미터** | Annotation에 상수를 넣을 때는 반드시 `const val`이어야 함      |
    | **Java 상호운용**       | Java에서 Kotlin 상수를 `public static final`처럼 사용 가능 |
    | **공용 상수 관리**        | DSL, 설정 값, enum-like 문자열 등에서 상수를 명확하게 정의 가능     |


- Java 코드에서 사용 예시
  + ```kotlin
    // Kotlin
    object Config {
      const val TOKEN = "abc123"
    }
    ```
  + ```java
    // Java
    String token = Config.TOKEN;  // → public static final처럼 동작
    ```
  
- 요약 
  + | 항목       | 설명                                  |
    | -------- | ----------------------------------- |
    | 선언 키워드   | `const val`                         |
    | 초기화 시점   | 컴파일 타임                              |
    | 사용 가능 타입 | 기본 타입만 (`Int`, `Long`, `String` 등)  |
    | 선언 위치    | top-level, object, companion object |
    | 주요 사용 사례 | Annotation, Java interop, 설정 상수 등   |

- 면접질문
  + `val`과 `const val`의 차이는 무엇인가요?
    * `val`은 런타임에 값이 결정되며, 모든 타입을 사용할 수 있습니다.
    * `const val`은 컴파일 타임 상수로, 기본 타입만 사용 가능하고 특정 위치(top-level, object 등)에만 선언할 수 있습니다.
  + `const`는 어디에서 선언할 수 있나요?
    * `Top-level` (파일 최상단)
    * `object` 내부
    * `companion object` 내부 ➡️ 함수나 일반 클래스 내부에서는 선언 불가
  + 그냥 `val` 쓰면 안 되고, 왜 굳이 `const`를 써야 하나요?
    * const는 컴파일 타임에 값이 확정되므로 퍼포먼스 측면에서 더 효율적입니다.
    * 특히 Annotation 파라미터나 Java와의 호환을 고려할 때 필수입니다.
    * 예: 아래와 같은 annotation은 const가 아니면 컴파일되지 않습니다.
    * ```kotlin
      @Header("Authorization: $TOKEN") // TOKEN은 반드시 const 여야 함
      ```


---
