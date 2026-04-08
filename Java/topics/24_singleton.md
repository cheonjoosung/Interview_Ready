### 싱글턴 (Singleton)
- 개념 및 정의
  + 객체를 단 하나만 생성하도록 보장하는 디자인 패턴
  + 동일 인스턴스를 전역적으로 공유 → 상태 일관성 + 리소스 절약

- 구현 방식별 정리
  + | 방식                               | 설명                          | Thread-safe | 단점                   |
    | -------------------------------- | --------------------------- | ----------- | -------------------- |
    | **Eager Initialization** (고전 방식) | 클래스 로딩 시 인스턴스 생성            | ✅ (JVM 보장)  | 필요 없을 때도 생성 (메모리 낭비) |
    | **Lazy Initialization**          | getInstance() 호출 시 생성       | ❌           | 멀티스레드에서 동시 접근 문제     |
    | **synchronized**                 | 메서드 전체 동기화                  | ✅           | 호출 성능 저하 (락 오버헤드)    |
    | **Double-Checked Locking**       | synchronized 최소화 + volatile | ✅           | 코드 복잡                |
    | **Static Inner Class**           | JVM 클래스 로딩 시점 보장            | ✅✅          | 없음 (가장 많이 추천)        |
    | **Enum Singleton**               | enum이 자동으로 싱글턴 보장           | ✅✅✅         | 상속 불가, Enum 특성 제약    |
 
- 기본 구현 방법
  + ```java
    public class Singleton {
      // 1. 클래스 내부에 static으로 유일한 인스턴스 생성
      private static Singleton instance = new Singleton();
      
      // 2. 외부에서 new로 생성하지 못하도록 생성자 private
      private singleton() {}

      // 3. 외부에서 인스턴스에 접근하는 static 메서드 제공
      public static Singleton getInstance() {
          return instance();
      } 
    }
    ```
  + ```java
    Singleton s1 = Singleton.getInstance();
    Singleton s2 = Singleton.getInstance();
  
    System.out.println(s1 == s2);
    ```
  
- 지연 초기화 (Lazy Initialization)
  + 실제로 필요할 때 생성하는 방식 (메모리 절약 가능)
    * ```java
      public class Singleton {
          private static Singleton instance();

          private Singleton() {}

          public static Singleton getInstance() {
              if (instance == null) {
                instance = new Singleton();
              }
              return instance();
          }
      }
      ```
    * 이 방식은 **멀티스레드 환경에서 동시 접근 시 문제가 생길 수 있음** (동기화 필요)
  
- Thread-safe Singleton (멀티스레드 안전)
  + sychronized 사용
  + ```java
    public class Singleton {
        private static Singleton instance;

        private Singleton() {}

        public static sychronized Singleton getInstance() {
            if (instance == null) 
                instance = new Singleton();
          
            return instance;
        }
    }
    ```
  + 단점 : sychronized는 성능에 영향을 줄 수 있음

- Double-checking locking (권장)
  + ```java
    public class Singleton {
      private static volatile Singleton instance;
  
      private Singleton() {}

      public static Singleton getInstance() {
          if (instance == null) {
              synchronized (Singleton.class) {
                  if (instance == null)
                      instance = new Singleton();
              }
          }

          return instance;
      }
    }
    ```
  + 단점 : volatile과 synchronized를 함께 써서 성능 + 스레드 안전을 모두 고려

- 가장 안전한 방법: 정적 내부 클래스 방식 (권장)
  + ```java
        public class Singleton {
            private Singleton() {}
   
            private static class Holder {
                private static final Singleton INSTANCE = new Singleton();
            }
   
            public static Singleton getInstance() {
                return Holder.INSTANCE;
            }
        }
    ```
  + 클래스가 로딩될 때까지 인스턴스 생성하지 않음 -> **Lazy**
  + JVM 클래스 로더의 **Thread-safe 보장**

- enum 싱글턴
  + ```java
        public enum Singleton {
            INSTANCE;
   
            public void doSomething() {
                System.out.println("작동 중");
            }
        }    
    ```
  + **직렬화, 리플렉션, 멀티스레드 모두 안전**
  + 단, **상속이 불가하고** 필요 이상의 제약이 있을 수 있음

- 직렬화 & 리플렉션 이슈
  + 직렬화 문제 -> 직렬화 후 역질려화하면 새로운 객체가 생성 됨
    * ```java
      class Singleton implements Serializable {
      private static final Singleton INSTANCE = new Singleton();
      private Singleton() {}
      public static Singleton getInstance() { return INSTANCE; }
      }
      ```
    * ✅ 해결방법 -> readResolve() 추가
    * ```java
      protected Object readResolve() {
        return INSTANCE;
      }
      ```  
  + 리플렉션 문제
    * Constructor.setAccessible(true) 로 private 생성자 호출 가능 → 여러 객체 생성됨
    * ✅ 해결 방법: 생성자 안에서 이미 인스턴스가 존재하면 예외 던지기
    * Enum Singleton은 리플렉션으로도 깨지지 않음 → 가장 견고 

- 실무 사용 예시
  + 실무 사용 예시
  + Logger (ex. SLF4J) → 하나의 인스턴스로 관리
  + Configuration Manager → 앱 전역 설정 공유
  + Connection Pool → DB 연결 자원 효율적 관리
  + Spring Bean (기본 Scope) → 사실상 싱글턴(컨테이너 관리형)

- 단점
  + 테스트 어려움 -> 상태가 공유되므로 테스트 간 격리 힘듦 (mocking 어려움)
  + 의존성 강함 (높은 결합도) -> 전역 접근이 쉬워지면서 무분별하게 사용되면 유지보수 어려움
  + 멀티스레드 동기화 이슈 -> 잘못 구현 시 Race Condition 발생
  + 라이프사이클 제어 어려움 -> 필요 이상 오래 살아남아 메모리 점유 가능 

- 면접 관련 질문
  + Sington 패턴이란 무엇인가요?
    * **하나의 클래스에 단 하나의 인스턴스만 존재하도록 보장하는 디자인 패턴**입니다.
    * 주로 전역 상태 관리, 설정 객체, 로그 처리 등에서 사용합니다.
  + Thread-safe Singleton은 어떻게 구현하나요?
    * synchronized, Double-Checked Locking, 정적 내부 클래스 (static inner class), enum을 사용할 수 있습니다.
    * **가장 안전하고 성능도 좋은 방법은 정적 내부 클래스 방식입니다.**
  + Singleton의 단점은 무엇인가요?
    * 테스트 어려움 -> 상태가 공유되므로 유닛 테스트 시 고립시키기 힘듦
    * 상태 공유 문제 -> 여러 스레드가 같은 인스턴스에 접근하면 동기화 필요
    * 높은 결합도 -> 클래스 간 의존성이 강해지고 유연성 저하


---
