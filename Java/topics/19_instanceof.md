### instanceof
- 개념
  + 객체가 특정 클래스 or 인터페이스 타입인지 런타임에 검사하는 연산자
  + 다형성 구조에서 업캐스팅된 객체를 다시 다운캐스팅할 때 안정성을 확보하는 용도

- 사용법
  + ```java
    public void typeCheck(Object obj) {
      if (obj instanceof String) {
          String s = (String) obj; // 안전하게 다운캐스팅
          System.out.println(s.length());
      }
    } 
    ```
  + null instanceof Something → 항상 false

- Java 최신 문법 (Pattern Matching for instanceof)
  + Java 14+: instanceof와 캐스팅을 동시에 처리 
  + ```java
    if (obj instanceof String s) {
      System.out.println(s.length()); // 자동 캐스팅
    }
    ```
  + 코틀린의 스마트 캐스트와 유사

- Kotlin (`is` 연산자)
  + ```java
    if (obj is String) {
      println(obj.length) // 스마트 캐스트 → 별도 캐스팅 불필요
    }
    ```
  + 장점: is 체크한 이후 블록 내부에서는 자동으로 해당 타입으로 취급
  + null인 경우 → 항상 false

- 다형성과 instanceof의 관계
  + 좋은 활용: 업캐스팅된 객체를 안전하게 특정 타입으로 다시 사용할 때
  + 나쁜 활용: instanceof로 타입 분기 코드 남발 → OCP(개방-폐쇄 원칙) 위반
  + 바람직한 대안: 다형성 오버라이딩 활용
  + ```java
    // ❌ 좋지 않은 예시
    if (animal instanceof Dog) { ((Dog) animal).bark(); }
    else if (animal instanceof Cat) { ((Cat) animal).meow(); }

    // ✅ 다형성을 활용한 방법
    animal.makeSound(); // 각 클래스가 override
    ```

- 비교 정리
  + | 항목      | Java `instanceof`         | Kotlin `is`                |
    | ------- | ------------------------- | -------------------------- |
    | 타입 검사   | `instanceof`              | `is`                       |
    | 캐스팅     | 수동 필요 (`(String) obj`)    | 스마트 캐스트 (자동)               |
    | 가독성     | 장황                        | 간결                         |
    | null 처리 | `null instanceof` → false | `obj is Something` → false |

- 면접 관련 질문
  + instanceof 언제 쓰면 좋을까?
    * 업캐스팅된 객체를 안전하게 다운캐스팅할 때
    * 런타임에만 알 수 있는 타입 로직이 필요할 때
  + 코틀린의 is 와 자바의 instanceof 차이점?
    * Java: 검사 후 명시적 캐스팅 필요 (Java 14 이후 일부 개선)
    * Kotlin: 스마트 캐스트로 자동 적용, 가독성/안전성 ↑
  + 왜 instanceof를 남용하면 안 되는가?
    * 타입 분기에 의존하는 코드가 늘어나 다형성의 장점을 잃음
    * SOLID 원칙 중 OCP 위반 


---
