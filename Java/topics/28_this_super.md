### this / super
- this
  + 현재 객체 자신을 참조하는 키워드
  + 사용 목적
    * | 목적             | 예시                |
      |----------------|-------------------|
      | 멤버 변수와 지역 변수 이름이 겹칠 때 구분     | this.name = name; |
      |생성자에서 다른 생성자를 호출할 때 (this(...))     | this(name, age);  |
      | 메서드 체이닝      | return this;      |
  + 예시
      ```java
          class Person {
              String name;

              Person(String name) {
                  this.name = name; // this는 현재 인스턴스의 필드
              }

              void printName() {
                  System.out.println(this.name);
              }
        }
      ```
- super
  + 부모 클래스의 멤버(필드/메서드/생성자) 를 참조할 때 사용
  + 사용 목적
    * | 목적             | 예시                          |
      |----------------|-----------------------------|
      | 부모 클래스의 필드/메서드에 접근      | super.name, super.sayHello() |
      |자식 클래스에서 오버라이딩된 메서드 내부에서 부모 메서드 호출     | super.methodName()      |
      | 자식 클래스 생성자에서 부모 생성자 호출      | super(...) → 생성자의 첫 줄에서만 사용 가능<br/>      |
    * 예시
     ```java
        class Animal {
            String name = "Animal";

            void speak() {
                System.out.println("Animal speaks");
            }
        }

        class Dog extends Animal {
            String name = "Dog";

            void printNames() {
                System.out.println(this.name);   // Dog
                System.out.println(super.name); // Animal
            }

            @Override
            void speak() {
                super.speak();  // Animal speaks
                System.out.println("Dog barks");
            }
        }
     ```
- 생성자에서의 this(...) vs super(...)
   + | 키워드        | 역할                | 위치 제안            |
     |------------|-------------------|------------------|
     | this(...)  | 같은 클래스의 다른 생성자 호출 | 생성자의 첫 줄    |
     | super(...) | 부모 클래스의 생성자 호출    | 생성자의 첫 줄 |
- this vs super 차이
   + | 항목       | this                      | super       |
     |----------|---------------------------|-------------|
     | 참조 대상    | 현재 클래스의 인스턴스              | 부모 클래스의 멤버  |
     | 접근 대상    | 필드, 메서드, 생성자              | 부모의 필드, 메서드, 생성자|
     | 생성자에서 사용 | this(...) – 같은 클래스의 생성자 호출 | super(...) – 부모 클래스 생성자 호출    |
     | 위치 제한     |  생성자에서 첫 줄에만 사용 가능     |  생성자에서 첫 줄에만 사용 가능     |

- 면접 질문
  + this와 super의 주요 차이점은 무엇인가요?
    * 참조 대상으로 this는 현재 객체 자신을 참조하고, super는 부모 클래스의 멤버 또는 생성자를 참조합니다.
    * this는 일반 메서드, 생성자 등 다양한 곳에서 사용될 수 있지만, super는 주로 자식 클래스에서 부모 클래스의 멤버에 접근하거나 생성자를 호출할 때 사용됩니다. super()는 생성자의 첫 줄에서만 사용 가능합니다.
    * this()는 같은 클래스의 다른 생성자를, super()는 부모 클래스의 생성자를 호출합니다.



---
