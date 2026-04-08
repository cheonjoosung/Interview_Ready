### AAB vs APK
- AAB(안드로이드 앱 번들) — 개념 & 특징
  + 정의  
    * 개발자가 빌드한 모든 코드·리소스·구성(모듈)을 묶은 번들 파일(.aab).
  + 핵심 아이디어
    * Play가 번들을 받아 기기(ABI, 언어, 화면밀도 등)별로 최적화된 APK(또는 split APK 세트) 를 생성해 제공 → 사용자는 필요한 리소스/코드만 다운로드.
  + 지원 기능
    * Dynamic Feature Modules: 기능을 install-time 또는 on-demand로 분리 가능
    * Configuration splits: ABI, density, language에 따른 분할 배포.
    * Play App Signing: Play가 최종 APK 서명(대부분의 경우) — 개발자는 업로드 키를 사용.
  + 흔히 사용하는 워크플로: `./gradlew bundleRelease` → `app-release.aab` 생성 → Play Console 업로드

- APK — 개념 & 특징
  + 정의
    * 설치 가능한 표준 안드로이드 패키지(.apk).
  + 구성  
    * `classes.dex`, `res/`, `AndroidManifest.xml`, `lib/`, `assets/`, `META-INF(서명)` 등.
  + 특징
    * 바로 adb install로 단말에 설치 가능.
    * Play로 업로드할 수도 있고, 다른 스토어/사이드로드용으로 배포 가능.
    * 단일 파일로 모든 리소스 포함 → 사용자에게 불필요한 리소스까지 전달될 가능성.

- 비교
  + | 항목       |                                          AAB (Android App Bundle) | APK (Android Package)                            |
    | -------- | ----------------------------------------------------------------: | ------------------------------------------------ |
    | 목적       |                              Play에서 앱을 **기기 맞춤형으로 생성·배포**하기 위한 번들 | 단말에 **설치 가능한 최종 패키지**                            |
    | 포함 요소    |   code (classes.jar), res, manifest, native libs, dynamic modules | classes.dex, res, manifest, native libs, 서명 정보 등 |
    | 배포 방식    | Google Play(또는 bundletool 이용)에서 기기별 APK(또는 split APK)를 생성·서명 후 배포 | 개발자가 직접 서명해서 배포(스토어/사이드로드)                       |
    | 설치 가능 여부 |               직접 설치 불가 — bundletool이나 Play Console을 통해 APK로 변환/설치 | 바로 설치 가능 (`adb install`)                         |
    | 장점       |          사용자 다운로드/설치 크기 감소, Dynamic Feature(온디맨드 모듈) 지원, Play 최적화 | 단순함, Play 외 배포(사이드로드, 다른 스토어)에 유리                |
    | 단점       |                                로컬 설치/테스트가 번거로움(번들툴 필요), Play에 의존적 | 사용자에게 전체 패키지 전달 → 큰 용량, 불필요한 리소스 포함              |
 
- Play Console과 App Signing
  + Play App Signing: AAB를 업로드하면 Play가 기기별 APK를 빌드하고 Play의 앱 서명 키로 최종 APK에 서명합니다.
    * 개발자는 upload key로 AAB를 업로드(서명)하고, Play가 App Signing Key로 재서명.
    * 장점: 키 관리 편리, 향후 키 교체 지원 등.
  + 테스트 방법
    * Internal testing / Closed testing 트랙을 통해 실제 Play에서 기기별 APK 생성 후 검증.
    * 로컬 테스트는 bundletool 사용.

- AAB의 장점 / 단점 — 실무 요약
  + 장점  
    * 다운로드/설치 크기 감소: 기기별로 필요한 리소스만 전달 → 사용자 경험 개선(데이터/용량 절약)
    * Dynamic Delivery: 기능을 모듈화해 on-demand로 제공 → 초기 설치 용량 축소
    * Play 최적화: Play가 기기 맞춤 APK를 생성·최적화
    * 보안/키 관리: Play App Signing으로 키 관리를 쉽게
  + 단점 / 고려사항
    * 직접 설치 불가: 로컬 테스트/사이드로드 시 번들툴 필요 → 개발/테스트 흐름 변경 필요
    * Play 의존성: AAB의 가장 큰 장점은 Play에서 발휘 → 다른 스토어 배포 시 추가 작업 필요
    * 테스트 복잡성: 다양한 split 조합·Dynamic feature 테스트 필요
    * 구현·운영 복잡성: 모듈화 설계·버전 관리·on-demand API 사용 등의 추가 비용

- AAB로 전환할 때 체크리스트 & 모범 사례
  + ./gradlew bundleRelease가 정상 빌드되는지 확인
  + Play App Signing에 등록(필요한 경우 upload key 준비)
  + 중요 라이브러리(native .so 등)가 ABI별로 잘 분리되는지 확인
  + Dynamic Feature 도입 시: 기능 경계, 의존성, 런타임 로드/언로드 시나리오 설계
  + 자동화(CI): 번들 빌드 및 Play Upload(Gradle Play Publisher, Fastlane) 도입
  + 테스트: internal testing 트랙 + bundletool 로컬 테스트
  + 리소스/문자열 다국어 분리 → language split으로 사이즈 절약 가능
  + ProGuard/R8로 코드 축소 & 리소스 shrink 활성화

- 면접 관련 질문
  + AAB와 APK의 가장 큰 차이는 무엇인가요?
    * APK는 설치 가능한 단일 패키지이고, AAB는 앱의 모든 코드·리소스를 묶은 번들입니다. 
    * Play가 AAB를 받아서 기기별로 최적화된 APK(또는 split APK)를 생성·서명해 배포한다는 점이 핵심 차이입니다.
  + AAB를 업로드하면 누가 서명하나요? 개발자는 추가로 할 일이 있나요?
    * 보통 Google Play가 최종 APK에 Play App Signing 키로 서명합니다. 
    * 개발자는 로컬에서 upload key로 AAB를 서명해 업로드하고, Play에서 재서명합니다. (Play App Signing을 사용하지 않는 옵션도 있지만 Play에서 서명 관리하는 것이 권장됩니다.)
  + AAB로 전환했을 때 테스트/로컬 설치는 어떻게 하나요?
    * AAB는 직접 adb install 불가합니다.
    * 로컬 테스트는 bundletool을 사용해 .apks를 생성하고 bundletool install-apks로 설치하거나 unzip 후 universal.apk를 추출해 설치합니다.
    * 또 Play Console의 internal testing 트랙을 활용하면 Play에서 기기별 APK를 생성·배포해서 테스트할 수 있습니다. 
