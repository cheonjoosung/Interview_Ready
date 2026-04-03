### 의존성 주입 (DI)
- 개념 및 정의
  + 객체 간의 의존성을 외부에서 주입하는 방식
  + 클래스 내부에서 직접 의존 객체를 생성하지 않고, 외부에서 전달(주입) 받아 사용
  + 주입 방식
    * 생성자 주입 → 가장 권장
    * 세터 주입 → 선택적 의존성일 때때
    * 인터페이스 주입 → 덜 쓰임

- 왜 필요한가?
  + | 이유      | 설명                                       |
    | ------- | ---------------------------------------- |
    | 유지보수 용이 | 구체 클래스가 아닌 추상(인터페이스)에 의존 → 교체 쉬움         |
    | 테스트 쉬움  | Mock 객체/Fake Repository 주입 가능            |
    | 확장성     | 기존 코드 수정 없이 새로운 구현체를 주입 가능               |
    | 결합도 낮춤  | 클래스 간 의존 관계 약화 → SRP, OCP 같은 SOLID 원칙 실현 |

- 예시
  + ❌ 잘못된 방식 (강한 결합)
    * ```kotlin
      class Car {
        private val engine = Engine() // 직접 생성 → Engine 교체 불가
      }
      ```
  + ✅ 좋은 방식 (생성자 주입)
    * ```kotlin
      class Car(private val engine: Engine) {
        fun drive() = engine.start()
      }

      // 외부에서 주입
      val car = Car(GasolineEngine())
      ```
  + ✅ 인터페이스 활용 (확장성↑)  
    * ```kotlin
      interface Engine {
        fun start(): String
      }

      class GasolineEngine : Engine {
        override fun start() = "Gasoline Engine Start"
      }

      class ElectricEngine : Engine {
        override fun start() = "Electric Engine Start"
      }

      class Car(private val engine: Engine) {
        fun drive() = engine.start()
      }

      // 주입 시점에서 엔진 교체 가능
      val car = Car(ElectricEngine()) // 쉽게 교체 가능
      ```

- 📌 안드로이드에서 DI가 중요한 이유
  + 안드로이드 객체 수명 주기(Activity, Fragment, ViewModel 등)가 복잡
  + Context, Repository, UseCase 등 의존성이 얽혀있음
  + 테스트 용이성: 예를 들어, UserRepository 대신 FakeUserRepository 주입 가능
  + 전역 싱글톤(Context, DB 등) 관리 편리 → ApplicationContext, Room DB, Retrofit 인스턴스

- 안드로이드에서 DI 도구
  + | DI 도구       | 특징                                |
    | ----------- | --------------------------------- |
    | **Hilt**    | 구글 공식 DI (Dagger 기반, 보일러플레이트 최소화) |
    | **Dagger2** | 성능 최상급, 복잡한 프로젝트에 강력, 단 설정 어려움    |
    | **Koin**    | 코틀린 DSL 기반, 설정 쉬움, 러닝커브 낮음        |
    | **수동 DI**   | 직접 객체 생성/팩토리, 소규모 프로젝트에 적합        |

- Hilt 예시
  + ```kotlin
    // Repository
    class UserRepository @Inject constructor() {
      fun getUser() = "철수"
    }

    // ViewModel
    @HiltViewModel
    class MainViewModel @Inject constructor(
      private val repository: UserRepository
    ) : ViewModel() {
      val user = repository.getUser()
    }

    // Application 설정
    @HiltAndroidApp
    class MyApp : Application()
    ```

- 면접 관련 질문
  + DI란 무엇이고, 왜 필요한가요?
    * 객체 간 결합도를 낮추고 유지보수/테스트를 쉽게 하기 위해 사용
  + 안드로이드에서 DI가 특히 중요한 이유는?
    * 복잡한 생명주기 관리, Context/Repository 의존성 관리, 테스트 용이성 확보
  + Hilt와 Koin의 차이는?
    * Hilt: 컴파일 타임 DI (안전하지만 설정 복잡)
    * Koin: 런타임 DI (코틀린 친화적, 간단하지만 성능 다소 떨어짐)
  + 싱글톤 패턴 대신 DI를 쓰는 이유는?
    * 싱글톤은 전역 상태 공유로 테스트/확장에 제약이 크지만, DI는 유연성과 확장성이 뛰어남


---
