### 가비지 컬렉션
- 개념
  + 가비지 컬렉션은 프로그램이 더 이상 사용하지 않는 객체를 자동으로 탐지하고 메모리에서 회수하여
  재사용 가능헤가 만드는 JVM의 자동 메모리 관리 기능이다.
  + 개발자가 free() 같은 명시적 해제를 하지 않아도 JVM이 힙(HEAP) 에 있는 객체를 관리함

- 원리
  + JVM 은 힙 메모리를 여러 영역으로 구분하고 객체의 생존 기간에 따라 다르게 관리
  + 참조 그래프(Reachability Graph)를 탐색하여 더 이상 참조되지 않는 객체를 unreachable 로 판단하고 제거
  + stop-the-world 방식으로 잠시 애플리케이션 실행을 멈춘 뒤 동작하는 방식이 일반적

- 영역
  + 메모리 구조
    * ![gc_memroy](../_assets/gc_memory.png)
  + 영역
    * ![gc_generation](../_assets/gc_generation.png)
    * Young Generation
      1. 새롭게 생성된 객체가 배치되는 공간
      2. 대부분의 객체는 이곳에서 생성되고 곧바로 소멸
    * Old Generation
      1. Young 영역에서 일정 횟수 이상 살아남은 장수 객체가 옮겨지는 공간
      2. 더 오래 유지되는 객체를 보관
    * Permanent Generation
      1. 클래스 메타데이터(클래스 정의, 메서드 ,상수 등)를 저장
      2. 자바 8 이후에는 Metaspace 로 대체

- GC 종류
  + Minor GC
    * Young Generation 에서 수행되는 가비지 컬렉션
    * 속도 빠르며 자주 발생
    * 작은 영역에서 단기 객체를 신속하게 수거
  + Major GC
    * Old Generation 포함하여 전체 힙 영역을 대상으로 하는 컬렉션
    * 더 느리고 애플리케이션 정지 시간이 길어짐
    * 메모리가 부족할 때 주로 발생

- 결론
  + JVM 의 가비지 컬렉션은 메모리 자동 관리를 가능하게 해주지만, Minor/Major GC,
  Young/Old/Permanent 같은 개념을 이해해야 불필요한 Full GC, 메모리 누수 등 피하면서 효율적 튜닝 가능

- 면접 관련 질문
  + JVM에서 Young Generation과 Old Generation의 차이는 뭔가요?
    * Young Generation은 새롭게 만들어진 객체가 저장되는 영역으로,
      짧은 생존 주기를 가진 객체를 빠르게 수거하기 위해 Minor GC가 자주 일어납니다.
      Old Generation은 Young에서 살아남은 장수 객체가 저장되는 공간으로,
      Major GC가 일어나 전체 힙을 대상으로 수거할 때 포함됩니다.
  + Major GC가 자주 발생하면 어떤 문제가 있을까요?
    * Major GC는 힙 전체(Old 영역 포함)를 정리하기 때문에
      stop-the-world 시간이 길어지고, 사용자 경험에 영향을 주거나 성능 저하가 발생할 수 있습니다.
      따라서 힙 튜닝이나 객체 생명주기 관리를 통해 Major GC 빈도를 줄이는 것이 중요합니다.
  + 자바 8부터 PermGen이 사라진 이유는 무엇인가요?
    * PermGen은 클래스 메타데이터 크기를 고정된 영역에 저장했는데,
      메타데이터가 많아지면 OutOfMemoryError를 일으키기 쉬웠습니다.
      그래서 자바 8부터는 PermGen 대신 Metaspace를 도입해
      메타데이터를 네이티브 메모리에 저장하고, 크기를 유연하게 관리할 수 있게 개선했습니다.


---
