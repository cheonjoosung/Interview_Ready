### 객체지향 설계의 5가지 원칙(SOLID)
- 개념
  + 객체지향 설계의 5가지 원칙인 SOLID는 소프트웨어 설계를 더 **이해하기** 쉽고, **유연하며**, **유지보수** 하기 쉽게 만드는 데 도움을 주는 지침 입니다.

- **SRP(Single Responsibility Principle)** : 단일 책임 원칙
  + 개념 : 한 클래스는 단 하나의 책임만 가져야 합니다.
  + 설명 : 클래스를 변경해야 하는 이유는 단 하나여야 한다는 의미 입니다. 
          클래스가 여러 책임을 가지게 되면, 
          한 책임의 변경이 다른 책임과 관련된 코드에 영향을 미칠 수 있어 코드 수정이 복잡해지고 오류 발생 가능성이 높아집니다.
  + 나쁜 예 : User 클래스가 사용자 정보관리, 사용자 인증, 이메일 전송 기능을 모두 가지고 있는 경우, 이메일 전송 로직 변경이 사용자 정보 관리 로직에 영향을 줄 수 있습니다.
    * ```java
      class User {
        void saveUser() { /* 사용자 저장 */ }
        void authenticate() { /* 인증 */ }
        void sendEmail() { /* 이메일 전송 */ }
      }
      ```
  + 좋은 예 : UserInfo(사용자 정보 관리), UserAuthenticator(사용자 인증), EmailService(이메일 전송) 클래스로 분리합니다.
    * ```java
      class UserRepository { void saveUser() {...} }
      class UserAuthenticator { void authenticate() {...} }
      class EmailService { void sendEmail() {...} }
      ```

- **OCP(Open-Closed Principle)** : 개방-패쇄 원칙
  + 개념 : 소프트웨어 요소(클래스, 모듈, 함수 등)는 확장에는 열려 있어야 하고, 변경에는 닫혀있어야 합니다.
  + 설명 : 기본 코드를 변경하지 않고도 기능을 추가하거나 확장할 수 있어야 합니다. 이는 주로 추상화와 다형성을 통해 달성됩니다.
  + 나쁜 예 : 특정 구분 값 추가에 따른 클래스 내부의 if-else 문을 계속 수정해야 하는 경우 새로운 구분값이 추가 될때마다 코드를 변경해야 합니다.
    * ```java
      class Payment {
        void pay(String type) {
          if (type.equals("CARD")) {...}
          else if (type.equals("CASH")) {...}
        }
      }
      ```
  + 좋은 예 : 메서드 인터페이스를 만들고, 각 구분 값에 대한 인터페이스를 구현하도록 합니다. 클래스는 메서드 인터페이스에 의존하여 새로운 구분값이 추가 되어도 기존 코드를 수정할 필요가 없습니다.
    * ```java
      interface PayStrategy { void pay(); }

      class CardPay implements PayStrategy { public void pay(){...} }
      class CashPay implements PayStrategy { public void pay(){...} }

      class Payment {
        void pay(PayStrategy strategy) { strategy.pay(); }
      }
      ```

- **LSP(Liskov Substitution Principle)** : 리스코프 치환 원칙
  + 개념 : 자식 클래스는 언제나 자신의 부모 클래스로 교체될 수 있어야 합니다.
  + 설명 : 자식 클래스는 부모클래스의 기능을 그대로 수행할 수 있어야 하며,
          자식 클래스를 사용한다고 해서 시스템의 동작이 예상치 못하게 변경되어서는 안 됩니다.
          즉, 부모클래스가 사용되는 곳에 자식 클래스를 넣어도 문제없이 동작해야 합니다.
  + 나쁜 예 : Bird 클래스에 fly()메서드가 있고, Penguin 클래스가 Brid 클래스를 상속 받지만 fly() 메서드를 오버라이드하여, 예외를 던지거나 아무것도 하지 않도록 구현하는 경우, Bird 타입으로 펭귄 객체를 다루면 fly() 호출시 문제가 발생할 수 있습니다.
    * ```java
      class Bird { void fly() {...} }

      class Penguin extends Bird {
        @Override
        void fly() { throw new UnsupportedOperationException(); }
      }
      ```
  + 좋은 예 : fly() 메서드가 필요한 새와 그렇지 않은 새를 구분하는 인터페이스를 도입하거나, 상속 관계를 재검토하여 Penguin이 Bird의 모든 행동 규약을 따르도록 설계합니다.
    * ```java
      interface Flyable { void fly(); }
      class Bird {}
      class Sparrow extends Bird implements Flyable { public void fly() {...} }
      class Penguin extends Bird {} // fly 불필요
      ```

- **ISP(Interface Segregation Principle)** : 인터페이스 분리 원칙
  + 개념 : 클라이언트는 자신이 사용하지 않는 메서드에 의존하도록 강요되어서는 안 됩니다.
  + 설명 : 하나의 거대한 인터페이스보다는 특정 클라이언트에 특화된 여러 개의 작은 인터페이스로 분리하는 것이 좋습니다. 이렇게 하면 클라이언트는 자신이 필요한 메서드만 알고 사용하게 되어 결합도가 낮아집니다.
  + 나쁜 예 : Worker 인터페이스에 work(), eat(), sleep() 메서드가 모두 포함되어 있고, 로봇 작업자는 eat()이나, sleep()메서드가 필요 없음에도, 이를 구현해야 하는 경우
    * ```java
      interface Worker {
        void work();
        void eat();
      }

      class Robot implements Worker {
        public void work() {...}
        public void eat() {...} // 필요 없음 → 강제 구현
      }
      ```
  + 좋은 예 : Workable(work()), Eatable(eat()), Sleepable(sleep()) 인터페이스로 분리하고, 각 클래스는 필요한 인터페이스만 구현하도록 합니다. 로봇작업자는 Workable 인터페이스만 구현할 수 있습니다.
    * ```java
      interface Workable { void work(); }
      interface Eatable { void eat(); }

      class Robot implements Workable { public void work() {...} }
      class Human implements Workable, Eatable {...}
      ```

- **DIP(Dependency Inversion Principle)** : 의존관계 역전 원칙
  + 개념 : 상위 수준 모듈은 하위 수준 모듈에 의존해서는 안 됩니다. 둘다 추상화에 의존해야하며, 추상화는 세부 사항에 의존해서 안 되며, 세부 사항이 추상화에 의존해야 합니다.
  + 설명 : 구체적인 클래스에 직접 의존하기 보다는 인터페이스나 추상클래스에 의존해야 합니다. 이를 통해 모듈 간의 결합도를 낮추고 유연성을 높일 수 있습니다. 의존성 주입은 이 원칙을 구현하는 일반적인 방법 중 하나 입니다.
  + 나쁜 예 : NotificationService 클래스가 구체적인 EmailSender 클래스에 직접 의존하는 경우, 나중에 SMS 알림으로 변경하려면 NotificationService 코드를 수정 해야 합니다.
    * ```java
      class EmailSender { void send(){...} }

      class Notification {
        private EmailSender emailSender = new EmailSender();
        void notifyUser() { emailSender.send(); }
      }
      ```
  + 좋은 예 : MessageSender 인터페이스를 만들고, EmailSender와, SmsSender가 이를 구현하도록 합니다.
            NotificationService는 MessageSender 인터페이스에 의존하며, 실제 사용할 MessageSender 구현체는 외부에서 주입 받습니다.
    * ```java
      interface MessageSender { void send(); }
      class EmailSender implements MessageSender { public void send(){...} }
      class SmsSender implements MessageSender { public void send(){...} }

      class Notification {
        private final MessageSender sender;
        Notification(MessageSender sender) { this.sender = sender; }
        void notifyUser() { sender.send(); }
      }
      ```

- SOLID 원칙의 이점
  + 유지보수성 향상 : 코드를 이해하고 수정하기 쉬워집니다.
  + 확장성 향상 : 새로운 기능을 추가하거나 기존 기능을 변경하기에 용이해집니다.
  + 재사용성 증가 : 모듈화가 잘 되어 있어 다른 프로젝트에서도 코드를 재사용하기 좋습니다.
  + 결합도 감소, 응집도 증가 : 모듈 간의 의존성은 줄어들고, 모듈 내의 요소들은 서로 밀접하게 관련됩니다.
  + 테스트 용이성 증가 : 각 모듈을 독립적으로 테스트하기 쉬워집니다.

- SOLID 요약
  + | 원칙  | 의미           | 잘못된 예 방지         |
    | --- | ------------ | ---------------- |
    | SRP | 단일 책임        | 하나 클래스가 여러 역할 수행 |
    | OCP | 확장 열림, 변경 닫힘 | if-else 수정 남발    |
    | LSP | 자식은 부모 대체 가능 | 잘못된 상속(펭귄-fly)   |
    | ISP | 인터페이스는 작게    | 필요 없는 메서드 강제 구현  |
    | DIP | 추상화에 의존      | 구체 클래스에 강한 결합    |


---
