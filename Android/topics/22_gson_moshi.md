### GSON & Moshi
- 직렬화(Serialization)
  + `직렬화`는 객체를 JSON, XML 등 문자열 형태로 변환하는 과정
  + `역직렬화(Deserialization)`는 문자열(JSON 등)을 다시 객체로 복원하는 과정
  + Android 앱에서는 API 통신 시 데이터를 객체 <-> JSON으로 변환할 일이 많은데 이럴 때 주로 쓰이는 것이 `Gson`, `Moshi` 같은 JSON 파서다.

- 공통점
  + | 항목 | 설명 |
    |------|------|
    | JSON <-> 객체 매핑 | 자동 지원 (Serialize/Deserialize) |
    | 어노테이션 지원 | `@SerializedName`, `@Json` 등 |
    | Null-safe 처리 가능 | 옵션 또는 어댑터로 제어 가능 |
    | Kotlin + Java 호환 | 모두 가능 |
    | Retrofit 연동 가능 | `converter-gson`, `converter-moshi`|

- 차이점
  + | 항목           | **Gson**                           | **Moshi**                           |
    |----------------|------------------------------------|-------------------------------------|
    | 개발 주체       | Google                             | Square (Retrofit 만든 회사)         |
    | Kotlin 지원     | 제한적 (Kotlin에 최적화 X)         | 우수함 (`KotlinJsonAdapterFactory`) |
    | 속도/성능       | 느린 편                            | 빠르고 가벼움                        |
    | Null 처리       | 더 관대함                          | 더 엄격함                            |
    | 타입 어댑터     | 복잡함 (`TypeToken` 필요)          | 심플하고 타입 안전함                |
    | 확장성          | 풍부한 기능, 오래된 레거시 지원    | 모던 설계, 가볍고 간결함            |
    | 빌드 사이즈     | 크다                               | 작다                                 |

- 샘플 코드 비교
  + GSON
    * ```kotlin
      data class User(
        @SerializedName("user_name") val name: String,
        val age: Int
      )

      val gson = Gson()
      val json = """{"user_name": "Lee", "age": 28}"""
      val user = gson.fromJson(json, User::class.java)
      val jsonOut = gson.toJson(user)
      ```
  + Moshi
    * ```kotlin
      @JsonClass(generateAdapter = true)
      data class User(
        @Json(name = "user_name") val name: String,
        val age: Int
      )

      val moshi = Moshi.Builder()
        .add(KotlinJsonAdapterFactory())
        .build()

      val adapter = moshi.adapter(User::class.java)
      val user = adapter.fromJson("""{"user_name": "Lee", "age": 28}""")
      val jsonOut = adapter.toJson(user)
      ```

- 선택 기준
  + | 상황                          | 추천                              |
    | --------------------------- | ------------------------------- |
    | Kotlin 프로젝트 (특히 data class) | ✅ Moshi                         |
    | 레거시 Java 프로젝트               | ✅ Gson                          |
    | 가볍고 빠른 파싱이 중요한 경우           | ✅ Moshi                         |
    | 기존 Gson 기반 코드 유지            | ✅ Gson (단, 필요시 Moshi 마이그레이션 고려) |

- 면접 질문
  + @SerializedName (GSON) / @Json (Moshi) 어노테이션은 어떤 역할을 하나요?
    * 이 어노테이션들은 JSON 데이터의 키 이름과 Kotlin/Java 객체의 필드(프로퍼티) 이름이 다를 경우 이를 매핑해주는 역할을 합니다.
  + JSON 데이터의 특정 필드가 null일 경우, GSON/Moshi는 어떻게 처리하나요? 기본값을 설정하는 방법이 있나요?
    * GSON은 기본적으로 필드 타입이 객체(Object) 타입이면 null로 설정합니다.
    * Moshi는 Kotlin data class의 프로퍼티에 기본값을 지정해두면, JSON에 해당 키가 없거나 값이 null일 때 그 기본값을 사용합니다.


---
