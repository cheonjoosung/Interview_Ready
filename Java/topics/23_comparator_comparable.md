### Comparator와 Comparable
- 정의
  + Java에서 객체의 순서를 비교하고 정렬하는 데 사용되는 두 가지 주요 인터페이스는 **Comparable과 Comparator**입니다. 
  + 이 둘은 비슷한 목적을 가지지만, 사용 방식과 적용 상황에 차이가 있습니다.

- 공통점
  + 둘 다 객체 정렬 기준을 정의하는 인터페이스
  + 정렬 메서드(Collections.sort, Arrays.sort) 또는 정렬 컬렉션(TreeSet, TreeMap)에서 사용 

- Comparator<T> 인터페이스
  + 목적
    * 객체의 **"외부적인 정렬 기준"**을 정의합니다. 
    * 클래스 자체의 코드를 변경하지 않고, 다양한 방식으로 객체를 정렬하고 싶을 때 사용합니다.
  + 구현
    * 별도의 클래스를 만들어 Comparator<T> 인터페이스를 구현하고, compare(T o1, T o2) 메서드를 오버라이드합니다.
  + compare(T o1, T o2) 메서드
    * 두 개의 객체 o1과 o2를 비교합니다.
  + 반환 값
    * o1이 o2보다 작으면 (앞 순서) -> 음수
    * o1이 o2와 같으면 -> 0
    * o1이 o2보다 크면 (뒷 순서) -> 양수
  + 특징
    * 정렬 로직이 비교 대상 클래스 외부에 존재합니다.
    * 하나의 클래스에 대해 여러 개의 서로 다른 정렬 기준을 만들 수 있습니다.
    * java.util 패키지에 속합니다.
    * Collections.sort(list, comparatorInstance) 또는 Arrays.sort(array, comparatorInstance)와 같이 정렬 메서드에 Comparator 인스턴스를 전달하여 사용합니다.
    * 클래스 코드를 수정할 수 없는 경우(예: 서드파티 라이브러리 클래스)에도 정렬 기준을 제공할 수 있습니다.
    * Java 8부터는 람다 표현식이나 메서드 참조를 사용하여 Comparator를 더 간결하게 만들 수 있습니다.

- Comparable<T> 인터페이스
  + 목적
    * 객체에 **"자연스러운 순서(natural ordering)"**를 정의합니다. 
    * 해당 클래스의 객체들이 본질적으로 어떻게 정렬되어야 하는지에 대한 기본 규칙을 클래스 자체에 내장합니다.
  + 구현
    * 정렬 기능을 부여하고 싶은 클래스가 직접 Comparable<T> 인터페이스를 구현하고, compareTo(T o) 메서드를 오버라이드합니다.
  + compareTo(T o) 메서드
    * 현재 객체(this)와 매개변수로 전달된 객체 o를 비교합니다. 
  + 반환 값
    * 현재 객체가 o보다 작으면 (앞 순서) -> 음수
    * 현재 객체가 o와 같으면 -> 0
    * 현재 객체가 o보다 크면 (뒷 순서) -> 양수
  + 특징
    * 클래스 자체에 정렬 로직이 포함됩니다.
    * 하나의 클래스에는 하나의 compareTo() 메서드만 정의할 수 있으므로, 단일 정렬 기준만 제공합니다.
    * java.lang 패키지에 속합니다.
    * Collections.sort(list) 또는 Arrays.sort(array)와 같이 별도의 Comparator 없이 정렬 메서드를 호출하면, 요소들이 Comparable을 구현한 경우 compareTo() 메서드가 사용됩니다.
    * TreeSet, TreeMap과 같은 정렬된 컬렉션은 Comparable을 구현한 객체를 저장할 때 compareTo()를 사용하여 순서를 유지합니다.

- Comparator vs Comparable 정리
  + | 특징       | Comparable                     | Comparator                                   | 
    |----------|---------------------------------|----------------------------------------------|
    | 정의 위치    | 클래스 내부 (자연스러운 순서)        | 클래스 외부 (다양한 정렬 기준)                     | 
    | 메서드      | int compareTo(T o)             | int compare(T o1, T o2)                      | 
    | 구현 주체    | 정렬될 클래스가 직접 구현            | 별도의 Comparator 클래스 또는 람다식으로 구현        | 
    | 정렬 기준 수  | 하나의 정렬 기준만 제공             | 여러 개의 정렬 기준 제공 가능                      | 
    | 패키지      | java.lang                      | java.util                                    | 
    | 클래스 수정   | 원본 클래스 코드 수정 필요           | 원본 클래스 코드 수정 불필요                       | 
    | 사용 시점    | Collections.sort(list) (자동 사용)| Collections.sort(list, comparator) (명시적 전달)  |
        
- Java 8 이후 Comparator 개선
  + 람다식과 메서드 참조로 간결해짐
    * ```java
      list.sort(Comparator.comparing(Person::getName)); // 이름 오름차순
      list.sort(Comparator.comparingInt(Person::getAge).reversed()); // 나이 내림차순
      ```  
  + 복합 정렬 지원
    * ```java
      list.sort(
        Comparator.comparing(Person::getName)
                  .thenComparing(Person::getAge)
      );
      ``` 

- 면접 관련 질문
  + Comparable과 Comparator 차이는?
    * Comparable → 클래스 자체에 "자연스러운 순서" 정의
    * Comparator → 외부에서 다양한 정렬 기준 제공 가능
  + TreeSet이나 TreeMap은 어떤 걸 사용할까?
    * 기본적으로 Comparable 사용 (compareTo)
    * Comparator를 생성자에 넘겨주면 해당 기준으로 정렬 유지
  + 왜 Comparable은 하나만 정의 가능할까?
    * "자연 순서"는 본질적으로 하나만 존재해야 하기 때문
    * 여러 기준이 필요하다면 Comparator 사용
  + Comparator와 Comparable을 동시에 사용할 수 있을까?
    * 가능. 기본 정렬은 Comparable 기준을 따르고,
    * 필요에 따라 Comparator를 전달해 다른 기준으로도 정렬 가능 


---
