### Any 클래스
- 정의
  + `Any`는 Kotlin의 모든 클래스가 암묵적으로 상속하는 최상위 타입
  + Java의 `Ojbect`와 유사한 역할을 하지만, 더 간결하고 null 허용하지 않음
  + ```kotlin
    open class MyClass // 내부적으로 Any를 상속함
    ```

- 주요 특징
  + 모든 클래스의 루트
    * Kotlin의 모든 객체는 Any 타입으로 간주될 수 있음.
    * 따라서 어떤 타입이든 담을 수 있는 변수 선언 가능.
    * ```kotlin
      val v1: Any = "문자열"     // String
      val v2: Any = 123         // Int
      val v3: Any = true        // Boolean
      ```
  + 기본 제공 메서드
    * `Any`는 객체가 가져야 할 최소한의 메서드 3가지 포함 
    * | 메서드 이름                | 설명           |
      | --------------------- | ------------ |
      | `equals(other: Any?)` | 두 객체가 같은지 비교 |
      | `hashCode()`          | 해시값 반환       |
      | `toString()`          | 객체를 문자열로 변환  |
  + Nullable 버전 (Any?)
    * `Any`: null 담을 수 없음
    * `Any?`: null 포함 가능
    * ```kotlin
      val nonNull: Any = "값"  
      // nonNull = null    // ❌ 컴파일 오류

      var nullable: Any? = "값"
      nullable = null        // ✅ 가능
      ```
  + Java `Object`와의 관계
    * JVM에서는 Kotlin의 `Any`가 Java의 `java.lang.Object`로 컴파일됩니다.
    * Kotlin ↔ Java 상호 운용 시, Java의 Object는 Kotlin에서 `Any` 또는 `Any?`로 취급됩니다.
  
- 요약
  + Any: Kotlin의 모든 클래스의 부모
  + 기본 메서드 3개만 포함: equals, hashCode, toString
  + null 불가 → Any?로 사용 시 가능
  + Java와의 호환성을 고려해 Object와 대응됨

- 면접 관련 질문
  + `Any` 클래스 역할?
    * Kotlin 클래스 계층의 최상위 부모. 모든 객체의 루트 
  + `Any` null 담을 수 있나?
    *	❌ `Any`는 null 불가. null 허용하려면 `Any?` 사용 
  + Java의 Object와 차이점?
    * `Any`는 Java `Object`보다 가볍고, 기본 메서드만 포함. 동기화, clone 등은 없음 
  + `Any` vs `Unit` vs `Nothing` 
    * 모두 Kotlin 기본 타입이지만 역할이 다름 → 필요하면 비교표 제공 가능 
    

---
