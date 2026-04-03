### object 키워드
- 정의
  + 클래스를 정의함과 동시에 인스턴스를 생성하는 키워드
  + 싱글톤 패턴을 간결하게 구현할 수 있게 해주며 코드 간결성 확보 및 Thread Safe
  + 최초 참조 시점에 단 한번만 인스턴스가 생성됨 (lazy + synchornized)
  + JVM class loader가 클래스 로딩을 Thread-safe하게 보장하기에 락 없이 안전
  
- 주요 사용 케이스
  + 싱글톤 (Singleton)
    * 하나만 존재하는 객체 정의
    * 생성자 호출 필요없고 Thread-safe 함
    * ```kotlin
      object Logger {
        fun log(msg: String) {
            println("Log: $msg")
        }
      }
      
      Logger.log("message") // 호출
      ```
  + 동반객체(Companion Object)
    * 클래스 내부에서 정적 멤버(static memeber)처럼 사용
    * @JvmField, @JvmStatic 붙이면 Java와의 상호운용성 을 더 쉽게 할 수 있다.
    * ```kotlin  
      class User(val name: String) {
          companion object {
              fun create(name: String): User = User(name)
              var country = "Korea" 

              @JvmStatic val BASE_ADDRESS = "서울특별시" // java static 처럼 
          }
      }
    
      User.create("Alice") // 클래스 이름으로 호출 가능
      User.BASE_ADDRESS // 서울특별시 java static 처럼 사용 가능
      ```
  + 익명객체
    * 일회성으로 인터페이스 또는 추상 클래스를 구현할 때 사용
    * Android 리스너 구현에 자주 사용
    ```kotlin
    val buttonClickListener = object: View.OnClickListener {    
        override fun onClick(v: View?) {
            println("Clicked!")
        }
    }
    ```
  + Object Declaration
    * 상속 구조나 전략 객체로 사용
    * 동일한 인터페이스를 구현한 여러 객체 중 하나를 선택해 실행 전략을 바꾸는 패턴
    * PaymentProcessor는 PaymentStrategy라는 상위 타입만 알고 있음
    * 실제 객체는 KakaoPay, CreditCard 등 다양하게 대체 가능 → 런타임 다형성
    * 조건에 따라 전략을 바꾸거나, 클라이언트 코드가 객체의 구체 타입을 몰라도 사용 가능
    * 전략 객체는 런타임에 교체 가능하며, 객체 간의 의존성 분리 + 다형성 확보가 가능
    ```kotlin
    interface PaymentStrategy {
        fun pay(amount: Int)
    }

    object KakaoPay : PaymentStrategy {
        override fun pay(amount: Int) {
            println("카카오페이로 $amount 원 결제")
        }
    }

    object CreditCard : PaymentStrategy {
        override fun pay(amount: Int) {
            println("신용카드로 $amount 원 결제")
        }
    }

    class PaymentProcessor(var strategy: PaymentStrategy) {
        fun processPayment(amount: Int) {
            strategy.pay(amount)
        }
    }

    fun main() {
        val processor = PaymentProcessor(KakaoPay)
        processor.processPayment(10_000)  // 카카오페이로 결제

        processor.strategy = CreditCard
        processor.processPayment(20_000)  // 신용카드로 결제
    }
    ```

- 기타
  + `object`는 생성자가 없으므로 생성자 매개변수는 가질 수 없다.
  + 내부적으로 `final`, `static`, `thread-safe singleton`으로 컴파일 됨

- 면접 예상 질문 & 답변
  + Q1. `object`는 어떻게 thread-safe한 Singleton을 보장하나요?
    * `object`는 JVM의 class loading 단계에서 한 번만 초기화되며 이 과정은 JVM 자체가 thread-safe 보장하므로 별도의 락이나 synchronized 키워드 없이 안전함. (Lazy initialization + Classloader Locking)
  + Q2. `object`와 `companion object`의 차이는 무엇인가요?
    * `object`는 클래스 외부에서 싱글톤 객체를 정의할 때 사용하며, `companion object`는 클래스 내부에서 정적 멤버처럼 동작하게 하기 위해 사용. 둘 다 싱글톤이지만 클래스 내부인지 외부인지의 차이가 있음
  + Q3. `object`와 Java의 `static` 키워드는 어떤 차이가 있나요?
    * Java의 `static` 클래스 로딩 시점에 메모리에 올라가는 반면 코틀린의 `object`는 최초 참조 시점에 lazy 하게 초기화 됨. 또한, Kotlin은 함수, 필드, 클래스 등에서 정적 멤버를 만들기 위해 `companion object` + `@JvmStatic` 활용


---
