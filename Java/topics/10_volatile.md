### volatile
- 개념
  + volatile 변수는 메인 메모리와 스레드 캐시 간의 동기화를 보장
  + 변수의 가시성(visibility) 은 보장하지만, 원자성(atomicity) 은 보장하지 않음
  + JMM (Java Memory Model) 에서 happens-before 관계 보장

- 필요한 이유?
  + Java에서 각 스레드는 자신만의 캐시(레지스터/CPU 캐시) 에 변수를 복사해 사용
  + → 메인 메모리 값과 불일치 발생 가능 (가시성 문제)
  + ```java
    class Example {
      private static boolean running = true;

      public static void main(String[] args) throws Exception {
        new Thread(() -> {
            while (running) { } // running 값이 캐시에 갇히면 무한 루프
        }).start();

        Thread.sleep(1000);
        running = false; // 메인 스레드에서 종료 신호
      }
    }
    ```
  + volatile 로 선언하면 무한 루프 정상 종료됨

- 언제 쓰면 좋을까?
  + 상태 플래그, 완료 여부, stop 신호 같은 단순 boolean 변수
  + Double-checked locking 패턴 (Singleton)
  + ```java
    class Singleton {
      private static volatile Singleton instance;

      private Singleton() {}

      public static Singleton getInstance() {
        if (instance == null) {
          synchronized (Singleton.class) {
            if (instance == null) {
              instance = new Singleton(); // 안전하게 보장
              }
          }
        }
        return instance;
      }
    }
    ```

- ❗주의할 점
  + 원자성 보장 X
  + ```java
    private volatile int count = 0;
    public void increment() {
        count++;
    }
    ```
  + 해결책 
    * synchronized 사용
    * AtomicInteger 등 java.util.concurrent.atomic 패키지 사용
    * ```java
      private AtomicInteger count = new AtomicInteger();

      public void increment() {
        count.incrementAndGet(); // 원자적 연산
      }
      ```
  
- 면접 관련 질문
  + volatile과 synchronized의 차이?
    * volatile → 가시성 보장 (최신 값 공유)
    * synchronized → 가시성 + 원자성 + mutual exclusion(상호 배제)
    * 단순 flag → volatile
    * 복잡한 연산 → synchronized or Atomic 클래스
  + volatile은 원자성을 보장하나요?
    * ❌ NO. count++ 같은 복합 연산은 여전히 경쟁 조건 발생
    * → AtomicInteger 같은 CAS 기반 원자 연산 필요
  + volatile을 사용하는 대표 사례는?
    * boolean running 같은 플래그 변수
    * Double-checked locking Singleton 패턴


---
