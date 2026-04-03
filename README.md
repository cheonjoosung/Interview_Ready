![main_image](./_assets/main.png)

# 안드로이드 개발자를 위한 완벽한 가이드

이 저장소는 **안드로이드 앱 개발자**를 위한 지식 정리 공간입니다.  
Java와 Kotlin 언어를 기반으로 Android의 핵심 개념, 아키텍처 패턴, 비동기 처리, UI 구성 등  
실무와 면접에 모두 활용 가능한 내용을 구조적으로 정리했습니다.

- 👨‍💻 **초보자부터 중급 개발자까지** 모두를 위한 기술 정리
- 📚 **기초 문법부터 실전 적용까지** 단계별 구성
- 🧩 **Android, Java, Kotlin, Architecture, Jetpack, Testing** 여섯 가지 주제로 분류

> 학습, 복습, 포트폴리오 작성, 면접 준비 등 다양한 목적에 활용할 수 있는 가이드입니다.

---

## 📂 카테고리

| 카테고리 | 주제 수 | 설명 |
|----------|---------|------|
| [🤖 Android](./Android) | 27개 | 컴포넌트, 생명주기, 시스템 |
| [☕ Java](./Java) | 32개 | 언어 기초, OOP, JVM, 동시성 |
| [🎯 Kotlin](./Kotlin) | 44개 | 언어 기초, 코루틴, 제네릭 |
| [🏗 Architecture](./Architecture) | 4개 | MVVM, Clean, MVI, Repository |
| [🚀 Jetpack](./Jetpack) | 5개 | ViewModel, Flow, Room, Hilt, Compose |
| [🧪 Testing](./Testing) | 1개+ | 단위 테스트, UI 테스트 |

---

## 📝 전체 목차

### 🤖 [Android](./Android)
- Activity / Fragment 생명주기
- 4대 컴포넌트 (Activity, Service, Content Provider, BroadcastReceiver)
- Context, Application, AndroidManifest
- 직렬화 (Parcelable & Serializable)
- 의존성 주입 (DI), WorkManager, DiffUtil
- OkHttp & Retrofit, Handler/Looper, Interceptor
- 성능 최적화, 앱 서명, AAB vs APK

### ☕ [Java](./Java)
- 기본형 vs 참조형, String, Equals & hashCode
- OOP (추상클래스, 인터페이스, SOLID, 싱글턴)
- JVM, GC, 메모리 구조
- Thread, synchronized, volatile, Atomic

### 🎯 [Kotlin](./Kotlin)
- 코틀린 언어 기초 (data class, sealed class, enum)
- 스코프 함수, 확장 함수, 고차 함수
- 제네릭 (reified, 공변성, 반공변성)
- 코루틴 (suspend, Dispatchers, Flow)

### 🏗 [Architecture](./Architecture)
- **MVVM** - ViewModel + LiveData/StateFlow
- **Clean Architecture** - Domain / Data / Presentation 계층 분리
- **MVI** - 단방향 데이터 흐름, 불변 상태
- **Repository 패턴** - 데이터 소스 추상화

### 🚀 [Jetpack](./Jetpack)
- **ViewModel & LiveData** - 생명주기 인식 데이터 홀더
- **Flow & StateFlow & SharedFlow** - 비동기 스트림
- **Room** - SQLite ORM, 마이그레이션
- **Hilt** - 컴파일 타임 DI 프레임워크
- **Jetpack Compose** - 선언형 UI

### 🧪 [Testing](./Testing)
- **Unit Testing** - ViewModel, UseCase, Repository 테스트
- MockK를 활용한 Mocking

---

## 📌 추가로 학습해야 할 개념 목록

> 아래 개념들은 이 저장소에 아직 포함되지 않은 주제입니다. 면접에서 자주 등장하므로 학습을 권장합니다.

### Android 심화
- [ ] **RecyclerView 내부 동작** - RecycledViewPool, ViewHolder 재사용 메커니즘
- [ ] **커스텀 View** - onMeasure, onDraw, onLayout
- [ ] **Navigation Component** - NavGraph, deeplink, SafeArgs
- [ ] **Paging 3** - 무한 스크롤, RemoteMediator
- [ ] **DataBinding vs ViewBinding** - 차이점, 성능
- [ ] **ConstraintLayout 내부 원리**
- [ ] **WindowInsets** (엣지 투 엣지 UI)
- [ ] **App Startup** - 초기화 라이브러리 최적화
- [ ] **ProGuard / R8** - 코드 난독화, 최적화

### 코루틴 심화
- [ ] **Channel** - Producer-Consumer 패턴
- [ ] **StructuredConcurrency** - 부모-자식 코루틴 관계
- [ ] **Exception Handling in Coroutines** - SupervisorJob, CoroutineExceptionHandler
- [ ] **Flow 연산자 심화** - flatMapLatest, zip, combine 실전

### 아키텍처 심화
- [ ] **Paging + Room + Network** - 오프라인 우선 아키텍처
- [ ] **멀티 모듈 아키텍처** - feature, core, app 모듈 분리
- [ ] **Compose Navigation** - Compose 환경에서의 화면 전환
- [ ] **Baseline Profiles** - 앱 시작 성능 최적화

### 네트워크 & 보안
- [ ] **Certificate Pinning** - MITM 공격 방어
- [ ] **OAuth 2.0 / JWT** - 인증/인가 흐름
- [ ] **Android Keystore** - 키 보안 저장소
- [ ] **Network Security Config**

### 테스트
- [ ] **Espresso** - UI 자동화 테스트
- [ ] **Compose UI Testing** - Composable 테스트
- [ ] **MockWebServer** - 네트워크 계층 테스트
- [ ] **TDD** - 테스트 주도 개발 방법론

### 빌드 & 배포
- [ ] **Gradle 빌드 시스템** - task, 의존성 설정, 멀티 플레이버
- [ ] **CI/CD** - GitHub Actions, Fastlane 활용
- [ ] **Firebase App Distribution** - 테스트 배포
- [ ] **Google Play Console** - 단계적 출시, A/B 테스팅

### 자료구조 & 알고리즘 (코딩 테스트 대비)
- [ ] **Array, LinkedList, Stack, Queue** - 시간 복잡도
- [ ] **HashMap 내부 동작** - 해시 충돌, 리해싱
- [ ] **Binary Search, DFS, BFS** - Android 면접에서 종종 출제
- [ ] **동적 프로그래밍(DP)** - 메모이제이션
