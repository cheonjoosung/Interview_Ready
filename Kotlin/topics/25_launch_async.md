### 코루틴 빌더 (runBlocking, launch, async)
- 정의
  + **코루틴 빌더** = 코루틴을 시작하는 진입점 함수
  + 대표: `runBlocking`, `launch`, `async`
  + 모두 내부적으로 새로운 `Job`을 생성하여 코루틴 실행

- runBlocking 빌더
  + | 항목         | 내용                                 |
    |------------|------------------------------------|
    | 목적         | **블로킹 방식**으로 코루틴 실행             |
    | 사용 위치      | `main()`, 테스트 코드 등 진입점               |
    | UI 스레드 사용 | ❌ 금지 (ANR 발생 가능)                   |
    | 리턴 타입      | T (마지막 표현식)                        |
    | 특징         | 블로킹 스코프, 내부 코루틴이 끝날 때까지 현재 스레드 대기 |
  + ```kotlin
    fun main() = runBlocking {
      println("runBlocking start")
          launch {
              delay(1000)
              println("done inside launch")
          }
      println("runBlocking end")
    }
    ```

- launch 빌더
  + | 항목    | 내용                                |
    | ----- | --------------------------------- |
    | 목적    | 결과 없이 **작업 실행** (fire-and-forget) |
    | 리턴 타입 | `Job` (취소, 완료 추적용)                |
    | 사용 예  | UI 업데이트, 로그, 애니메이션 등              |
    | 실행 방식 | 비동기, 블로킹 없음                       |
  + ```kotlin
    GlobalScope.launch {
      delay(1000)
      println("World!")
    }
    Thread.sleep(2000) // main이 먼저 끝나지 않게 대기
    ```

- async 빌더
  + | 항목    | 내용                          |
    | ----- | --------------------------- |
    | 목적    | **결과 반환이 필요한 비동기 작업** 수행    |
    | 리턴 타입 | `Deferred<T>`               |
    | 결과 받기 | `await()` 사용                |
    | 사용 예  | 계산 작업, API 호출 등 병렬 결과 필요할 때 |
  + ```kotlin
    runBlocking {
      val a = async { delay(1000); "A" }
      val b = async { delay(1500); "B" }
      println("${a.await()}, ${b.await()}")
    }
    ```

- async + awaitAll()
  + ```kotlin
    val result = listOf(
      async { delay(300); "One" },
      async { delay(200); "Two" },
      async { delay(100); "Three" }
    ).awaitAll()

    println(result) // ["One", "Two", "Three"]
    ```  
  + `awaitAll()`은 여러 결과를 동시에 기다리고, 첫 예외 발생 시 나머지를 자동 취소해줌.

- 비교 정리
  + | 항목           | launch           | async                       | runBlocking        |
    | ------------ | ---------------- | --------------------------- | ------------------ |
    | 결과 반환        | ❌ 없음             | ✅ `Deferred<T>` → `await()` | ❌ (내부에서 변수로 반환 가능) |
    | 블로킹 여부       | ❌ (비차단)          | ❌ (비차단)                     | ✅ 현재 스레드 블로킹       |
    | 대표 용도        | UI 작업, 로그 등      | API, 계산 결과 등                | 테스트, main 진입점      |
    | 코루틴 범위 설정 가능 | ✅ CoroutineScope | ✅ CoroutineScope            | 자체가 Scope          |

- 면접 관련 질문
  + 코루틴 빌더 중 launch와 async의 차이점은?
    * `launch`는 결과 없이 작업 실행 (`Job`)
    * `async`는 결과 반환 (`Deferred`) → `await()`로 결과 수신
  + `runBlocking`을 언제 쓰나요?
    * `main()` 함수, 테스트 진입점에서만 사용
    * 스레드를 블로킹하므로 UI에서는 절대 사용 금지
  + `async` 병렬 사용하려면?
    * ```kotlin
      val a = async { workA() }
      val b = async { workB() }
      val result = a.await() + b.await()
      ```
  + `awaitAll()`과 `joinAll()`의 차이는?
    * awaitAll()은 여러 Deferred를 한꺼번에 기다리면서, 코드가 깔끔하고 직관적으로 작성되도록 해줍니다.
    * 또한 예외가 발생할 경우, 첫 번째 예외를 즉시 전달하고 나머지 Deferred는 자동으로 취소되기 때문에 안정적인 병렬 처리를 보장할 수 있습니다.


---
