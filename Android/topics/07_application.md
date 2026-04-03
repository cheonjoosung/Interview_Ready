### Application 클래스 역할
- 정의
  + 앱 프로세스가 시작될 때 생성되어, 앱 전역에서 사용 가능한 **싱글톤 컨텍스트(Context)**를 제공하는 클래스
  + onCreate()는 앱이 실행될 때 가장 먼저 호출
  + 앱이 완전히 종료되기 전까지 인스턴스는 살아있음
  + 전역 상태 관리, 초기화, 라이브러리 설정 등에 사용

- 주요 역할
  + | 역할                | 설명                                                             |
    | ----------------- | -------------------------------------------------------------- |
    | **전역 Context 제공** | 어떤 컴포넌트에서도 `applicationContext`로 접근 가능                         |
    | **앱 시작 시 초기화 작업** | DI 초기화, Logger 설정, SDK 초기화 등                                   |
    | **전역 상태/데이터 관리**  | 로그인 상태, 글로벌 설정, 싱글톤 객체 관리 등                                    |
    | **생명주기 감지**       | `registerActivityLifecycleCallbacks()`를 통해 Activity 생명주기 추적 가능 |
    | **공통 처리 추상화**     | 공통 에러 처리, 공통 폰트, 공통 테마 적용 등                                    |

- 사용 방법
  + MyApp 클래스 생성 및 Application 상속 
    * ```kotlin
      class MyApp : Application() {
         override fun onCreate() {
           super.onCreate()
           // 예: Timber 초기화, Hilt, Firebase 등
         }
      }
      ```
  + AndroidManifest.xml 선언
    * ```xml
      <application
         android:name=".MyApp" >
      </application>
      ```

- Application과 Context 관계
  + Application은 앱 프로세스 단위의 Context
  + Activity, Service, BroadcastReceiver 등은 모두 Application Context를 내부적으로 참조함
  + 따라서 applicationContext는 항상 안전하게 앱 전체에서 사용할 수 있음
  + ❌ 단, UI 관련 작업 (View.inflate, Dialog 등)은 Activity Context 필요

- 전역 상태 관리의 위험성
  + Application은 앱 실행 내내 살아있음
  + 전역 변수처럼 쓰면 편하지만, 잘못 관리하면 메모리 누수, 동시성 문제 유발
  + 특히 Activity, Fragment, View 참조를 저장하면 GC가 불가능해져 누수 발생
  + ✅ 안전한 활용법
    * Application에는 라이브러리 초기화, DI Container, 공용 util만 저장
    * UI 컴포넌트 참조는 절대 저장하지 않음

- 주의사항
  + | 주의점                   | 설명                                    |
    | --------------------- | ------------------------------------- |
    | **메모리 누수 주의**         | Activity, Context 등 참조 저장 금지 (GC 안 됨) |
    | **성능 영향**             | `onCreate()`에서 너무 무거운 작업 금지           |
    | **Thread-safe 설계 필요** | 전역 변수 접근 시 동시성 문제 고려                  |

- 면접 관련 질문
  + Application 생성과 소멸 시점?
    * 앱 프로세스 시작 시 onCreate()가 호출되고, 앱이 완전히 종료될 때 소멸
  + Application과 Activity의 Context 차이?
    * Application은 앱 전역의 Context, Activity는 UI에 특화된 Context
    * Activity UI와 관련된 작업(ViewInflate, Theme 등)은 Application Context 에서 하면 안 됨
  + 전역 데이터 Application에 저장?
    * 간단한 설정 or 일시적 정보는 괜찮음
    * DB나 SharedPreferences가 필요한 복잡한 데이터 별도 관리 권장
    * 장기 보관은 ViewModel, Repository 등을 사용


---
