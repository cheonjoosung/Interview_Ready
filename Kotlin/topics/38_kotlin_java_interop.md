### 코틀린 & Java 상호운용성
- 개요
  + Kotlin은 Java와 100% 상호운용 되도록 설계된 언어입니다. Kotlin으로 작성된 .kt 파일은 컴파일 시 .class 파일로 변환되어 Java와 동일한 JVM 바이트코드를 생성합니다. 따라서 Java 코드와 Kotlin 코드를 동일 프로젝트 내에서 섞어서 사용할 수 있습니다.

- 왜 상호윤용성이 중요한가?
  + | 이유           | 설명                                           |
    | ------------ | -------------------------------------------- |
    | ✅ 레거시 코드 유지  | 대부분의 Android 프로젝트는 기존 Java 기반의 코드가 포함되어 있음   |
    | ✅ 점진적 마이그레이션 | Kotlin 도입 시 전체 코드를 한 번에 바꾸기보다는 단계적으로 이전      |
    | ✅ 생태계 활용     | Java 라이브러리, 프레임워크, 도구 등을 Kotlin에서도 그대로 활용 가능 |
 

- Kotlin -> Java 호출 시 유의사항
  + | Kotlin 요소                  | Java에서 호출 방식                                           | 설명                                    |
    | -------------------------- | ------------------------------------------------------ | ------------------------------------- |
    | Top-level 함수               | `UtilsKt.function()` 또는 `@file:JvmName("Utils")` 설정 가능 | 파일명 기반 클래스 생성됨                        |
    | `object` 키워드 사용 객체         | `ObjectName.INSTANCE`                                  | Kotlin의 싱글톤 객체 접근 방식                  |
    | `companion object`         | `ClassName.Companion`                                  | 정적 멤버처럼 접근                            |
    | `@JvmStatic`               | 정적 메서드처럼 `ClassName.method()` 호출 가능                    | `companion object` 또는 `object` 안에서 사용 |
    | `@JvmField`                | 필드처럼 접근 (`obj.field`)                                  | getter/setter 없이 직접 접근 가능             |
    | `@JvmOverloads`            | 자바에서 오버로딩 메서드 자동 생성                                    | Kotlin 기본 인자값이 Java에선 없기 때문           |
    | `@file:JvmName("MyUtils")` | 생성되는 클래스 이름 지정 가능                                      | 이름 충돌 방지 또는 가독성 향상 목적                 |
 
- Java -> Kotlin 호출 시 주의점
  + | 이슈                | 설명                                        | 해결 방법                                      |
    | ----------------- | ----------------------------------------- | ------------------------------------------ |
    | **Null 안정성 부족**   | Java는 null에 대한 명확한 정보가 없음                 | Kotlin에서 \*\*플랫폼 타입(T!)\*\*으로 처리됨          |
    | **명확한 타입 보장 어려움** | Java 메서드의 반환값이 null인지 아닌지 알 수 없음          | `@Nullable`, `@NotNull` 어노테이션을 Java 코드에 명시 |
    | **오버로드된 메서드 처리**  | Kotlin은 기본 인자로 오버로드를 줄이지만 Java는 직접 정의해야 함 | Kotlin에서 `@JvmOverloads` 사용                |
 
- 예시 : 코틀린 코드를 Java 에서 호출
  + ```kotlin
    // File: MyUtils.kt
    @file:JvmName("MyUtils")

    object Logger {
      @JvmStatic
      fun log(message: String) = println("Log: $message")
    }

    @JvmOverloads
    fun greet(name: String = "Guest") {
      println("Hello, $name")
    }
    ``` 
  + ```java
    // Java code
    MyUtils.greet(); // 기본값 사용
    MyUtils.greet("Tom");

    Logger.log("Hi"); // static 메서드처럼 사용
    ```  

- 면접 관련 질문
  + Kotlin과 Java는 정말 100% 상호운용되나요?
    * 기술적으로는 거의 100% 호환되도록 설계되었습니다.
    * Kotlin 컴파일러는 Java와 동일한 .class 파일을 생성하고, JVM 기반에서 동일하게 동작합니다.
    * 단, 일부 언어적 특성(Null 안전성 등)은 개발자가 명시적으로 관리해야 합니다.
  + Kotlin과 Java를 혼용할 때 가장 주의할 점은?
    * Java는 null 안전성 정보가 없기 때문에 Kotlin에서는 플랫폼 타입(T!) 으로 인식되어 NPE 위험이 존재합니다.
    * 이를 방지하려면 Java 코드에 @Nullable, @NotNull 어노테이션을 명확히 지정해야 합니다.
  + Kotlin의 @JvmOverloads는 어떤 상황에서 사용하나요?
    * Kotlin은 기본 인자를 지원하지만, Java는 이를 이해하지 못합니다.
    * 그래서 Java에서 호출 가능하도록, 기본 인자에 따라 오버로드된 메서드를 자동으로 생성하려면 @JvmOverloads를 사용해야 합니다.
    * ```kotlin
      @JvmOverloads
      fun greet(name: String = "Guest", message: String = "Welcome") { ... }
      ```
    * ```java
      greet(); // Guest, Welcome
      greet("Tom"); // Tom, Welcome
      greet("Tom", "Hi"); // Tom, Hi
      ```


---
