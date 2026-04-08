### Bundle
- 정의
  + Key-Value 기반 데이터 저장 객체
  + Parcelable 구현 → 프로세스 간 통신(IPC)도 가능
  + Android의 Activity, Fragment, Service, BroadcastReceiver, ContentProvider 등 모든 주요 컴포넌트 간 데이터 교환에 사용됨

- 사용 예시
  + Activity <-> Activity
    * ```kotlin
      // 전송
      val intent = Intent(this, DetailActivity::class.java)
      val bundle = Bundle().apply {
          putString("name", "Bae")
          putInt("age", 30)
      }
      intent.putExtras(bundle)
      startActivity(intent)

      // 수신
      val name = intent.getStringExtra("name")
      val age = intent.getIntExtra("age", 0)
      ``` 
  + Fragment 초기 데이터 전달
    * ```kotlin
      val fragment = MyFragment().apply {
        arguments = Bundle().apply {
            putString("title", "Hello Fragment")
        }
      }

      // Fragment 내부
      val title = arguments?.getString("title")
      ``` 
  + 상태 저장/복원
    * ```kotlin
      override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("count", counter)
      }

      override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val restoredCount = savedInstanceState?.getInt("count") ?: 0
      }
      ``` 
  + Service, Broadcast, ContentProvider
    * Intent 내부 extras 로 전달 → 결국 Bundle이 사용됨 

- 특징
  + Key-Value 저장소
  + 다양한 타입 지원 (primitive, String, CharSequence, Parcelable, Serializable, ArrayList 등)
  + Parcelable 기반 직렬화 → 성능 좋음
  + 데이터 크기 제한 있음
  + 너무 큰 데이터 전달 시 TransactionTooLargeException 발생 (Binder 트랜잭션 한계, 보통 1MB 내외)
  + 대용량 데이터는 File, DB, SharedPreferences 등으로 처리 후 경로만 전달하는 방식 추천

- Bundle 사용 시 주의 사항
  + Null 안전성: get 계열 호출 시 항상 null 체크 or defaultValue 제공
  + Key 관리: 하드코딩 지양 → object Keys { const val USER_NAME = "user_name" }
  + 타입 일치: putInt() 했으면 반드시 getInt()로 꺼내야 함
  + Serializable vs Parcelable
    * Parcelable : 안드로이드 최적화된 직렬화, 빠름, 권장
    * Serializable : 구현 간단하지만 리플렉션 기반, 느림

- 면접 관련 질문
  + Bundle과 Intent의 차이는?
    * Intent는 컴포넌트 실행을 위한 "메시지 객체"
    * Bundle은 그 안에서 데이터를 담는 "데이터 컨테이너"
  + Serializable과 Parcelable 차이는?
    * Serializable: 자바 표준 직렬화, 느림, 리플렉션 기반
    * Parcelable: 안드로이드 전용 직렬화, 빠름, 직접 구현 필요
  + Bundle에 큰 데이터 넣으면 어떻게 되나요?
    * Binder 트랜잭션 한계(보통 1MB) 초과 시 TransactionTooLargeException 발생
    * 큰 데이터는 파일, DB, 캐시에 저장하고 식별자(id, uri, path)만 전달


---
