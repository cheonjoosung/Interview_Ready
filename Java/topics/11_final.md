### final 키워드
- 개념
  + final = "더 이상 바꿀 수 없음" 을 명시하는 키워드
  + 적용 대상: 변수 / 메서드 / 클래스 → 각각 다른 의미

- final 변수
  + 한번 초기화 → 값 변경 불가
  + 초기화 방법
    * 선언과 동시에 초기화
    * 생성자에서 초기화 (인스턴스 변수)
    * static 블록에서 초기화 (클래스 변수)
  + ```java
    class MyClass {
      final int MAX_VALUE = 100; // 선언과 동시에 초기화
      final String GREETING;
      static final double PI = 3.14159;

      public MyClass(String greeting) {
          this.GREETING = greeting; // 생성자에서 초기화
      }

      public void doSomething() {
          // MAX_VALUE = 200; // 컴파일 오류! final 변수는 재할당 불가
          // GREETING = "Hello"; // 컴파일 오류!
      }
    }
    ```
  + 참조 타입 final 변수
    * 참조 변경 불가 (다른 객체를 가리킬 수 없음)
    * 하지만 참조된 객체의 내부 상태는 변경 가능
    * ```java
      final List<String> list = new ArrayList<>();
      list.add("hello");   // ✅ 가능 (리스트 내부 변경)
      // list = new ArrayList<>(); // ❌ 불가 (참조 변경)
      ```

- final 메서드
  + 오버라이딩 불가
  + 의도: 핵심 로직 보호, 일관성 유지
  + ```java
    class Animal {
      final void makeSound() {
        System.out.println("동물 소리");
      }
    
      void eat() {
        System.out.println("먹는다");
      }
    }
    
    class Dog extends Animal {
    // ❌ 컴파일 오류 (sound() 오버라이딩 불가)
    // @Override
    // void sound() { System.out.println("멍멍"); }
    }
    ```

- final 클래스
  + 상속 불가
  + 의도: 더 이상 확장할 필요 없는 클래스, 보안/불변 보장
  + ```java
    final class String { ... } // 실제 JDK의 String 클래스도 final
    ```
  
- final 키워드 사용의 이점
  + 불변성 확보 → 버그 예방, thread-safe한 객체 설계 용이
  + 보안 강화 → 상속이나 재정의로 인한 의도치 않은 동작 변경 방지
  + 의도 명확 → "이건 바뀌지 않는다"를 코드 차원에서 전달 가능
  + 성능 최적화 힌트 → JIT 컴파일러가 final 필드를 상수처럼 다룰 수 있음

- 면접 관련 질문
  + final 변수와 참조 타입 차이 설명 가능?
    * 참조는 바꿀 수 없지만 객체의 내부 상태는 변경 가능
  + final, finally, finalize 차이점은?
    * final → 변경 불가 (변수/메서드/클래스)
    * finally → 예외 처리 시 무조건 실행되는 블록
    * finalize() → GC가 객체 회수 직전에 호출하는 메서드 (Java 9부터 deprecated)
  + final 메서드와 private 메서드 차이?
    * private 메서드도 사실상 오버라이딩 불가 → 묵시적으로 final
    * 하지만 private은 접근 제한, final은 오버라이딩 제약이라는 의미 차이 있음


---
