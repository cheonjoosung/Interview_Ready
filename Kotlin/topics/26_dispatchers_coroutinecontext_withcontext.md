### Dispatchers & CoroutineContext & withContext
- Dispatchers
  + 코루틴이 어디서 실행될지 정하는 장치
  + | Dispatcher            | 용도/특징                                                                 |
    |----------------------|-------------------------------------------------------------------------|
    | `Dispatchers.Default` | CPU 작업 (정렬, 계산 등). 코어 수 만큼의 스레드 풀 사용                             |
    | `Dispatchers.IO`      | I/O (네트워크, DB, 파일). 많은 동시 작업 처리를 위한 스레드 풀                          |
    | `Dispatchers.Main`    | Android UI. **UI 업데이트는 반드시 여기서 실행**                                |
    | `Dispatchers.Unconfined` | 호출한 스레드에서 시작, 일시 중단 이후에는 재개 위치가 예측되지 않음 → 특수한 경우에만 사용 |
  + 예시
    * ```kotlin
      launch(Dispatchers.Default) { doHeavyCalculation() }
      launch(Dispatchers.IO) { val data = fetchFromDb() }
      launch(Dispatchers.Main) { textView.text = "UI 업데이트" }
      ```
     
- CoroutineContext
  + 코루틴이 실행될때 사용되는 환경정보의 집합체
  + | 요소                          | 설명                   |
    | --------------------------- | -------------------- |
    | `Dispatcher`                | 어떤 스레드/풀에서 실행할지 설정   |
    | `Job`                       | 취소, 부모-자식 코루틴 트리 구성용 |
    | `CoroutineName`             | 디버깅용 이름              |
    | `CoroutineExceptionHandler` | 예외 처리용 핸들러           |
  + 예시
    * ```kotlin
      val context = Dispatchers.IO + CoroutineName("UserFetcher")
      launch(context) {
        println(coroutineContext[CoroutineName]) // "UserFetcher"
      }
      ```
     
- withContext
  + suspend 함수 내에서 컨텍스트 전환
  + | 항목         | launch                     | withContext                |
    | ---------- | -------------------------- | -------------------------- |
    | 목적         | 병렬 작업 실행 (fire-and-forget) | **컨텍스트 전환 후 결과 반환**        |
    | 반환값        | 없음 (`Job`)                 | 있음 (`T`)                   |
    | suspend 필요 | ❌                          | ✅ `suspend` 함수 내부에서만 사용 가능 |
    | 대표 쓰임      | UI 작업, 로그, 알림 등            | IO 처리 후 UI 복귀, 계산 후 전환 등   |
  + 예시
    * ```kotlin
      suspend fun loadFile(): String {
        return withContext(Dispatchers.IO) {
          File("data.txt").readText()
        }
      }

      // 복합 예시 (IO -> UI)
      viewModelScope.launch {
        val result = withContext(Dispatchers.IO) { fetchNetworkData() }
        withContext(Dispatchers.Main) { updateUI(result) }
      }
      ```
- 요약 정리
  + | 개념                 | 요약 설명                                                   |
    | ------------------ | ------------------------------------------------------- |
    | `Dispatcher`       | 코루틴이 **어느 스레드/풀에서 실행될지 지정**                             |
    | `CoroutineContext` | 실행 환경 전체 (Dispatcher, Job, Name, ExceptionHandler 등 포함) |
    | `withContext`      | suspend 함수 안에서 **작업별로 컨텍스트를 분리 실행**하고 결과 반환             |
 
- 면접 관련 질문
  + `withContext 와 launch의 차이는 무엇인가요?
    * `launch` : `Job` 반환, 병렬 실행, 반환값 없음
    * `withContext` : `T` 반환, context 전환 + suspend 필요, 순차 실행
  + UI 스레드에서 무거운 연산을 하면 어떤 문제가 발생하나요? 해결방안은 무엇인가요?
    * 문졔: UI 스레드에서 무거운 연산을 하면 ANR(Application Not Responding)이 발생
    * Dispatchers.Default 또는 Dispatchers.IO로 context를 전환하여 백그라운드에서 작업하고, UI 업데이트 시에는 다시 Dispatchers.Main으로 돌아와야 합니다.
    * ```kotlin
      viewModelScope.launch {
        val result = withContext(Dispatchers.Default) { doHeavyWork() }
        withContext(Dispatchers.Main) { updateUI(result) }
      }
      ```
  + Coroutine에서 Dispatcher란 무엇인가요?
    * 코루틴이 어떤 스레드 또는 스레드 풀에서 실행될지 결정하는 요소
    * 대표적으로: 
      1. Dispatchers.Main: UI
      2. Dispatchers.IO: 디스크/네트워크
      3. Dispatchers.Default: CPU 작업
  + withContext는 언제 쓰면 좋은가요?
    * IO → UI처럼 컨텍스트가 명확히 나뉘는 작업
    * 결과가 필요한 작업
    * 동시성이 아닌 순차 처리가 필요한 비동기 로직


---
