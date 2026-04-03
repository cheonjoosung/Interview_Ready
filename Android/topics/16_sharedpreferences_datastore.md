### SharedPreferences & DataStore
- SharedPreferences
  + 작은 양의 key-value 형태 데이터를 로컬에 저장할 수 있도록 제공되는 기본 저장 API
  + 주로 로그인 정보, 앱 설정, 토글 상태 등 간단한 데이터를 저장하는데 사용
  + 핵심특징
    * XML 파일 기반 저장
    * 동기적 or제한적인 비동기 방식 (apply())
    * 앱 재시작 후에도 데이터 유지됨 (영속성)
    * Thread-safe 보장 없기에 다중 접근 시 위험
    * 복잡한 데이터 구조 저장에 부적합
  + Sample Code
    * ```kotlin
      // 데이터 쓰기
      val prefs = context.getSharedPreferences("my_prefs", Context.MODE_PRIVATE)
      prefs.edit {
        putString("userName", "james")
        .putBoolean("isLogin", true)
      }
      
      // 데이터 읽기
      val username = prefs.getString("userName", null)
      val isLoggedIn = prefs.getBoolean("isLogin", false)
      ```
  + apply() vs commit()
    * `apply()` → 비동기, 결과 반환 없음 (권장)
    * `commit()` → 동기, boolean 반환 (성공 여부 확인 가능)
    * UI 스레드에서 commit() 호출 시 ANR 위험
  + Migration 이슈
    * 앱 데이터가 많거나 빈번히 쓰기 작업이 있을 경우 XML 파싱 지연 발생 가능
    * SharedPreferences는 내부적으로 하나의 파일(XML)에 모든 데이터를 저장하므로, 데이터 크기가 커질수록 성능 저하  

- DataStore
  + Jetpack에서 제공하는 최신 데이터 저장 솔루션으로, 비동기 처리와 타입 안전성을 갖춘 Key-Value 저장 방식
  + SharedPreferences의 단점을 보완하고, Flow 및 Coroutine을 기반으로 설계
  + 핵심 특징
    * 코루틴 기반 비동기 저장
    * Flow 통한 실시간 데이터를 감지 및 수집
    * 두 종류 제공
      1. `Preferences DataStore`: Key-Value 저장
      2. `Proto DataStroe`: 구조화된 커스텀 객체 저장 (protobuf 기반)
    * 스레드 안전 + ANR 방지
    * JetPack 공식 권장 방식
  + Sample Code
    * 의존성 추가 `implementation("androidx.datastore:datastore-preferences:1.0.0")`
    * ```kotlin
      class UserPreferencesRepository(private val context: Context) {
        // 초기 설정
        private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "user_settings")

        // 키 정의
        object UserPreferencesKeys {
          val USERNAME = stringPreferencesKey("userName")
          val IS_LOGIN = booleanPreferencesKey("isLogin")
        }

        val userData: Flow<Pair<String?, Boolean>> = context.dataStore.data
          .map { prefs ->
            val username = prefs[UserPreferencesKeys.USERNAME]
            val isLoggedIn = prefs[UserPreferencesKeys.IS_LOGIN] ?: false
            username to isLoggedIn
        }

        suspend fun saveUser(username: String, isLoggedIn: Boolean) {
          context.dataStore.edit { prefs ->
            prefs[UserPreferencesKeys.USERNAME] = username
            prefs[UserPreferencesKeys.IS_LOGIN] = isLoggedIn
          }
        }

        suspend fun clearUser() {
          context.dataStore.edit { prefs ->
            prefs.clear()
          }
        }
      }
      ```
  + Preferences vs Proto
    * Preferences DataStore → 단순 Key-Value (SharedPreferences 대체)
    * Proto DataStore → 구조화된 데이터 저장 (스키마 기반, 타입 안전 보장
  + Migration 지원
    * 기존 SharedPreferences 데이터를 DataStore로 마이그레이션 가능
    * SharedPreferencesMigration(context, "prefs_name")
  + 실시간 반응
    * Flow 기반 → 데이터 변경 시 UI 자동 업데이트 가능 (collectLatest) 

- 비교
  + | 항목         | SharedPreferences  | DataStore                            |
    | ---------- | ------------------ | ------------------------------------ |
    | 저장 구조      | XML 파일 (Key-Value) | Preferences (Key-Value) / Proto (객체) |
    | API 방식     | get/put 직접 호출      | Flow + suspend 함수                    |
    | 저장/읽기      | 동기 + 제한적 비동기       | 완전 비동기 (Coroutine 기반)                |
    | 스레드 안전성    | ❌ 보장 안 됨           | ✅ 보장됨                                |
    | 데이터 타입 안전성 | ❌ 없음               | ✅ (특히 ProtoDataStore)                |
    | 성능/확장성     | 데이터 많아지면 느려짐       | 대량 데이터에도 효율적                         |
    | 권장 여부      | 레거시 유지용            | ✅ Jetpack 공식 권장                      |


- 면접 관련 질문
  + SharedPreferences에서 ANR이 발생할 수 있는 상황은?
    * 메인 스레드에서 commit() 호출 시 발생
    * 큰 데이터를 자주 저장할 때 XML 파싱이 오래 걸리면 위험
  + DataStore에서 Flow를 쓰는 이유는?
    * 데이터 변경 시 자동으로 UI 업데이트 반영 가능
    * Reactive 패턴에 적합 → MVVM, LiveData/StateFlow와 궁합 좋음
  + DataStore Proto를 써야 하는 상황은?
    * 데이터 구조가 복잡하고 타입 안정성이 중요한 경우
    * 예: 사용자 설정(UserSettings), 앱 환경설정, 다중 필드 상태 저장


---
