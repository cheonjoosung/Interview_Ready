### 추상클래스 vs 인터페이스
- 추상 클래스(Abstract Class) 란?
  + 개념
    * 불완전한 클래스
    * abstract 키워드로 선언
    * 직접 객체 생성 불가 → 상속 후 구현해야 사용 가능
  + 추상 클래스 특징
    * | 특징     | 설명                                       |
      | ------ | ---------------------------------------- |
      | 메서드    | 추상 메서드 + 일반 메서드 모두 가능                    |
      | 필드     | 인스턴스 변수 가질 수 있음                          |
      | 생성자    | 가질 수 있음                                  |
      | 상속     | **단일 상속**만 가능                            |
      | 접근 제한자 | `public`, `protected`, `private` 등 사용 가능 |
  + 추상 클래스 사용 목적
    * 여러 클래스에서 공통 기능 및 속성 정의
    * 일부 구현은 제공하고, 일부는 강제 구현
    * 상속 계층 구조의 기반 클래스로 활용 
  + 예시
    * ```java
      abstract class Shape {
        String color;
        public Shape(String color) { this.color = color; }
        abstract double getArea();
        public String getColor() { return color; }
      } 

      class Circle extends Shape {
        double radius;
        public Circle(String color, double radius) {
            super(color);
            this.radius = radius;
        }
        @Override
        double getArea() { return Math.PI * radius * radius; }
      }
      ```

- 인터페이스(Interface) 란?
  + 개념
    * 메서드의 명세(규약)만 정의
    * 구현 클래스가 반드시 메서드를 구현해야 함
    * 다중 상속 가능
  + 인터페이스 특징
    * | 특징     | 설명                                                      |
      | ------ | ------------------------------------------------------- |
      | 메서드    | Java 7까지는 추상 메서드만, Java 8부터는 `default`, `static` 메서드 가능 |
      | 필드     | \*\*상수(`public static final`)\*\*만 가능                   |
      | 상속     | **다중 구현 가능**                                            |
      | 접근 제한자 | 메서드는 묵시적으로 `public abstract`                            |
  + 인터페이스 사용목적
    * 다형성 구현
    * 서로 관련 없는 클래스가 동일한 규약을 따르게 함
    * 모듈 간 결합도 낮추기
  + 예시
    * ```java
      interface Animal {
        void makeSound();
        void eat();
      }

      class Dog implements Animal {
        public void makeSound() { System.out.println("Woof!"); }
        public void eat() { System.out.println("Dog food"); }
      }
      ```
  
- 차이점 비교
  + | 구분     | 추상 클래스                  | 인터페이스                                     |
    | ------ | ----------------------- | ----------------------------------------- |
    | 객체 생성  | 불가                      | 불가                                        |
    | 메서드    | 추상 메서드 + 일반 메서드         | (Java 8 이상) 추상 메서드 + default + static 메서드 |
    | 필드     | 인스턴스 변수 가능              | 상수(`public static final`)만 가능             |
    | 상속     | 단일 상속                   | 다중 구현                                     |
    | 생성자    | 가능                      | 불가                                        |
    | 접근 제한자 | 자유롭게 지정 가능              | 메서드는 묵시적으로 `public abstract`              |
    | 사용 목적  | 공통 속성과 기능 공유 + 일부 구현 제공 | 구현 강제 + 다형성 제공                            |

- 선택 기준
  + 추상 클래스
    * 관련성이 높은 클래스들
    * 공통 필드나 메서드가 필요
    * 접근 제어자 다양하게 사용해야 함
    * 일부 구현을 부모에서 제공하고 싶음
  + 인터페이스
    * 관련 없는 클래스들이 동일한 기능 제공
    * 다중 상속 필요
    * 규약만 정의하고 구현은 각 클래스에 맡김

- 면접 관련 질문
  + 추상 클래스와 인터페이스의 가장 큰 차이는?
    * 추상 클래스는 단일 상속이며 필드·구현 메서드를 가질 수 있고, 인터페이스는 다중 구현이 가능하며 상수와 규약 위주로 구성됩니다.
  + 인터페이스에 default 메서드가 추가된 이유는?
    * 기존 인터페이스에 새로운 메서드를 추가할 때, 모든 구현체를 수정하지 않고도 기본 구현을 제공하기 위해서입니다.
  + 추상 클래스와 인터페이스 중 어느 것을 선택할지 어떻게 결정하나요?
    * 공통 상태와 동작이 필요 → 추상 클래스
    * 구현체 간 관계가 없고, 기능 규약만 필요 → 인터페이스
  + 클래스가 이미 다른 클래스를 상속 중인데 추상 클래스의 기능까지 쓰고 싶으면?
    * Java는 다중 상속 불가이므로, 인터페이스를 사용하거나 조합(Composition) 으로 기능을 추가합니다.


---
