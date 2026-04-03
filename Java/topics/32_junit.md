### JUnit
- 개념
  + JUnit은 자바 프로그래밍 언어용 단위 테스트(Unit Test) 프레임워크입니다. 
  + 개발자가 작성한 코드의 작은 단위(주로 메소드 또는 클래스)가 의도한 대로 정확하게 작동하는지 검증하는 도구입니다.

- 단위 테스트(Unit Test)란?
  + 소프트웨어 개발에서 가장 작은 단위의 코드가 예상대로 동작하는지 확인하는 테스트입니다.
  + 버그를 조기에 발견하고, 코드 변경 시 안정성을 확보하며, 코드의 품질을 높이는 데 중요한 역할을 합니다.

- JUnit의 주요 특징 및 장점
  + 자동화된 테스트 
    * 테스트 코드를 작성해두면, JUnit이 자동으로 테스트를 실행하고 결과를 알려줍니다. 수동으로 일일이 확인하는 것보다 훨씬 효율적이고 반복 가능합니다.
  + 테스트 케이스 작성 용이
    * 어노테이션(Annotation) 기반으로 테스트 케이스를 쉽게 작성하고 관리할 수 있습니다. (예: @Test, @BeforeEach, @AfterEach 등)
  + 결과 확인 명확
    * 테스트 성공 시 녹색 막대, 실패 시 빨간색 막대로 결과를 시각적으로 보여주어 문제점을 빠르게 파악할 수 있습니다. (IDE 연동 시)
  + 독립적인 테스트
    * 각 테스트 케이스는 서로 독립적으로 실행되도록 권장됩니다. 하나의 테스트가 다른 테스트에 영향을 주지 않아 안정적인 테스트 환경을 제공합니다.
  + 통합 용이성
    * Maven, Gradle과 같은 빌드 도구 및 IntelliJ IDEA, Eclipse, Android Studio와 같은 IDE와 쉽게 통합되어 개발 과정에 자연스럽게 녹아들 수 있습니다.
  + 오픈 소스
    * 무료로 사용할 수 있으며, 활발한 커뮤니티를 통해 지속적으로 발전하고 있습니다. JUnit의 기본 구조 및 주요 어노테이션: JUnit 테스트 코드는 주로 다음과 같은 구조를 가집니다.

- 코드 예제
  + ```java
    class MyClassTest {

        private MyClass myClassInstance; // 테스트 대상 클래스의 인스턴스

        // 각 테스트 메소드 실행 전에 호출되는 메소드
        @BeforeEach
        void setUp() {
            System.out.println("Setting up for a test...");
            myClassInstance = new MyClass(); // 테스트에 필요한 객체 초기화
        }

        // 각 테스트 메소드 실행 후에 호출되는 메소드
        @AfterEach
        void tearDown() {
            System.out.println("Tearing down after a test...");
            myClassInstance = null; // 사용한 리소스 해제
        }

        // 모든 테스트 메소드 실행 전에 딱 한 번 호출되는 메소드 (static 이어야 함)
        @BeforeAll
        static void setUpAll() {
            System.out.println("Setting up once before all tests...");
        }

        // 모든 테스트 메소드 실행 후에 딱 한 번 호출되는 메소드 (static 이어야 함)
        @AfterAll
        static void tearDownAll() {
            System.out.println("Tearing down once after all tests...");
        }

        // 실제 테스트를 수행하는 메소드
        @Test
        void testAddition() {
            System.out.println("Running testAddition...");
            int result = myClassInstance.add(2, 3);
            // Assertions.assertEquals(expected, actual, [message]);
            assertEquals(5, result, "2 + 3 should be 5"); // 예상 결과와 실제 결과를 비교
        }

        @Test
        void testSubtraction() {
            System.out.println("Running testSubtraction...");
            int result = myClassInstance.subtract(5, 2);
            assertEquals(3, result, "5 - 2 should be 3");
        }

        @Test
        @Disabled("This test is currently disabled") // 이 테스트는 실행하지 않음
        void testMultiplication() {
            // ...
        }

        // 예외 발생을 테스트하는 경우
        @Test
        void testDivideByZero() {
            System.out.println("Running testDivideByZero...");
            // 특정 예외가 발생하는지 확인
            assertThrows(ArithmeticException.class, () -> {
                myClassInstance.divide(1, 0);
            }, "Dividing by zero should throw ArithmeticException");
        }
    }

    // 테스트 대상 클래스 (예시)
    class MyClass {
        public int add(int a, int b) {
        return a + b;
    }

        public int subtract(int a, int b) {
            return a - b;
        }

        public int divide(int a, int b) {
            if (b == 0) {
                throw new ArithmeticException("Cannot divide by zero");
            }
            return a / b;
        }
    }
    ``` 
    
- 주요 어노테이션 설명
  + @Test: 해당 메소드가 테스트 케이스임을 나타냅니다. JUnit은 이 어노테이션이 붙은 메소드를 찾아 실행합니다.
  + @BeforeEach: 각 @Test 메소드가 실행되기 직전에 실행됩니다. 테스트에 필요한 공통적인 준비 작업을 수행합니다 (예: 객체 생성, 변수 초기화). JUnit 4에서는 @Before였습니다.
  + @AfterEach: 각 @Test 메소드가 실행된 직후에 실행됩니다. 테스트 후 정리 작업을 수행합니다 (예: 리소스 해제). JUnit 4에서는 @After였습니다.
  + @BeforeAll: 현재 테스트 클래스의 모든 @Test 메소드가 실행되기 전에 딱 한 번 실행됩니다. 반드시 static 메소드여야 합니다. JUnit 4에서는 @BeforeClass였습니다.
  + @AfterAll: 현재 테스트 클래스의 모든 @Test 메소드가 실행된 후에 딱 한 번 실행됩니다. 반드시 static 메소드여야 합니다. JUnit 4에서는 @AfterClass였습니다.
  + @Disabled: 해당 @Test 메소드를 일시적으로 비활성화합니다. 테스트를 실행하지 않습니다. JUnit 4에서는 @Ignore였습니다.
  + @DisplayName("테스트 이름"): 테스트 결과에 표시될 사용자 정의 이름을 지정할 수 있습니다.

- Assertion 메소드
  + assertEquals(expected, actual): 두 값이 같은지 확인합니다.
  + assertNotEquals(unexpected, actual): 두 값이 다른지 확인합니다.
  + assertTrue(condition): 조건이 참인지 확인합니다.
  + assertFalse(condition): 조건이 거짓인지 확인합니다.
  + assertNull(object): 객체가 null인지 확인합니다.
  + assertNotNull(object): 객체가 null이 아닌지 확인합니다.
  + assertSame(expected, actual): 두 객체가 동일한 객체(메모리 주소)를 참조하는지 확인합니다.
  + assertNotSame(unexpected, actual): 두 객체가 다른 객체를 참조하는지 확인합니다.
  + assertThrows(expectedThrowable, executable): 특정 예외가 발생하는지 확인합니다.

- 왜 JUnit을 사용해야 할까요?
  + 버그 조기 발견: 개발 초기 단계에서 버그를 찾아 수정 비용을 줄일 수 있습니다.
  + 코드 변경에 대한 자신감: 코드를 리팩토링하거나 기능을 추가할 때, 기존 기능이 깨지지 않았는지 테스트를 통해 빠르게 확인할 수 있습니다.
  + 문서화 효과: 테스트 코드는 해당 코드가 어떻게 동작해야 하는지에 대한 살아있는 문서 역할을 합니다.
  + 설계 개선: 테스트하기 쉬운 코드를 작성하려고 노력하다 보면 자연스럽게 코드의 모듈성이 높아지고 결합도가 낮아지는 등 더 나은 설계로 이어질 수 있습니다.
  + 개발 생산성 향상: 디버깅 시간을 줄이고, 안정적인 코드를 빠르게 개발하는 데 도움을 줍니다. 
    JUnit은 현대 자바 개발에서 거의 필수적인 도구로 여겨지며, 특히 테스트 주도 개발(TDD - Test-Driven Development)과 같은 개발 방법론에서 핵심적인 역할을 합니다. 
    Android 개발에서도 유닛 테스트를 위해 JUnit이 널리 사용됩니다.

- 면접 질문
  + 좋은 단위 테스트의 특징은 무엇이라고 생각하시나요?
    * 좋은 단위 테스트는 FIRST 원칙을 따르는 경향이 있습니다.
    * Fast : 테스트는 빠르게 실행되어야 합니다. 느린 테스트는 개발 흐름을 방해하고 자주 실행되지 않게 됩니다.
    * Independent : 각 테스트는 다른 테스트에 의존하거나 영향을 주어서는 안됩니다. 테스트 실행 순서에 관계없이 동일한 결과를 보장해야합니다.
    * Repeatable : 어떤 환경에서도 동일한 결과를 내야합니다. 외부 요인에 의존하지 않아야 합니다.
    * Self-Validating : 테스트는 성공 또는 실패를 명확하게 스스로 판단할 수 있어야 합니다. 별도의 해석이나 수동 확인이 필요없어야 합니다.
    * Timely : 테스트는 프로덕션 코드를 작성하기 직전이나 동시에 작성하는 것이 이상적입니다. 너무 늦게 작성하면 테스트의 효과가 떨어질 수 있습니다.
  + Mock 객체는 언제 사용하며, JUnit 테스트와 어떤 관련이 있나요?
    * Mock 객체는 테스트 대상 객체가 의존하는 다른 객체를 실제 객체 대신 모방하여 사용하는 가짜 객체 입니다.
  + @BeforeEach와 @BeforeAll 차이점은 무엇인가요?
    * @BeforeEach : 각 @Test 메소드가 실행되기 전에 매번 호출됩니다.
    * @BeforeAll : 해당 테스트 클래스 내의 모든 @Test 메소드가 실행되기 전에 딱 한 번만 호출됩니다.
