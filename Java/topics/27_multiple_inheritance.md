### 다중 상속
- 하나의 클래스가 두 개 이상의 클래스로부터 상속을 받는 것
- ex)
  ```java
      class A { ... }
      class B { ... }
      class C extends A, B { ... } // ❌ Java에서는 불가
  ```
  
- Java에서 클래스는 다중 상속을 지원하지 않음
  ```java
      class A {
          void hello() { System.out.println("A"); }
      }
  
      class B {
          void hello() { System.out.println("B"); }
      }
  
      class C extends A {
          void hello() { System.out.println("C"); }
      }
     
      // class D extends B, C { ... } // ❌ Java는 이걸 허용하지 않음
  ```
    + 이렇게 되면 D.hello()는 B의 hello인지, C의 hello인지 충돌 → 컴파일러가 모호성으로 인해 에러 발생

- Java에서 지원되는 다중 상속의 형태
  + | 종류              | 설명                            |
                                        |-----------------|-------------------------------|
    | 클래스 다중 상속       | ❌ 지원하지 않음 (extends는 1개만 가능)   |
    | **인터페이스 다중 상속** | ✅ 완벽히 지원                      |
    | 디폴트 메서드 충돌 해결   | ✅ super<인터페이스>.메서드() 로 해결 가능  |

- 인터페이스를 통한 다중 상속
  ```java
      interface Flyable {
          default void move() {
              System.out.println("Flying");
          }
      }
  
      interface Swimmable {
          default void move() {
              System.out.println("Swimming");
          }
      }
  
      class Duck implements Flyable, Swimmable {
          @Override 
          public void move() {
              // 명시적으로 충돌 해결
              Flyable.super.move();  // 또는 Swimmable.super.move();
          }
      }
  ``` 
    + Java 8 이후, 인터페이스에 default 메서드를 정의할 수 있어 동작을 상속하게 되었음
    + → 따라서 인터페이스도 메서드 충돌이 생기면 직접 override 해서 해결해야 함

- 요약
  + | 항목                 | 지원 여부 | 설명                  |
              |--------------------|-------|---------------------|
    | 고전 방식              | ❌    | extends는 1개 클래스만 가능 |
    | synchronized       | ✅ | 여러 개 implements 가능  |
    | double-checked     | ✅ | 충돌 시 override 필요    |
    | static inner class | ❌    | 생성자는 상속되지 않음        |

- 다중 상속 충돌 해결법
  ```java
      interface A {
          default void hello() {
              System.out.println("A");
          }
      }
  
      interface B {
          default void hello() {
              System.out.println("B");
          }
      }
  
      class MyClass implements A, B {
          @Override
          public void hello() {
              A.super.hello();  // 또는 B.super.hello();
          }
      }
  ```
    + 충돌이 날 경우 반드시 override 해서 선택적으로 호출해야 함

- 면접 질문
  + Java에서 다중 상속이 가능한가요?
    * Java에서는 클래스 다중 상속은 불가능하지만, 인터페이스 다중 상속은 가능합니다.
  + 인터페이스는 왜 다중 상속이 가능한가요?
    * 인터페이스는 기본적으로 구현이 없었기 때문에 충돌이 발생하지 않았고,
    * Java 8 이후 default 메서드가 도입되면서도, 충돌 시 개발자가 명시적으로 override해서 해결해야 하므로 문제가 없습니다.
  + 추상 클래스와 인터페이스를 동시에 상속하면 어떤 게 우선인가요?
    * 클래스(추상 클래스 포함)의 메서드 구현이 우선 적용됩니다.
    * 같은 시그니처의 메서드가 있을 경우, 클래스 쪽 메서드가 우선됩니다.



---
