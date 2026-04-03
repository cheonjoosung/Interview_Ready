### Interceptor
- 정의
  + retrofit2는 안드로이드에서 가장 많이 쓰이는 HTTP 클라이언트 라이브러리이며, 내부적으로 OkHttp를 사용하는데 OkHttp의 핵심 기능 중 하나가 바로 **Interceptor**이고 기능을 그대로 사용
  + Interceptor는 모든 네트워크 요청/응답을 **가로채서 가공하거나 검사할 수 있는 훅(Hook)** 이자 인터페이스**

- 특징
  + | 특징                        | 설명 |
    |-----------------------------|------|
    | 요청/응답 모두 개입 가능     | 헤더 추가, 응답 처리 등 |
    | 체인 구조                   | 여러 Interceptor를 연결 가능 (`chain.proceed()`) |
    | 순차 실행                  | 등록된 순서대로 실행됨 |
    | 인증 처리 용이              | JWT, OAuth 토큰 등 자동 삽입 가능 |
    | 로그, 디버깅에도 활용 가능   | OkHttpLoggingInterceptor |
    | 캐싱, 리트라이 커스터마이징 가능 | 고급 제어 가능 |

- 기본 구조
  + ```kotlin
    class AuthInterceptor(private val tokenProvider: () -> String) : Interceptor {
      override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
            .newBuilder()
            .addHeader("Authorization", "Bearer ${tokenProvider()}")
            .build()
        return chain.proceed(request)
      }
    }
    ```
  + 이 코드는 모든 요청에 Authorization 헤더를 자동으로 추가

- 샘플 코드
  + Retrofit + Interceptor 설정
    * ```kotlin
      val client = OkHttpClient.Builder()
        .addInterceptor(AuthInterceptor { getAccessToken() })
        .addInterceptor(HttpLoggingInterceptor().apply {
          level = HttpLoggingInterceptor.Level.BODY
        })
        .build()

      val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com")
        .client(client)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
      ```
  + 토큰 만료 시 자동 갱신
    * ```kotlin
      class TokenRefreshInterceptor(...) : Interceptor {
        override fun intercept(chain: Interceptor.Chain): Response {
            val request = chain.request()
            val response = chain.proceed(request)

            if (response.code == 401) {
                synchronized(this) {
                    val newToken = refreshAccessToken()
                    val newRequest = request.newBuilder()
                        .header("Authorization", "Bearer $newToken")
                        .build()
                    return chain.proceed(newRequest)
                } 
            }

           return response
        }
      }
      ```
    * ️만료된 토큰을 자동으로 갱신할 수 있음
    * 모든 API 호출이 통합된 방식으로 보안 처리됨

- 결론
  + | 용도          | Interceptor 활용 예                     |
    | ----------- | ------------------------------------ |
    | 인증 토큰 자동 추가 | 헤더에 Authorization 삽입                 |
    | 요청 공통 헤더    | User-Agent, Locale, App-Version 등 삽입 |
    | 응답 가공       | 응답 코드에 따른 처리                         |
    | 로그 기록       | `HttpLoggingInterceptor` 사용          |
    | 토큰 갱신       | `401` 응답 시 자동 재요청                    |
  + Interceptor는 네트워크의 관문이다. 요청과 응답 모두를 지켜보며 수정, 검사, 자동화할 수 있다. Retrofit을 쓰면 반드시 함께 써야 할 고급 툴

- 면접 관련 질문
  + Interceptor는 언제, 왜 사용하는가요?
    * Interceptor는 네트워크 요청이나 응답을 가로채서 공통 작업을 처리하거나 수정할 수 있도록 해주는 구조입니다.
      주로 다음과 같은 경우에 사용합니다:
      1. 모든 요청에 공통 헤더 추가 (예: Authorization, App-Version 등)
      2. 인증 토큰이 만료되었을 때 자동으로 갱신 처리
      3. API 요청/응답 로깅 및 디버깅
      4. 캐시 정책, 네트워크 리트라이 제어 등
    * 이런 공통 처리를 각 API마다 반복하지 않고 Interceptor로 한 곳에서 제어할 수 있어서, 유지보수성이 뛰어납니다.
  + addInterceptor()와 addNetworkInterceptor()의 차이는 무엇인가요?
    * addInterceptor()는 전체 요청/응답을 가로챌 수 있는 애플리케이션 레벨의 인터셉터입니다.
      1. 요청이 캐시에서 처리되든 네트워크로 가든 모두 동작합니다.
    * addNetworkInterceptor()는 오직 네트워크를 통해 실제 서버와 통신할 때만 동작합니다.
      1. 캐시에서 응답이 반환될 경우, 호출되지 않습니다.
    * 실무에서는 대부분 addInterceptor()를 사용하며, 네트워크 상세 조작이 필요할 때만 addNetworkInterceptor()를 사용합니다.
  + 토큰 인증이 필요한 앱에서 Interceptor를 어떻게 활용해봤나요?
    * Interceptor를 활용해서 모든 API 요청에 자동으로 Bearer 토큰을 추가했습니다.
    * 또한, 서버에서 401 응답을 받았을 때는 Interceptor 내에서 토큰을 자동 갱신하고 원래 요청을 재시도하는 로직도 구현했습니다.
    * 이 구조 덕분에 API 호출부에서는 인증 처리 코드를 신경 쓸 필요가 없었고, 보안 로직을 중앙 집중화할 수 있었습니다.
  + Interceptor에서 무한 루프나 요청 중복 문제가 발생할 수 있는 경우는 언제인가요?
    * Interceptor에서 401 처리 후 토큰을 갱신하고 요청을 다시 보낼 때,
      → 재시도 로직이 반복적으로 호출되면 무한 루프가 발생할 수 있습니다.
    * 이걸 방지하려면 다음과 같은 방어 로직을 넣습니다:
      1. 새로 보낸 요청에는 토큰 갱신 플래그를 붙이거나,
      2. 재귀 요청 횟수를 제한하거나,
      3. chain.request()의 URL이나 Tag로 이미 재시도했는지 확인
    * 이런 방어 코드를 통해 네트워크 폭주나 앱 크래시를 막을 수 있습니다.
  + Retrofit과 OkHttp의 관계는 무엇인가요?
    * Retrofit은 고수준 HTTP 클라이언트이며, 내부적으로 OkHttp를 기반으로 동작합니다.
      1. 요청 전송, 응답 수신, 캐싱, 연결 풀링 등 낮은 수준의 네트워크 처리는 모두 OkHttp가 수행하고,
      2. Retrofit은 이를 래핑해서 인터페이스 기반 API 호출, Gson 파싱, 코루틴 연동 같은 기능을 제공합니다.
    * Interceptor는 Retrofit이 아니라 OkHttp의 기능이며, Retrofit에서 OkHttpClient를 설정해줄 때 함께 구성합니다.


---
