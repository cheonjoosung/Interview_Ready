### synchronized
- 개념 및 정의
  + 동기화 키워드: 멀티스레드 환경에서 공유 자원을 안전하게 접근하도록 보장
  + 모니터 락(monitor lock) 기반 → 객체마다 1개씩 모니터 존재
  + 상호 배제(Mutual Exclusion) → 한 시점에 하나의 스레드만 진입 가능
  + 가시성(Visibility) → synchronized 블록 안에서의 변수 변경은 메인 메모리에 즉시 반영되고, 블록 진입 시 다시 읽어옴 (→ volatile 효과 포함)

- 동작 원리 (JVM 관점)1. 기본 개념
  + synchronized가 보장하는 것:
    * Mutual Exclusion
      1. 한 스레드가 monitor를 획득하면 다른 스레드는 대기
    * Happens-before 관계 : 
      1. 락을 해제(unlock) 한 시점 → 이후 같은 락을 획득(lock) 하는 스레드가 변경된 값을 반드시 볼 수 있음
      2. 즉, 쓰기(write) → unlock → lock → 읽기(read) 순서 보장
    * Reentrancy (재진입성)
      1. 같은 스레드가 이미 획득한 락은 다시 획득 가능 (호출 중첩 시 deadlock 방지)

- 사용 패턴
  + 인스턴스 메서드
    * ```java
      public synchronized void increment() { count++; }
      // this 객체 모니터 사용
      ```
  + 블록 동기화
    * ```java
      synchronized (lock) {
        count++;
      }
      // 원하는 락 객체 지정 가능(성능 최적화)
      ```
  + static synchronized
    * ```java
      public static synchronized void increment() { value++; }
      // 클래스 객체 (Class<T>) 모니터 사용 -> 인스턴스 간에도 공유됨
      ```
    - 모니터 락(monitor lock)
        - 각 객체가 모니터(lock)를 하나씩 가지고 있음
        - 스레드는 synchronized가 선언된 영역에 들어가기 전 해당 객체의 모니터를 획득해야 하고, 끝나면 해제
    - 상호 배제(mutual exclusion)
        - 한 번에 단 하나의 스레드만 락을 획득하므로, 동기화된 영역 내부에서는 다른 스레드가 동시에 실행되지 않음

- 주의사항
  + 락 범위 최소화: 큰 범위 동기화는 성능 저하 → 꼭 필요한 코드만
  + Deadlock 위험: 여러 락을 교차 획득하는 구조 주의
  + Lock Contention: 경쟁이 심하면 대기 시간 증가
  + 원자성 보장 O, but 조건 동기화 직접 X

- 대안 (java.util.concurrent 패키지)
  + ReentrantLock: 공정성(fairness), 조건변수(Condition) 지원
  + ReadWriteLock: 읽기-쓰기 동시성 분리
  + StampedLock: optimistic read 등 고성능 시나리오 대응

- 면접 관련 질문
  + synchronized와 volatile 차이점?
    * volatile → 가시성만 보장 (원자성 ❌)
    * synchronized → 원자성 + 가시성 + happens-before 보장
    * 따라서 count++ 같은 복합 연산은 volatile로 해결 불가, synchronized 필요
  + synchronized와 ReentrantLock 차이점?
    * ReentrantLock은 tryLock(), 공정성 모드, 여러 Condition 지원
    * synchronized는 문법적으로 단순하고 GC 친화적 (명시적 unlock 필요 없음)
  + synchronized가 보장하는 것은?
    * 상호 배제 (Mutual Exclusion)
    * 메모리 가시성 (Visibility)
    * happens-before 관계
    

---
