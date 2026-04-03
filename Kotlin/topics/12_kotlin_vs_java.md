### Kotlin & Java
- 두 언어 모두 JVM에서 실행되는 언어, 안드로이드 개발이나 서버 개발에서 많이 쓰임 

- 공통점
  + | 항목            | 설명 |
    |-----------------|------|
    | JVM 기반        | 둘 다 Java Virtual Machine에서 실행 |
    | Java API 호환   | Kotlin은 Java API 100% 호출 가능 |
    | 객체지향 지원    | 클래스, 인터페이스, 상속, 다형성 등 동일 구조 |
    | Android 개발 가능 | Android 공식 지원 언어 (Java 7+ / Kotlin 1.3+) |
    | 스레드, 네트워크 등 | 동등 수준의 저수준 기능 지원 가능 |

- 차이점
  + | 항목                | Java                          | Kotlin                                  |
    |---------------------|-------------------------------|------------------------------------------|
    | 문법 간결성         | 장황한 편                      | `val`, `data class`, `when` 등 간결 |
    | Null 안정성         | `NullPointerException` 자주 발생 | 컴파일 타임에서 null 검사 (`?`, `!!`) |
    | 데이터 클래스        | getter/setter, equals 수동 작성 | `data class` 한 줄로 자동 생성         |
    | 함수형 스타일        | 람다/Stream 사용 불편              | 고차 함수, 람다, `map/filter` 자연스러움 |
    | 확장 함수           | 불가                             | 클래스 외부에서 메서드 확장 가능        |
    | 타입 캐스팅         | 강제 `instanceof + cast`         | `is` + 스마트 캐스팅 자동화            |
    | 비동기/코루틴 지원   | 외부 스레드/라이브러리 필요         | `suspend`, `launch`, `async` 기본 지원  |
    | 기본 타입 분리       | `int`, `double` 등 primitive 존재 | 전부 객체(`Int`, `Double`)로 통일        |


- 예시
  + ```java
    // 변수선언
    String name = "철수";

    // Null 안전 처리
    if (user != null) {
      System.out.println(user.getName());
    }

    // 데이터 클래스
    public class User {
      String name;
      int age;

      // 생성자, getter, equals, hashCode ...
    }
    ```
  + ```kotlin
    // 변수선언
    val name = "철수"   // 자동 타입 추론

    // Null 안전 처리
    println(user?.name ?: "이름 없음") // null-safe + default

    // 데이터 클래스
    data class User(val name: String, val age: Int)
    ```

- 결론
  + Kotlin은 간결함 + 안전성 + 현대적 문법에 중점을 둔 언어
  + Java는 보편성 + 방대한 생태계를 갖춘 기반 언어
  + Android 개발이나 현대 서버 개발에서는 Kotlin이 점점 우세
  + 하지만 상호운용성 완벽 → 둘을 함께 사용하는 프로젝트도 많음 

- 면접 관련 질문
  + Kotlin이 Java보다 유리한 점은?
    * Null 안정성, 코루틴, 간결한 문법, 고차 함수 등의 지원으로 코드 품질 및 생산성이 향상됩니다.
  + Kotlin 코드가 Java에서 호출 가능한가요?
    * 가능함. Kotlin은 JVM 바이트코드로 변환되므로 Java 클래스와 완전히 상호 운용됩니다. 
    * @JvmStatic, @JvmOverloads 등으로 더 자연스러운 호환도 가능.
  + Kotlin의 확장 함수는 Java의 어떤 대안인가요?
    * 기존 클래스에 새로운 메서드를 추가할 수 있는 방식으로, Java의 유틸 클래스보다 가독성과 사용성이 좋습니다.
    * 하지만 private 멤버에는 접근 불가한 제약도 있습니다.


---
