### object 클래스
- 정의
  + Java의 모든 클래스가 암묵적으로 상속받는 최상위 클래스
  + extends Object는 자동으로 포함됨
  + ```java
    class Person {
        // 컴파일 시 자동으로 extends Object가 포함
    }   
    ```

- 사용 이유
  + 모든 객체를 동일하게 다루기 위함
    * → 다형성(polymorphism) 기반: 모든 객체를 Object 타입으로 처리 가능
  + 공통 기능 제공
    * equals(), hashCode(), toString() 같은 메서드 기본 제공 

- 주요 메드 정리
  + | 메서드                             | 설명                   | 비고                                  |
    | ------------------------------- | -------------------- | ----------------------------------- |
    | `equals(Object obj)`            | 객체의 동등성 비교           | 기본은 `==` (참조 비교), 필요 시 오버라이딩        |
    | `hashCode()`                    | 객체 해시값 반환            | 해시 기반 컬렉션(`HashMap`, `HashSet`)에 필요 |
    | `toString()`                    | 객체 문자열 표현            | 기본: "클래스명@해시코드"                     |
    | `getClass()`                    | 런타임 클래스 정보 반환        | Reflection에 자주 사용                   |
    | `clone()`                       | 객체 복제 (얕은 복사)        | `Cloneable` 인터페이스 필요                |
    | `finalize()`                    | GC 직전 호출(Deprecated) | 자바 9 이후 사용 X                        |
    | `wait(), notify(), notifyAll()` | 스레드 동기화              | 동기화된 블록 내에서만 호출 가능                  |
 
- eqauls() & hashCode() 규약
  + ```java
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return this.name.equals(other.name);
    }

    @Override
    public int hashCode() {
        return name.hashCode();  // equals와 일관성 유지
    }
    ```
  + equals()가 true라면 → hashCode()는 반드시 동일해야 함
  + hashCode()가 같다고 해서 반드시 equals()가 true인 것은 아님 

- 사용 예시
  + ```java
    Person p1 = new Person("Bae");
    Person p2 = new Person("Bae");

    System.out.println(p1 == p2);           // false (주소 비교)
    System.out.println(p1.equals(p2));      // true (논리적 비교)
    System.out.println(p1.toString());      // "Person[name=Bae]"
    ```
   
- Object 클래스와 인터페이스 비교
  + | 항목    | Object 클래스      | 인터페이스       |
    | ----- | --------------- | ----------- |
    | 역할    | 모든 클래스의 최상위 구현체 | 행동 명세서 (규약) |
    | 상속    | 단일 상속만 가능       | 다중 구현 가능    |
    | 자동 포함 | 모든 클래스에 자동 상속   | 명시적 구현 필요   |

- 면접 관련 질문
  + == vs equals() 차이?
    * == → 참조 비교 (주소값)
    * equals() → 논리적 동등성 비교 (값), 필요시 오버라이딩
  + equals()와 hashCode()를 왜 같이 오버라이딩해야 할까?
    * HashMap, HashSet 같은 해시 기반 컬렉션에서
      1. hashCode() → 객체의 버킷 위치 결정
      2. equals() → 버킷 내에서 실제 동등성 비교
    * 둘 중 하나만 오버라이딩하면 데이터 불일치 발생 가능
  + toString()을 오버라이딩하는 이유?
    * 디버깅, 로깅 시 사람이 읽기 좋은 문자열 제공
    * 기본 구현은 클래스명@해시코드라 가독성이 떨어짐
  + finalize()는 왜 Deprecated 되었나?
    * GC 실행 시점 불확실, 성능 저하/리소스 누수 문제
    * 대안: try-with-resources 또는 Cleaner API 사용 


---
