### 싱글턴 패턴 및 동기화
- 기본 개념
  + object 키워드를 사용하면 **자동으로 싱글턴 객체**가 생성
  + 예시
    * ```kotlin
      object Logger {
        fun log (msg: String) {
          println("Log : $msg")
        }
      }

      Logger.log("Hello")
      ```
  + object는 클래스처럼 정의되지만 단 **1개의 인스턴스만 생성됨**
  + 별도 생성자 필요 없음 (new 사용 불가)
  + Java의 enum Singleton 처럼 안전하게 단일 객체 보장

- Kotli이 object만으로 싱글턴이 가능한 이유
  + Kotlin의 object는 내부적으로 JVM에서 static 필드로 변환되어 로딩됩니다.
  + 클래스가 처음 로딩될 때 static 필드가 초기화되므로, **객체는 1회만 생성**되고 이 과정이 **동기화되어 있어 thread-safe** 합니다.

- Thread-safe 한가?
  + 기본적으로 object는 Thread-safe 함.
  + Kotlin의 object는 JVM 클래스 로딩 규칙에 따라 안전하게 초기화 되므로 **멀티스레드 환경에서 절대 2번 생성되지 않는다.**
  + 모두 안전
    * ```kotlin
      object DatabaseManager {
        fun connect() = println("DB 연결")
      }    
      ```
   
- 상태를 가질 경우 주의할 점
  + object는 싱글턴이기 때문에 내부에 var 같은 **mutable state (변경 가능한 값)**를 두면 
     여러 스레드에서 동시 접근할 경우 **경쟁 상태 (Race condition)**가 발생할 수 있음
  + ```kotlin
    object Counter {
      var count = 0
   
      fun increment() {
        count++
        }
      }
    ```
   - 멀티스레드 환경에서는 count++가 Thread-safe 하지 않음 -> **동기화 필요**

- 동기화 (synchronized) 방법
  + @Synchronized 사용 (JVM 에서 제공)
    * ```kotlin
      object SafeCounter {
        var count = 0
   
        @Synchronized
        fun increament() {
          count++
          }            
        }
     ```
    * Jvm의 Synchronized 블록과 동일하게 작동
    * 단점 : 성능 저하 가능성
  + AtomicInteger 등 동시성 클래스 사용
    * ```kotlin
      import java.util.concurrent.atomic.AtomicInteger
     
      object AtomicCounter {
        private val count = AtomicInteger(0)
     
        fun increment() {
          count.incrementAndGet()
        }
     
        fun get(): Int = count.get()
      }
      ```
    * AtomicInteger는 **CAS(Compare-And-Set) 기반**으로 동기화를 처리
    * 빠르고 가벼운 방식 -> **멀티스레드 환경에 권장**

- Lazy Singleton 만드는 방법 (지연 초기화 된 싱글턴)
  + ```kotlin
    class MyService private constructor() {
      companion object {
        val instance : MyService by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
          MyService()
        }
      }
    }
    ```
  + by lazy는 기본적으로 Thread-safe인데, lazy(LazyThreadSafetyMode.NONE) 등으로도 설정 가능
  + option
    * SYNCHRONIZED (기본값): 멀티스레드 환경에서도 안전
    * PUBLICATION: 여러 번 초기화 가능성 있으나 가장 최신 객체를 사용
    * NONE: 동기화 없음 (단일 스레드일 때만 사용)* 

- Singleton 구현방식 정리
  + | 방식      | 코드          | Thread-safe   | 특징         |
    |---------|-------------|---------------|------------|
    | object  | object A {} |   ✅     | 가장 간단한 싱글턴 |
    | by lazy |  val x by lazy {}   |    ✅    | 지연 초기화     |
    |  @Synchronized     |   메서드에 어노테이션    |   ✅     |   간단하지만 느림         |
    |  AtomicInteger     |    동시성 클래스     |      ✅     |  빠르고 안정적     |

- 요약
  + | 목적          | 추천 방식                              | 설명                         |
    | ----------- | ---------------------------------- | -------------------------- |
    | 간단한 싱글턴     | `object`                           | 가장 직관적이며 기본적으로 thread-safe |
    | 지연 초기화      | `by lazy`                          | 필요 시 초기화, thread-safe      |
    | 상태 포함 + 동기화 | `@Synchronized` or `AtomicInteger` | 동기화 필요, 성능 차이에 따라 선택       |
 

- 면접 질문
  + kotlin object는 왜 thread-safe 하나요?
    * object는 JVM의 class loader에 의해 최초 1회만 초기화되므로, 여러 스레드가 동시에 접근해도 인스턴스가 중복 생성되지 않습니다.
    * Thread-safe가 기본으로 보장됩니다.
  + Kotlin에서 상태(state)를 가지는 싱글턴은 문제가 없나요?
    * 싱글턴 자체는 Thread-safe지만, 내부에 var처럼 mutable state가 있을 경우 Race Condition이 발생할 수 있습니다.
    * 따라서 동기화가 필요하거나, AtomicInteger 등의 동시성 도구를 사용해야 안전합니다.
  + @Synchronized 키워드는 어떤 역할을 하나요?
    * Kotlin에서 @Synchronized는 Java의 synchronized와 동일하게 작동하며, 해당 메서드를 동시에 하나의 스레드만 실행할 수 있게 합니다.
    * 단점은 성능 저하가 발생할 수 있습니다.


---
