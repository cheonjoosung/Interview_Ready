### suspend 함수
- 개념 및 정의
  + `suspend` 함수 = **일시 중단 가능한 함수**
  + 코루틴 내에서만 호출 가능
  + **Thread는 차단하지 않고**, 함수 실행만 잠깐 멈췄다가 재개 가능
  + **네트워크 요청, DB 쿼리, delay** 등에서 주로 사용 

- 특징
  + | 항목                | 설명                                                            |
    |-------------------|-----------------------------------------------------------------|
    | 💤 중단 가능           | `delay()`, 네트워크/IO 작업 중 일시 정지 후 재개 가능                         |
    | 🧵 Non-blocking      | 스레드를 멈추지 않음 → **UI 스레드에서 안전**                                 |
    | 🔄 Resume 가능       | 중단된 지점부터 이어서 실행됨                                         |
    | 🚫 일반 함수에서 직접 호출 불가 | `launch`, `async` 같은 **코루틴 블록 내에서만 호출 가능**                 |

- 사용 예제
  + 기본
    * ```kotlin
      suspend fun fetchData(): String {
        delay(1000)  // 1초 대기 (Thread.sleep 아님!)
        return "데이터 완료"
      }

      lifecycleScope.launch {
        val result = fetchData()  // ✅ suspend 함수는 코루틴 안에서 호출
        println(result)
      }
      ```
  + Dispatcher 지정 (withContext)
    * ```kotlin
      suspend fun loadData(): String {
        return withContext(Dispatchers.IO) {
          // IO 작업 수행
          val data = File("file.txt").readText()
          data
        }
      }
      ```
    * withContext() 사용 이유:
      1. 작업에 적합한 스레드 풀에서 실행
      2. Main, IO, Default 등 명확히 분리 가능
  + 병렬 실행 (async)
    * ```kotlin
      suspend fun parallelWork() = coroutineScope {
        val task1 = async { fetchData1() }
        val task2 = async { fetchData2() }
        println("${task1.await()} + ${task2.await()}")
      }
      ```
    * async는 병렬 작업, await()는 결과 대기
    * launch는 결과 없는 단순 실행용

- 일반 함수와 suspend 함수 비교
  + | 항목        | 일반 함수                 | suspend 함수                            |
    | --------- | --------------------- | ------------------------------------- |
    | 중단 가능     | ❌                     | ✅ `delay()`, `IO`, `network` 등에서 일시정지 |
    | 스레드 차단 여부 | 차단 (ex. Thread.sleep) | 비차단 (non-blocking)                    |
    | 실행 환경     | 어디서든 호출 가능            | 코루틴 안에서만 호출 가능                        |
    | 비동기 처리 방식 | Callback, Thread 등 필요 | `async/await` 등으로 간단하게 구현 가능          |

- suspend 호출 위치 예시
  + | 컨텍스트              | 사용 방법                           |
    | ----------------- | ------------------------------- |
    | ViewModel         | `viewModelScope.launch { ... }` |
    | Activity/Fragment | `lifecycleScope.launch { ... }` |
    | 단순 진입점            | `runBlocking { ... }`           |
    | 테스트               | `runTest { ... }` (코루틴 테스트용)    |
 

- 면접 질문
  + suspend 함수란?
    * 일시 중단 가능한 함수입니다.
    * 코루틴 안에서 실행되어야 하며, 스레드는 차단하지 않습니다.
    * 비동기 처리에서 콜백 없이 순차 코드처럼 사용 가능합니다.
  + 일반 함수와 suspend 함수의 차이는 무엇인가요?
    * | 일반 함수      | suspend 함수     |
      | ---------- | -------------- |
      | 블로킹        | 논블로킹           |
      | 중단 불가      | 중단 가능          |
      | 어디서든 호출 가능 | 코루틴 내에서만 호출 가능 |
  + 왜 UI 스레드에서 suspend 함수가 유용한가요? 
    * `Thread.sleep()`과 달리 UI thread를 멈추지 않고 작업을 일시 중단하므로, 앱이 멈추지 않고 부드럽게 동작할 수 있습니다.
  + suspend 함수를 병렬로 실행하려면?
    * async {} 와 await() 사용
    * coroutineScope 내에서 실행


---
