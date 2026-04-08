### 오버로딩(Overloading) vs 오버라이딩(overriding)
- **오버로딩**
  + 개념
    * 같은 클래스 내에서 메서드 이름이 같고 매개변수 목록(타입·개수·순서) 이 다른 메서드를 여러 개 정의
    * 반환 타입만 다르면 시그니처가 동일하므로 오버로딩 불가
  + 특징
    * 컴파일 시점에 호출할 메서드 결정 (정적 바인딩, Static Binding)
    * 메서드 시그니처: 이름 + 매개변수 목록
    * 같은 기능을 다양한 입력에 맞게 제공할 때 사용
    * 가독성과 유지보수성 향상
    * ```java
      public class Calculator {
        public int add(int a, int b) { return a + b; }
        public double add(double a, double b) { return a + b; }
        public int add(int a, int b, int c) { return a + b + c; }
      }

      // 호출
      Calculator calc = new Calculator();
      calc.add(1, 2);       // add(int,int)
      calc.add(1.5, 2.3);   // add(double,double)
      calc.add(1, 2, 3);    // add(int,int,int)
      ```
  
- **오버라이딩**
  + 개념
    * 상속 관계에서 자식 클래스가 부모 클래스의 메서드를 같은 시그니처로 재정의
    * 부모 클래스의 기본 동작을 변경·확장 가능
  + 특징
    * 런타임 시점에 실제 객체 타입에 따라 호출 메서드 결정 (동적 바인딩, Dynamic Binding)
    * @Override 애노테이션 사용 권장
    * 접근 제어자는 부모보다 좁게 변경 불가
    * 예외 선언(throws)은 부모 메서드보다 넓게(더 많은) 선언 불가
    * 다형성을 구현하는 핵심 기능 확장할 때 사용
    * ```java
      class Animal {
        public void sound() { System.out.println("동물이 웁니다"); }
      }

      class Dog extends Animal {
        @Override
        public void sound() { System.out.println("멍멍!"); }
      }

      class Cat extends Animal {
        @Override
        public void sound() { System.out.println("야옹~"); }
      }

      // 호출
      Animal a1 = new Dog();
      Animal a2 = new Cat();
      a1.sound(); // 멍멍!
      a2.sound(); // 야옹~
      ```

- 차이점 비교
  + | 구분       | 오버로딩 (Overloading)      | 오버라이딩 (Overriding)          |
    | -------- | ----------------------- | --------------------------- |
    | 적용 범위    | 같은 클래스 내                | 상속 관계(부모·자식 클래스)            |
    | 시그니처     | 메서드 이름 동일, 매개변수 목록 다름   | 메서드 이름, 매개변수, 반환 타입 모두 동일   |
    | 바인딩 시점   | 컴파일 타임 (Static Binding) | 런타임 (Dynamic Binding)       |
    | 반환 타입    | 매개변수가 다르면 다르게 가능        | 부모와 같거나 하위 타입(Covariant) 가능 |
    | 접근 제어자   | 제약 없음                   | 부모보다 좁게 불가                  |
    | 예외 선언    | 제약 없음                   | 부모보다 넓게 불가                  |
    | 사용 목적    | 같은 기능의 다양한 버전 제공        | 부모 기능 재정의 및 확장              |
    | 다형성과의 관계 | 무관                      | 다형성 구현의 핵심                  |

- 면접 관련 질문
  + 오버로딩과 오버라이딩의 가장 큰 차이는 무엇인가요?
    * 오버로딩: 같은 클래스 내, 매개변수 목록이 다른 동일 이름 메서드, 컴파일 시점 결정
    * 오버라이딩: 상속 관계에서 부모 메서드를 재정의, 런타임 시점 결정
  + 오버라이딩 시 접근 제어자나 예외 선언에 제약이 있는 이유는?
    * 접근 제어자는 부모보다 좁히면 다형성 원칙을 깨서 객체 사용 시 접근 불가 문제가 생김
    * 예외 선언을 넓히면 기존 코드에서 처리하지 못하는 예외가 발생해 호환성이 깨짐
  + 오버로딩에서 반환 타입만 다르게 하면 왜 불가능한가요?
    * 자바는 메서드 시그니처에 반환 타입을 포함하지 않음
    * 호출 시 어떤 메서드를 사용할지 컴파일러가 구분할 수 없어 모호성이 발생


---
