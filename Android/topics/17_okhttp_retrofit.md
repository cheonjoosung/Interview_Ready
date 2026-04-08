### okHttp와 Retrofit 차이
- 정의
  + Android 및 Java 애플리케이션에서 네트워킹을 처리하는 데 널리 사용되는 라이브러리이지만, 서로 다른 계층에서 작동하며 다른 목적을 가지고 있습니다.

- OkHttp : HTTP 클라이언트
  + 역할
    * OkHttp는 HTTP 및 HTTP/2 클라이언트 입니다.
    * 실제 네트워크 요청을 만들고 응답을 받는 저수준(low-level) 라이브러리입니다.
  + 주요 기능
    * HTTP GET, POST, PUT, DELETE 등 다양한 요청 메서드 지원
    * 요청/응답 헤더 및 본문 처리
    * 연결 풀링을 통한 성능 최적화
    * Gzip 압축을 통한 데이터 크기 감소
    * 응답 캐싱
    * 타임아웃 설정
    * TSL/SSL 보안 연결 지원
    * HTTP/2 및 SPDY 프로토콜 지원
    * 자동 재시도 및 리디렉션 처리
    * WebSocket 지원
  + 사용 방식 : 개발자가 직접 Request 객체를 만들고, OkHttpClient를 사용하여 이를 실행하고, Response 객체를 받아 처리해야 합니다.
    ```kotlin
    // OkHttp 사용 예시
    val client = OkHttpClient()

    val request = Request.Builder()
        .url("https://api.example.com/data")
        .header("Authorization", "Bearer your_token")
        .build()

    client.newCall(request).enqueue(object : Callback {
        override fun onFailure(call: Call, e: IOException) {
            // 오류 처리
        }

        override fun onResponse(call: Call, response: Response) {
            if (response.isSuccessful) {
                val responseBody = response.body?.string()
                // 응답 처리 (예: JSON 파싱)
            } else {
                // 서버 오류 처리
            }
        }
    })
    ```
  + 장점
    * 매우 효율적이고 강력한 HTTP 통신 기능 제공
    * 세밀한 요청/응답 제어 가능
    * 다양한 네트워크 최적화 기능 내장
  + 단점
    * 상대적으로 저수준 API이므로, 반복적인 코드 작성이 필요할 수 있음
    * API 엔드포인트가 많아질수록 코드 관리가 복잡해질 수 있음

- Retrofit : 타입-세이프 HTTP 클라이언트
  + 역할
    * Retrofit은 타입-세이프한 REST 클라이언트입니다. 즉, HTTP API를 Java/Kotiln 인터페이스로 변환하여 사용하기 쉽게 만들어주는 고수준 라이브러리입니다.
  + 기반
    * Retrofit은 내부적인 OkHttp를 HTTP 클라이언트로 사용합니다. 즉 Retrofit은 OkHttp 위에 구축된 추상화 계층입니다.
  + 주요 기능
    * 인터페이스 기반 API 정의 : 어노테이션을 사용하여 HTTP 요청을 인터페이스 메서드로 매핑합니다.
    * 자동 직렬화/역직렬화 : 요청 본문과 응답 본문을 Java/Kotlin 객체로 자동으로 변환해줍니다.
    * 동기/비동기 요청 지원 : Kotlin 코루틴, RxJava, Call 객체를 통한 비동기 처리 지원
    * URL 파라미터, 쿼리 파라미터, 요청 본문 등을 어노테이션으로 쉽게처리
    * 멀티파트 요청지원
  + 사용 방식 : 개발자는 API 엔드포인트를 정의하는 인터페이스를 만들고, Retrofit 빌더를 사용하여 해당 인터페이스의 구현체를 생성합니다.
    ```kotlin
    // Retrofit 인터페이스 정의 예시
    interface ApiService {
        @GET("data")
        suspend fun getData(@Header("Authorization") token: String): MyData // MyData는 데이터 모델 클래스
    }

    // Retrofit 사용 예시
    val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .client(OkHttpClient()) // 선택 사항: 커스텀 OkHttpClient 설정 가능
        .addConverterFactory(GsonConverterFactory.create()) // JSON <-> 객체 변환기
        .build()

    val apiService = retrofit.create(ApiService::class.java)

    // 코루틴 내에서 호출
    viewModelScope.launch {
        try {
            val myData = apiService.getData("Bearer your_token")
            // 성공 처리
        } catch (e: Exception) {
            // 오류 처리
        }
    }
    ```
  + 장점
    * API 호출 코드를 매우 간결하고 직관적으로 작성 가능
    * 타입-세이프 : 컴파일 시점에 API 정의 오류를 발견할 수 있음
    * JSON/XML 등의 데이터 변환을 자동으로 처리하여 보일러플레이트 코드 감소
    * OkHttp의 강력한 네트워킹 기능을 그대로 활용 가능
  + 단점
    * OkHttp에 대한 의존성을 가짐
    * 매우 세밀한 저수준 제어가 필요한 경우 OkHttp를 직접 사용하는 것보다 유연성이 떨어질 수 있음
- OkHttp와 Retrofit의 관계 및 차이점 요약
  + | 특징       | OkHttp                    | Retrofit              |
    |----------|---------------------------|-----------------------|
    | 계층       | 저수준 HTTP 클라이언트            | 고수준 REST 클라이언트        | 
    | 주요목적     | HTTP 요청/응답처리, 네트워크 최적화    | API엔드포인트를 인터페이스로 추상화  |
    | 사용방식     | Request.Response 객체 직접 처리 | 어노테이션 기반 인터페이스 정의, 자동 객체 매핑 |
    | 코드 복잡도   | 상대적으로 높음                  | 상대적으로 낮음 |
    | 데이터 변환   | 수동처리                      | 자동처리 |
    | 의존성      | 독립성                       | OkHttp에 의존 |

- 면접 관련 질문
  + Retrofit은 OkHttp와 어떤 관계를 가지고 있나요?
    * Retrofit은 OkHttp 위에 구축된 추상화 계층입니다.
      Retrofit은 내부적으로 OkHttp를 HTTP클라이언트로 사용합니다. Retrofit은 OkHttp의 강력한 네트워킹 기능을 활용하면서 개발자가 더 쉽고 간결하게 API를 호출할 수 있도록 도와줍니다.
  + OkHttp를 "저수준(low-level) HTTP 클라이언트"라고 하고 Retrofit을 "고수준(high-level) REST 클라이언트"라고 부르는 이유는?
    * 저수준 HTTP 클라이언트는 개발자가 직접 Request 객체를 만들고, OkHttpClient를 사용하여 이를 실행하고, Response 객체를 받아 헤더나 본문을 직접 파싱하는 등 HTTP 통신의 세부적인 부분을 다루기 때문입니다.
    * 고수준 HTTP 클라이언트는 HTTP API의 엔드포인트, 파라미터, 요청/응답 타입 등을 인터페이스와 어노테이션으로 추상화합니다. 개발자는 실제 HTTP 요청 생성이나 응답 파싱의 복잡한 과정을 직접 다루지 않고, 마치 일반적인 메소드를 호출하듯이 API를 사용할 수 있게 해주기 때문입니다.
  + Retrofit의 가장 큰 장점 중 하나인 "타입-세이프(type-safe)"하다는 것은 어떤 의미인가요?
    * Retrofit은 API 인터페이스를 정의할 때 요청 파라미터의 타입과 응답 객체의 타입을 명시합니다. 
      이를 통해 컴파일 시점에 파라미터 타입 불일치나 잘못된 응답 타입 사용 등의 오류를 미리 발견할 수 있다는 의미입니다.
    * 이로 인해 잘못된 타입으로 인한 런타임 예외 발생 가능성을 줄여주고, API 명세가 코드로 명확하게 표현되므로, API 사용 방법을 이해하기 쉽고 실수를 줄일 수 있습니다.


---
