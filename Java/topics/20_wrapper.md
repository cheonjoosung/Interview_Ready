### Wrapper 클래스
- 정의
  + 자바에서 기본형 타입(primitive type) 을 객체로 감싸서 참조형 타입처럼 다룰 수 있도록 만든 클래스
  + java.lang 패키지에 포함 → import 불필요

- 기본 타입 <-> Wrapper 클래스 매핑
  + | 기본 타입 (Primitive) | Wrapper 클래스 | 특징                  |
    | ----------------- | ----------- | ------------------- |
    | byte              | Byte        | Number 상속           |
    | short             | Short       | Number 상속           |
    | int               | Integer     | Number 상속, 가장 많이 사용 |
    | long              | Long        | Number 상속           |
    | float             | Float       | Number 상속           |
    | double            | Double      | Number 상속           |
    | char              | Character   | Number 상속 ❌         |
    | boolean           | Boolean     | Number 상속 ❌         |
 
- Wrapper 클래스가 필요한 이유
  + 컬렉션/제네릭
    * 컬렉션은 객체만 저장 가능 → 기본형은 못 넣음
    * List<int> ❌ → List<Integer> ✅
  + null 표현 가능
    * int i; → 무조건 값 가짐 (기본 0)
    * Integer i = null; → 값 없음 표현 가능
  + 유틸 메서드 제공
    * Integer.parseInt("123")
    * Double.isNaN(x)
  + 객체 지향적 프로그래밍 지원
    * 상수, 메서드, 상속 구조 (ex. Number)

- 박싱 & 언박싱
  + Boxing: 기본형 → Wrapper 객체
    * ```java
      Integer i = Integer.valueOf(10);
      ``` 
  + Unboxing: Wrapper 객체 → 기본형
    * ```java
      int j = i.intValue();
      ``` 

- 오토박싱 & 오토언박싱
  + 오토박싱 (자동 변환, JDK 1.5+)
    * ```java
      Integer i = 10; // int → Integer 자동 변환
      List<Integer> list = new ArrayList<>();
      list.add(5);    // int → Integer 자동 박싱
      ```
  + 오토언박싱
    * ```java
      Integer i = 20;
      int j = i;          // Integer → int
      int sum = i + 5;    // 자동 언박싱 후 연산
      ``` 

- 주요 특징
  + 불변(Immutable)
    * 한 번 생성된 Wrapper 객체의 값은 변경 불가
    * 변경하려면 새 객체 생성
  + 캐싱(Caching)
    * Boolean, Byte, Character(0127), Short(-128127), Integer(-128~127)
    * 동일 범위 값은 같은 객체 재사용 
    * ```java
      Integer a = 100;
      Integer b = 100;
      System.out.println(a == b); // true (캐싱됨)

      Integer x = 200;
      Integer y = 200;
      System.out.println(x == y); // false (새 객체)
      ```
  + 상속구조
    * 숫자형 Wrapper 클래스는 Number 상속
    * intValue(), doubleValue() 등 변환 메서드 제공 

- 요약
  + Wrapper = primitive → 객체 변환 클래스
  + 필요 이유: 컬렉션/제네릭/Null 표현/유틸 메서드
  + 특징: 불변 + 캐싱 범위 존재
  + 주의: == 대신 equals 사용, 오토박싱 성능 문제 주의 
    
- 면접 관련 질문
  + Wrapper 클래스는 왜 불변인가?
    * 값 안정성 보장, 스레드 안전성 ↑, 캐싱 최적화 가능
  + Integer 객체 비교에서 == vs equals 차이는?
    * ```java
      Integer a = 100;
      Integer b = 100;
      System.out.println(a == b);      // true (캐싱 범위)
      System.out.println(a.equals(b)); // true

      Integer x = 200;
      Integer y = 200;
      System.out.println(x == y);      // false (다른 객체)
      System.out.println(x.equals(y)); // true
      ``` 
    * == → 참조 비교
    * equals() → 값 비교
  + 오토박싱/언박싱 주의할 점은?
    * 반복문에서 불필요한 박싱/언박싱 발생 시 성능 저하 
    * ```java
      Long sum = 0L;
      for (long i = 0; i < 1000000L; i++) {
          sum += i; // sum = Long + long → 오토언박싱 + 오토박싱 매번 발생
      }
      ```
    * long sum = 0L; 으로 기본형 쓰는 게 성능상 더 낫다.


---
