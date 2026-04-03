### 플랫폼 타입
- 개념
  + 플랫폼 타입(platform type)은 코틀린이 자바 코드를 호출할 때
    * Java 의 Nullable 여부를 정확히 알 수 없기 때문
    * Kotlin 대신 `nullable` / `non-null`을 추론하지 않고 보류하는 타입
  + 이 값이 null 이 될 수도 있고 아닐 수도 있은 개발자가 책임지고 처리하라 라는 의도를 가지는 타입
    * Kotlin 문법상으로는 T! 라고 표현되며 실제 코드에서는 T!가 표기되지는 않지만 컴파일러가 내부적으로 구분해ㅓ서 다룸

- 왜 플랫폼 타입이 생겼는가?
  + Kotlin & Java 는 100% 상호운용을 목표로 하고 있지만 Java 는 @Nullable / @NotNull 같은 어노테이션이 없으면 타입에 null 가능 여부 정보가 없다
  + Kotlin 은 null 안정성을 엄격히 체크하지만 Java 에서 넘어오는 타입의 null 안정성 보장할 수 없음
  + 개발자에게 책임을 맡긴 플랫폼 타입을 도입

- 샘플코드
  + ````java
    public class User {
      public String getName() {
        return null; // 실수로 null 리턴해도 java 는 컴파일 오류 없음
      }

      @NotNull
      public String getNameAnnotation() { return "John"; }

      @Nullable
      public String getNicknameAnnotation() { return null; }
    }
    ````
  + ```kotlin
    val user = User()
    val name = user.name // platform type: String! -> null 체크 강제 안됨
    println(name.length) // NPE 발생 가능 (코틀린이 강제 체크 못함)

    val nameAnnotation: String = user.getNameAnnotation()    // NotNull → String
    val nicknameAnnotation: String? = user.getNicknameAnnotation() // Nullable → String?
    ```
  + user.name 의 타입은 String! 으로 플랫폼 타입으로 처리되며 null이 올 수도 있다는 사실을 코틀린이 보류하고 있음

- 자주 쓰는 안전 패턴
  + | 상황                          | 추천 방식                                |
    | --------------------------- | ------------------------------------ |
    | 의심되는 플랫폼 타입 호출              | `val value = user.name ?: "default"` |
    | 널일 경우 처리 분기 필요              | `user.name?.let { use(it) }`         |
    | 확신이 있으면 강제 unwrap (`!!`) 사용 | `val name = user.name!!` (권장하지 않음)   |
    | 자바 코드 수정 가능                 | `@Nullable` / `@NotNull` 붙이기         |
 
- 결론
  + 플랫폼 타입은 코틀린과 자바를 연결하기 위한 타협
  + 코틀린이 null 안정성을 유지하면서도 Java 코드를 그대로 쓸 수 있도록 
    * Nullable 여부가 불확실할 때는 플랫폼 타입으로 처리
    * 개발자가 명확하게 null 체크 직접 해줘야 함
    * Java 코드에서 annotation 으로 명확한 정보 표시

- 면접 관련 질문
  + 플랫폼 타입이 Kotlin에 왜 필요한가요?
    * Java 타입 시스템에는 null 안정성 정보가 없기 때문에 코틀린이 자바의 타입을 바로 단정할 수 없음
    그래서 T! 형태의 플랫폼 타입으로 두고 개발자가 책임지고 null 처리 하도록 한 것
  + 플랫폼 타입을 방치하면 어떤 문제가 발생할 수 있나요?
    * 플랫폼 타입은 컴파일러가 null 검사 강제를 하기 않기 때문에 Java 코드에서 null 넘어와도 코틀린
    쪽에서 NPE(NullPointerException) 터질 수 있음. 따라서 플랫폼 타입을 사용할 때는 반드시 null 검증 습관 필요
  + 플랫폼 타입을 null-safe 하게 다루는 방법이 있을까요?
    * safe call(?.), elvis 연산자(?:), 명시적인 null 체크(if value != null)
    * 코틀린 측에서 직접 안전하게 다루거나 자바 코드에 어노테이션(@Nullable, @NotNull) 추가해서
    코틀린이 플랫폼 타입을 올바르게 추론하게 해야합니다.


---
