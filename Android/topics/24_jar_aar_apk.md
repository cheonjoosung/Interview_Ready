### jar, aar, apk
- JAR (Java ARchive)
  + 정의
    *  Java 클래스와 메타데이터를 압축한 라이브러리 패키지
  + 구성
    * .class 파일 (컴파일된 Java 바이트코드)
    * META-INF/MANIFEST.MF
    * 일반 리소스 파일(.properties, .xml 등) 포함 가능
    * ❌ Android 리소스(res/, AndroidManifest.xml) 포함 불가
  + 사용처
    * 순수 Java 코드 기반 라이브러리 (ex: Gson, Apache Commons, OkHttp JAR 버전)

- AAR (Android ARchive)
  + 정의
    *  안드로이드 라이브러리 모듈을 패키징한 파일 (JAR의 Android 확장 버전)
  + 구성
    * classes.jar (Java/Kotlin 코드)
    * res/, assets/, jni/
    * AndroidManifest.xml
    * proguard.txt 등 추가 설정 파일
  + 사용처
    * Android 전용 모듈을 라이브러리 형태로 배포
    * 다른 프로젝트에서 공통 UI, SDK, 커스텀 컴포넌트 재사용
    
- APK (Android Package)
  + 정의
    * 안드로이드 앱 최종 패키지 (사용자 단말기에 설치 가능한 실행 파일)
  + 구성
    * classes.dex (Dalvik/ART 바이트코드)
    * res/, assets/, AndroidManifest.xml
    * META-INF/ (서명 정보)
    * lib/ (네이티브 라이브러리)
  + 사용처
    * 최종적으로 배포되는 앱 파일 (Google Play 스토어 업로드 및 단말기 설치)
    
- 비교표
  + | 항목             | JAR (Java Archive) | AAR (Android Archive) | APK (Android Package) |
    | -------------- | ------------------ | --------------------- | --------------------- |
    | 목적             | Java 라이브러리         | Android 라이브러리         | 앱 최종 실행 패키지           |
    | 포함 가능 요소       | Java 코드만           | 코드 + 리소스 + Manifest   | 코드 + 리소스 + 서명         |
    | 설치 가능 여부       | ❌                  | ❌                     | ✅                     |
    | 리소스 포함 여부      | ❌                  | ✅                     | ✅                     |
    | Manifest 포함 여부 | ❌                  | ✅                     | ✅                     |
    | 사용 대상          | Java 애플리케이션        | Android 프로젝트 공유 모듈    | 사용자 단말기 설치 앱          |

- APK 빌드 과정
  + ```text
    .java / .kt (소스 코드)
        │
        ▼
    .class (Java 바이트코드)     ← javac / kotlinc 컴파일러
        │
        ▼
    .jar (Java Archive)        ← 클래스 + 메타데이터 묶음
        │
        ▼
    .dex (Dalvik Executable)   ← D8/R8 (ART/Dalvik VM용 변환)
        │
        ▼
    APK (Android Package)       ← .dex + res/ + Manifest + assets/ + lib/ + META-INF
          │
          ▼
    (서명 & zipalign)           ← 서명(Signature) + 최적화
          │
          ▼
    최종 설치 파일 (.apk)         ← 사용자 단말기에 설치 가능
    ```
  + 소스 코드 (.java / .kt)
    * 개발자가 작성한 코드
  + .class (바이트코드)
    * javac (Java), kotlinc (Kotlin) 컴파일러가 바이트코드로 변환
  + .jar (Java Archive)
    * 여러 .class 파일과 메타데이터를 묶은 라이브러리 형태
  + .dex (Dalvik Executable)
    * D8 (또는 R8: 최적화/압축 포함)이 .class → .dex로 변환
    * 안드로이드 런타임(ART/Dalvik VM)에서 실행 가능한 코드
  + APK 패키징
    * classes.dex + res/(리소스) + AndroidManifest.xml + assets/ + lib/
    * 모든 앱 구성 요소를 하나로 묶음
  + 서명(Signing) & zipalign 최적화
    * Google Play에 배포하거나 단말기에 설치하려면 반드시 앱에 서명(Signature) 필요
    * zipalign은 압축을 정렬해 런타임 성능 최적화
  + 최종 APK
    * 사용자가 설치할 수 있는 실제 앱 파일

- 면접 질문
  + JAR, ARR, APK의 차이가 무엇일까요?
    * JAR : Java 아카이브로, 컴파일된 .class 파일과 메타정보만 포함하며, 안드로이드 리소스는 포함할 수 없습니다. 주로 Java 라이브러리에서 사용됩니다.
    * AAR: Android 전용 아카이브로, classes.jar 외에도 res/, AndroidManifest.xml, assets/ 등을 포함할 수 있어 Android 라이브러리 모듈 배포에 사용됩니다.
    * APK: Android Package로, 실제 앱을 설치할 수 있는 최종 패키지입니다. 실행 파일(.dex), 리소스, 서명 정보까지 모두 포함합니다.
  + AAR 파일은 어떤 경우에 사용하나요?
    * 안드로이드 앱에서 재사용 가능한 모듈을 분리해 라이브러리로 만들고, 이를 다른 프로젝트에 배포하거나 의존성으로 추가할 때 사용됩니다.
    * ex) 커스텀 버튼 UI, 공통 ViewModel, 인증 SDK 등을 AAR로 만들어 다른 앱에서 활용할 수 있습니다.
  + AAR과 JAR의 주요 차이점은 무엇인가요?
    * AAR은 Android 라이브러리 전용 형식으로, 코드뿐 아니라 리소스(res/), AndroidManifest, assets 등을 포함할 수 있습니다.
    * 반면 JAR은 Java 코드만 포함하므로, Android 앱에서 UI 구성이나 리소스를 사용하는 경우 적합하지 않습니다.


---
