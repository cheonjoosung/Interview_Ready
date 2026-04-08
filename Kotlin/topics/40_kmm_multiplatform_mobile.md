### KMM (Kotlin Multiplatform Mobile)
- 개념 및 정의
  + KMM은 Android와 iOS 앱에서 공통 비즈니스 로직(네트워크, DB, 유틸 등)을 Kotlin 코드로 작성하고 UI나 플랫폼 특화 부분은 각 플랫폼에서 따로 구현할 수 있도록 하는 멀티플랫폼 기술입니다.

- 프로젝트 구조 예시
  + ```text
    ├── shared/ (Kotlin 공통 코드)
    │   ├── commonMain/    // 공통 로직 (ViewModel, Repository, UseCase 등)
    │   ├── androidMain/   // Android 전용 구현
    │   └── iosMain/       // iOS 전용 구현

    ├── androidApp/        // Android UI (Compose, Jetpack)
    └── iosApp/            // iOS UI (SwiftUI, UIKit)
    ``` 

- KMM의 주요 특징 
  + | 특징                  | 설명                                   |
    | ------------------- | ------------------------------------ |
    | Cross-platform 지원   | 하나의 Kotlin 코드로 Android & iOS 대응      |
    | UI는 네이티브            | Compose, SwiftUI 등 플랫폼 네이티브 UI 유지    |
    | 비즈니스 로직 재사용         | 네트워크, DB, 유즈케이스, 데이터 모델 등 공유         |
    | Gradle 기반           | Kotlin DSL 빌드 구성 사용                  |
    | Native Framework 생성 | iOS에서 XCFramework로 컴파일 후 Swift 호출 가능 |

- 공통화 할 수 있는 코드
  + 비즈니스 로직, 데이터 모델, Repository, UseCase
  + 네트워크 처리 (Ktor)
  + 데이터 저장 (SQLDelight)
  + 날짜/시간 처리 (kotlinx-datetime)
  + 🚫 UI(View, Activity, SwiftUI 등)는 각 플랫폼에서 구현

- 예시
  + 공통코드 (commonMain)
    * ```kotlin
      // shared/commonMain
      class GreetingViewModel {
        fun greet(): String = "Hello, KMM!"
      }
      ```
  + iOS(Swift)
    * ```markdown
      let viewModel = GreetingViewModel()
      print(viewModel.greet())
      ``` 
  + Android
    * ```kotlin
      val vm = GreetingViewModel()
      Log.d("Greet", vm.greet())
      ```
  
- 대표 라이브러리
  + | 용도      | 라이브러리              |
    | ------- | ------------------ |
    | HTTP 통신 | Ktor               |
    | DB 저장   | SQLDelight         |
    | 비동기 처리  | Kotlinx.coroutines |
    | DI      | Koin, Kodein       |
    | 날짜/시간   | kotlinx-datetime   |

- 장점 & 단점
  + | 장점                           | 단점                        |
    | ---------------------------- | ------------------------- |
    | Kotlin 하나로 Android/iOS 코드 공유 | UI 공유 불가 (Flutter 대비 제한적) |
    | 네이티브 UI 활용 가능                | iOS 개발자가 Kotlin 이해 필요     |
    | 네이티브 수준 성능                   | 라이브러리 생태계 일부 미성숙          |
    | JetBrains + Google 공식 지원     | 초기 설정이 복잡할 수 있음           |
 
- 면접 관련 질문
  + KMM과 Flutter의 가장 큰 차이점은?
    * KMM은 비즈니스 로직만 공유하고 UI는 각 플랫폼에서 네이티브로 구현, Flutter는 UI까지 공유하는 크로스플랫폼 프레임워크입니다.
  + KMM에서 공통화가 불가능한 코드는 무엇인가요?
    * UI(View, Activity, SwiftUI 등)는 플랫폼 종속이므로 각 플랫폼에서 따로 구현해야 합니다.
  + iOS에서 KMM 코드를 어떻게 호출하나요?
    * 공통 Kotlin 코드를 iOS용 XCFramework로 빌드한 후 Swift/Objective-C에서 호출합니다. 


---
