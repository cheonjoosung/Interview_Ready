### Equals & hashCode()
- `Equals` 개념 및 정의
  + 객체의 동등성 비교에 사용되는 메서드
  + Object 클래스에 기본 구현 존재 → 기본 구현은 참조 동일성(==) 비교
  + 값 비교가 필요한 경우 오버라이드하여 논리적 동등성 비교 구현

- `Equals()` 규약`
  + | 규약          | 설명                                                                |
    | ----------- | ----------------------------------------------------------------- |
    | **반사성**     | `o.equals(o)`는 항상 `true`                                          |
    | **대칭성**     | `o1.equals(o2)`가 `true`면 `o2.equals(o1)`도 `true`                  |
    | **추이성**     | `o1.equals(o2)`와 `o2.equals(o3)`가 `true`면 `o1.equals(o3)`도 `true` |
    | **일관성**     | 비교에 사용되는 정보가 바뀌지 않는 한 결과는 항상 동일                                   |
    | **null 비교** | `o.equals(null)`은 항상 `false`                                      |

- `hashCode()` 개념
  + 객체를 식별하는 정수 해시 값 반환
  + HashMap, HashSet, Hashtable 등 해시 기반 컬렉션에서 객체 저장 위치(버킷) 결정
  + equals()와 함께 오버라이드해야 정상 동작 보장

- `hashCode()` 규약
  + | 규약                    | 설명                                          |
    | --------------------- | ------------------------------------------- |
    | **일관성**               | equals 비교 정보가 변하지 않으면 같은 실행 중에는 같은 해시 값 반환  |
    | **equals → hashCode** | `equals()`가 `true`면 `hashCode()`도 같아야 함     |
    | **hashCode 충돌 허용**    | `equals()`가 `false`여도 `hashCode()`는 같을 수 있음 |

- 재정의하지 않았을 때의 문제
  + ```java
    class Person {
      String name;
      int age;
      Person(String name, int age) {
          this.name = name;
          this.age = age;
      }
    }

    Person p1 = new Person("Alice", 30);
    Person p2 = new Person("Alice", 30);

    System.out.println(p1.equals(p2)); // false (기본 Object equals는 참조 비교)
    System.out.println(p1.hashCode()); // 예: 12345678
    System.out.println(p2.hashCode()); // 예: 87654321
    ```

- HashMap 문제 예시
  + `hashCode()`가 다르므로 p1과 p2는 다른 버킷에 저장됨
  + 논리적으로 같은 값이어도 HashMap에서는 찾을 수 없음
  + ```java
    HashMap<Person, String> map = new HashMap<>();
    map.put(p1, "정보 1");

    System.out.println(map.get(p2)); // null
    ```

- equals() & hashCode() 같이 재정의
  + ```java
    class Person {
      String name;
      int age;
      
      Person(String name, int age) {
          this.name = name;
          this.age = age;
      }
      
      @Override
      public boolean equals(Object o) {
          if (this == o) return true; // 참조 동일성
          if (!(o instanceof Person)) return false; // 타입 체크
          Person p = (Person) o;
          return age == p.age && name.equals(p.name); // 값 비교
      }
      
      @Override
      public int hashCode() {
          return Objects.hash(name, age); // Java 7+ 유틸
      }
    }
    ```

- equals()와 hashCode() 비교
  + | 항목      | equals()               | hashCode()                |
    | ------- | ---------------------- | ------------------------- |
    | 역할      | 값 비교(동등성)              | 해시 기반 컬렉션에서 버킷 결정         |
    | 기본 구현   | 참조 동일성 비교              | 객체 주소 기반 해시 값             |
    | 재정의 필요성 | 논리적 값 비교를 원할 때         | equals() 재정의 시 반드시 함께     |
    | 컬렉션 영향  | HashMap, HashSet 검색/저장 | HashMap, HashSet의 키 버킷 탐색 |

- 면접 관련 질문
  + equals()만 재정의하고 hashCode()를 재정의하지 않으면 어떤 문제가 생기나요?
    * HashMap, HashSet에서 같은 값의 객체를 찾아올 수 없음
    * hashCode()가 다르면 같은 객체로 인식되지 않고 다른 버킷에 저장됨
  + hashCode()만 재정의하고 equals()를 재정의하지 않으면?
    * 같은 해시 값의 객체를 같은 버킷에서 찾을 수는 있지만, equals 비교에서 false가 되어 동등성 확인에 실패
    * 결과적으로 검색/삭제 실패 가능
  + hashCode 충돌이 발생하면 어떻게 되나요?
    * 같은 버킷에 여러 객체가 저장되고, equals()로 최종 비교 수행
    * 충돌 자체는 허용되지만, equals() 구현이 올바르지 않으면 잘못된 비교 결과가 나올 수 있음


---
