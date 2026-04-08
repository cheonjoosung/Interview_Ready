### Thread 쓰레드
- 개념
  + 쓰레드 : 프로세스 내부에서 실행되는 작업 단위.
  + 프로세스 :  실행중인 프로그램 단위. 자체 메모리 공간을 가짐
  + 하나의 프로세스는 최소 1개의 쓰레드를 가지며, 여러 쓰레드가 메모리 공간을 공유하면서 동작
  + | 항목     | 프로세스 (Process)                          | 쓰레드 (Thread)                       |
    | ------ | --------------------------------------- | ---------------------------------- |
    | 정의     | 실행 중인 프로그램                              | 프로세스 내 실행 단위                       |
    | 메모리 구조 | 고유한 메모리 공간 (Heap, Stack, Data, Code 분리) | Heap, Code, Data 공유, **Stack은 개별** |
    | 속도/성능  | 생성·전환 비용 큼 (**Heavyweight**)            | 전환 빠름 (**Lightweight**)            |
    | 안정성    | 하나 죽어도 다른 프로세스 영향 없음                    | 하나 오류 → 전체 프로세스 영향 가능              |
    | 통신 방식  | IPC (Socket, Pipe 등) 필요                 | 메모리 공유로 직접 통신                      |
    | 예시     | 브라우저, 게임 프로세스                           | 브라우저 탭, 음악 재생/다운로드 쓰레드             |


- 쓰레드 생성 방법
  + 쓰레드 클래스 상속
  + ```java
    class MyThread extends Thread {
      @Override
      public void run() {
        System.out.println("Running in MyThread");
      }
    }

    MyThread t1 = new MyThread();
    t1.start();
    ```
  + Runnable 구현 (선호)
  + ```java
    class MyRunnable implements Runnable {
      @Override
      public void run() {
        System.out.println("Running in MyRunnable");
      }
    }

    Thread t2 = new Thread(new MyRunnable());
    t2.start();
    ```
  + 람다식 (Java 8 이상)
  + ```java
    Runnable r = () -> System.out.println("Running with lambda");
    new Thread(r).start();
    ```
  + Runnable 선호 이유
    * | 항목    | Thread 상속               | Runnable 구현               |
      | ----- | ----------------------- | ------------------------- |
      | 상속 제약 | 다른 클래스 상속 불가 (단일 상속 한계) | 다른 클래스 상속 가능              |
      | 결합도   | 실행 코드 + 쓰레드 실행 강하게 결합   | 실행 로직과 쓰레드 실행 분리          |
      | 재사용성  | 낮음                      | 높음                        |
      | 실무 활용 | 제한적                     | ExecutorService 등과 호환성 높음 |

- 쓰레드 생명주기
  + ```text
    NEW → RUNNABLE → RUNNING → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED
    ```
  + NEW: new Thread()로 생성, 아직 start() 안 함
  + RUNNABLE: 실행 대기 상태 (JVM 스케줄러가 실행 시간 배정)
  + RUNNING: CPU 점유 중
  + BLOCKED / WAITING / TIMED_WAITING: 자원·신호 대기
  + TERMINATED: 실행 종료
  + ```java
    Thread t = new Thread(() -> {});
    System.out.println(t.getState()); // NEW
    t.start();
    System.out.println(t.getState()); // RUNNABLE or RUNNING
    ```

- 동기화 관련 키워드
  + synchronized: 임계 구역 동기화
  + volatile: 변수 가시성 보장
  + ReentrantLock: 유연한 Lock 제어
  + Semaphore / Mutex: 자원 접근 제어
  + Deadlock: 서로 Lock 점유 대기 → 시스템 멈춤

- 면접 관련 질문
  + Runnable vs Thread 상속 차이는?
    * Runnable은 단일 상속 제약이 없고 실행 로직과 쓰레드 실행이 분리돼 재사용·유연성이 높아 실무에서 선호.
  + Thread의 상태 전이는 누가 관리하나요?
    * JVM이 관리하며, OS 스케줄러 정책(우선순위·시간 분할)에 따라 RUNNABLE 상태에서 실행이 선택됨.
  + 멀티스레드에서 가장 주의할 점은?
    * 공유 자원 접근 시 동기화 문제와 데드락 방지. 동기화 시 성능 저하도 고려.


---
