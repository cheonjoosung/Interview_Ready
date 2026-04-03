### 리플렉션(Reflection)
- 정의
  + 실행 중인 클래스의 메타데이터를 읽고 수정할 수 있는 기능
  + java.lang.reflect 패키지 제공
  + 컴파일 시점이 아니라 런타임 시점에 클래스, 메서드, 필드 등에 접근 가능

- 주요 기능 및 사용 사례
  + 클래스 정보 얻기
    * 런타임에 클래스의 이름, 슈퍼클래스, 인터페이스, 접근 제한자, 생성자, 메서드, 필드 등 상세 정보를 얻을 수 있습니다.
    * ```java
      Class<?> clazz = Class.forName("com.example.User");
      System.out.println(clazz.getName());  // 클래스 이름
      ```
  + 객체 생성
    * Class 객체의 newInstance() 메서드 또는 Constructor 객체의 newInstance() 메서드를 사용하여 런타임에 객체를 동적으로 생성할 수 있습니다.
    * 이를 통해 특정 조건에 따라 다른 클래스의 객체를 생성하는 유연한 코드를 작성할 수 있습니다.
    * ```java
      Constructor<?> constructor = clazz.getConstructor(String.class, int.class);
      Object obj = constructor.newInstance("Alice", 30);
      ```
      1. clazz.newInstance() (deprecated → 예외처리 불명확)
      2. 보통 Constructor.newInstance() 권장
  + 메서드 호출
    * Method 객체를 얻은 후, invoke() 메서드를 사용하여 해당 메서드를 동적으로 호출할 수 있습니다.
    * 메서드 이름이나 파라미터 타입을 런타임에 결정하여 호출할 수 있습니다.
    * ```java
      Method method = clazz.getMethod("sayHello");
      method.invoke(obj); // obj.sayHello() 실행\
      ```
  + 필드 값 접근 및 수정
    * Field 객체를 얻은 후, get() 메서드로 필드 값을 읽고, set() 메서드로 필드 값을 변경할 수 있습니다.
    * private 필드에도 접근 제한을 우회하여 접근할 수 있는 기능을 제공합니다.
    * ```java
      Field field = clazz.getDeclaredField("name");
      field.setAccessible(true); // private 접근 허용
      field.set(obj, "Bob");
      System.out.println(field.get(obj)); // Bob
      ```

- 리플렉션의 장점
  + 유연성/확장성: 이름만 알고 있어도 동적 호출 가능
  + 프레임워크 필수 기능: Spring DI, Hibernate ORM, JUnit 등이 내부적으로 리플렉션 사용
  + 플러그인 아키텍처: 런타임에 모듈 로딩, 객체 주입 가능

- 리플렉션의 단점 및 주의사항:
  + 성능 저하
    * 일반 메서드 호출 대비 훨씬 느림 (메서드 캐싱/MethodHandle로 최적화 가능)
  + 캡슐화 위반
    * private 접근 가능 → 보안 취약점 유발
  + 런타임 에러 위험
    * 컴파일러가 타입 검증 못 함 (메서드 이름/파라미터 불일치 시 런타임 예외)
  + 가독성/유지보수성 저하
  
- 활용 사례
  + Spring: Bean 생성/주입, AOP 프록시
  + Hibernate: 엔티티 필드 접근
  + JUnit: private 메서드 테스트 가능
  + Jackson/Gson: 런타임에 필드 조회해 JSON 직렬화/역직렬화
  
- 면접 관련 질문
  + 리플렉션이 왜 필요한가?
    * → 런타임에 객체를 동적으로 다루고, 프레임워크가 개발자 대신 객체 생성/DI/프록시를 구현할 수 있기 때문
  + 단점은?
    * → 성능 저하, 캡슐화 위반, 유지보수성 저하
  + 대체할 방법은?
    * → 가능하다면 제네릭, 인터페이스, DI 같은 정적 타입 기반 설계로 해결 → 리플렉션은 최후의 수단


---
