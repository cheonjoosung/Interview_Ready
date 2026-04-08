### Atomic Class
- 개념 및 정의
  + Java 멀티 쓰레드 환경에서도 안전하게 값을 조작할 수 있도록 Lock-Free(락 없는) 원자성 보장 도구
  + `java.util.concurrent.atomic` 패키지에서 제공
  + 원자성은 하나의 연산지 중간에 끊기거나 다른 스레드에 의해 방해받지 않고 한 번에 실행되는 성질

- 왜 필요한가?
  + | 문제       | 설명                                                 |
    | -------- | -------------------------------------------------- |
    | 멀티스레드 환경 | 여러 스레드가 같은 변수 접근 시 **경합 조건(Race Condition)** 발생 가능 |
    | 동기화 비용   | `synchronized` → 락 성능 비용 발생                        |
    | 대안 필요    | 빠르고 안전한 **원자적 연산 제공 (CAS 기반)**                     |
  + Atomic 클래스는 락 없이 경량화된 동시성 제어 도구

- 주요 클래스와 특징
  + | 클래스                          | 용도                      |
    | ---------------------------- | ----------------------- |
    | `AtomicInteger`              | int 원자 연산               |
    | `AtomicLong`                 | long 원자 연산              |
    | `AtomicBoolean`              | boolean 원자 연산           |
    | `AtomicReference<T>`         | 객체 참조 원자 교체             |
    | `AtomicStampedReference<T>`  | 객체 + 버전 스탬프 (ABA 문제 방지) |
    | `AtomicMarkableReference<T>` | 참조와 Boolean 마크 결합       |
  + 공통 특징
    * 내부적으로 CAS (Compare And Swap) 기반
    * `get()`, `set()`, `incrementAndGet()`, `compareAndSet()` 제공
    * 락 없이 스레드 안전

- 예시
  + ```java
    import java.util.concurrent.atomic.AtomicInteger;

    public class AtomicExample {
        private static final AtomicInteger counter = new AtomicInteger(0);

        public static void main(String[] args) {
            counter.incrementAndGet(); // +1
            System.out.println(counter.get()); // 1

            counter.addAndGet(5); // +5
            System.out.println(counter.get()); // 6

            boolean success = counter.compareAndSet(6, 10); // CAS
            System.out.println(success ? "변경 성공" : "실패");
            System.out.println(counter.get()); // 10
        }
    }
    ```

- 주요 메서드
  + | 메서드                             | 설명                |
    | ------------------------------- | ----------------- |
    | `get()`                         | 현재 값 반환           |
    | `set(value)`                    | 값 설정              |
    | `incrementAndGet()`             | +1 후 반환           |
    | `addAndGet(n)`                  | +n 후 반환           |
    | `compareAndSet(expect, update)` | 예상값 일치 시 변경 (CAS) |

- 주의점
  + | 주의사항                 | 설명                                   |
    | -------------------- | ------------------------------------ |
    | 복합 연산 주의 (get → set) | 복합 로직이면 여전히 동기화 필요                   |
    | 데이터 구조 변경 시 한계       | Atomic은 **변수 단위** 동시성만 해결            |
    | 고성능 동시성 필요 시         | `LongAdder`, `LongAccumulator` 사용 고려 |

- 면접 관련 질문
  + Atomic 클래스는 어떻게 스레드 안전 보장
    * CAS(Compare And Swap)를 이용해 다른 스레드의 변경 여부를 검사하고 안전하게 값을 업데이트
    * 락 없이 스레드 안전한 연산을 제공하므로 성능 우수
  + Atomic 클래스와 synchronized 차이
    * `synchronized` 락을 걸어 코드 블록 보호하고 `Atomic`은 변수 단위로 원자성(CAS)만 보장
    * Atomic 기 가볍고 빠르지만 복한 연산에는 부적합
  + AtomicReference 언제 사용?
    * `AtomicReference`는 객체 참조를 원자적으로 교체해야 할 때 사용
    * 멀티스레드 환경에서 캐시 교체, 상태 전이 관리 등에 활용


---
