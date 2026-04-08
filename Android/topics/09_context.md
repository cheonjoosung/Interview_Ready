### Context
- 개념 및 정의
  + Context = "현재 애플리케이션/컴포넌트가 어떤 환경에서 실행 중인지에 대한 핸들"
  + 즉, OS와 앱 사이의 다리 역할
    * 시스템 서비스 접근 (getSystemService)
    * 리소스 접근 (getResources)
    * 컴포넌트 실행 (startActivity, startService, sendBroadcast) 의 현재 상태와 환경에 대한 정보를 담고 있는 객체
  
- Context 종류
  + | Context 종류            | 설명                   | 대표 사용 예                                |
    | --------------------- | -------------------- | -------------------------------------- |
    | **Application**       | 앱 전역에서 하나            | DB, SharedPreferences, 전역 Toast        |
    | **Activity**          | UI 관련, 화면 단위         | 뷰 inflate, 다이얼로그, 테마 적용                |
    | **Service**           | 백그라운드 실행 환경          | 음악 재생, 위치 추적                           |
    | **BroadcastReceiver** | 브로드캐스트 수신 시 일시적으로 제공 | 알람 수신 후 동작 실행                          |
    | **ContextWrapper**    | Context 기능을 감싸서 확장   | `ContextThemeWrapper` (테마 적용된 Context) |
   
- Application vs Activity Context
  + Application Context
    * 전역적, 오래 살아남음
    * UI 관련 기능은 ❌ (예: Dialog 띄우기 불가 → Theme 없음)
  + Activity Context
    * 화면 단위로 UI 포함
    * 액티비티가 끝나면 함께 소멸해야 함
  + ❗실수 사례
    * 싱글톤 객체에 Activity Context를 저장하면 → Activity가 해제되지 않고 메모리 누수 발생

- context 사용
  + | 할 일            | 예시                                                         |
    |------------------|--------------------------------------------------------------|
    | 리소스 접근       | `getString(R.string.app_name)`                               |
    | 뷰 생성          | `LayoutInflater.from(context).inflate(...)`                  |
    | 파일 읽기/쓰기    | `context.openFileInput("file.txt")`                          |
    | 토스트 띄우기     | `Toast.makeText(context, "Hello", Toast.LENGTH_SHORT)`       |
    | 다른 액티비티 시작 | `context.startActivity(intent)`                              |
    | 서비스 시작       | `context.startService(...)`                                  |
 
- 면접 관련 질문
  + Application Context와 Activity Context 차이는?
    * Application: 앱 전역, UI 불가, 수명 길다
    * Activity: UI 포함, Activity 생명주기와 동일
  + 왜 Application Context로 Dialog를 띄우면 안 되나요?
    * Application Context에는 Theme/Window 정보 없음
    * UI 관련 기능은 반드시 Activity Context 사용해야 함
  + Context 메모리 누수는 언제 발생하나요?
    * Activity Context를 싱글톤/전역 static 변수에 저장했을 때
    * Activity 종료 후에도 GC가 수거하지 못해 메모리 누수 발생
  + getApplicationContext()와 this 차이는?
    * this: 현재 Activity Context
    * applicationContext: 앱 전역 Application Context  


---
