### AndroidManifest.xml
- 정의 
  + Android 앱의 핵심 구성 파일 입니다.
  + 앱에 대한 필수 정보를 Android 빌드 도구, Android 운영체제 및 Google Play에 제공합니다.
  + 모든 Android 엡 프로젝트의 루트 디렉터리에 반드시 존재해야 합니다.

- 주요 역할 및 내용
  + 애플리케이션 패키지 이름
    * 앱의 고유 식별자 입니다.
    * Java 또는 Kotlin의 패키지 이름 규칙을 따릅니다
    * Google Play Store에서 앱을 식별하는 데 사용되며, 한번 게시된 후에는 변경할 수 없습니다.
    * <manifest> 태그의 package 속성에 정의됩니다.
    
  + 애플리케이션 컴포넌트 선언
    * 앱을 구성하는 모든 핵심 컴포넌트(Activity, Service, BroadcastReceiver, ContentProvider)를 선언해야 합니다.
    * 각 컴포넌트는 <activity>, <service>, <receiver>, <provider> 태그를 사용하여 선언됩니다.
    * 컴포넌트의 이름, 속성 기능 등을 정의합니다.
    * ```xml
      <application>
            <activity android:name=".MainActivity" android:exported="true">
                <intent-filter>
                    <action android:name="android.intent.action.MAIN" />
                    <category android:name="android.intent.category.LAUNCHER" />
                </intent-filter>
            </activity>
            <service android:name=".MyBackgroundService" />
            <receiver android:name=".MyBroadcastReceiver" android:exported="false">
                <intent-filter>
                    <action android:name="android.intent.action.BOOT_COMPLETED"/>
                </intent-filter>
            </receiver>
            <provider
                android:name=".MyContentProvider"
                android:authorities="com.example.myapp.provider"
                android:exported="true" />
        </application>
      ```
      
  + 권한 선언
    * 앱이 시스템의 보호된 부분이나 다른 앱의 데이터에 접근하기 위해 필요한 권한을 선언합니다.
    * <uses-permission> 태그를 사용하여 요청합니다.
    * 인터넷 접근, 카메라 사용, 위치 정보 접근 등
    * ```xml
      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.CAMERA" />
      ```
      
  + 하드웨어 및 소프트웨어 기능 요구사항 선언
    * 앱이 정상적으로 작동하기 위해 필요한 하드웨어 또는 소프트웨어 기능을 명시합니다.
    * <users-feature>태그를 사용하여 선언 합니다.
    * 카메라 기능, 특정 센서 등
    * ```xml
      <uses-feature android:name="android.hardware.camera" android:required="true" />
      <uses-feature android:name="android.software.leanback" android:required="false" />
      ```
  
  + API 레벨 요구사항
    * 앱이 호환되는 최소 Android API 레벨, 타겟 API 레벨, 그리고 선택적으로 최대 API 레벨을 지정합니다.
    * <uses-sdk> 태그 내에 정의됩니다.
    * ```xml
      <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="33" />
      ```
      
  + 애플리케이션 메타데이터
    * 앱의 아이콘, 앱 이름, 테마 등 전역적인 애플리케이션 설정을 정의합니다.
    * <application> 태그의 속성으로 설정됩니다.
    * 추가적인 메타데이터를 <meta-data> 태그를 사용하여 키-값 쌍으로 제공할 수 있습니다.
    * 라이브러리나 특정 시스템 기능에 필요한 설정을 전달하는 데 사용됩니다.
    * ```xml
      <application
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
            <meta-data
                android:name="com.google.android.geo.API_KEY"
                android:value="YOUR_API_KEY"/>
        </application>
      ```

  + 인텐트 필터
    * 컴포넌트가 어떤 종류의 인텐트에 반응할 수 있는지 시스템에 알립니다.
    * <intent-filter> 태그 내에 <action>, <category>, <data> 요소를 사용하여 정의합니다.
    
- 면접 관련 질문
  + taskAffinity, launchMode 설정이 앱 동작에 어떤 영향을 주나요?
    * launchMode는 Activity의 인스턴스 생성 방식, 
    * taskAffinity는 액티비티가 어떤 태스크에 소속될지를 결정 합니다.
    
  + Android App Bundle 사용시 Manifest에서 주의할 점은 무엇인가요?
    * AAB는 기능 모듈별로 APK를 생성하는 방식이므로 Manifest도 각 모듈 별로 존재할 수도 있고, 병합 규칙에 따라 최종 Manifest가 결정됩니다.

  + allowBackup, fullBackupContent 설정의 보안적 의미는 무엇인가요?
    * android:allowBackup="true"가 설정되면 사용자가 adb로 앱 데이터를 백업/복원할 수 있고,
    * 민감한 데이터가 포함된 앱에서는 보안 위험이 될 수 있습니다.


---
