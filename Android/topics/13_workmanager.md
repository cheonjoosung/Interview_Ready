### WorkManager
- 정의
  + 안드로이드에서 백그라운드 작업을 보장하기 위한 Jetpack 라이브러리
  + 앱이 종료되거나 기기가 재부팅돼도 실행이 보장
  + 내부적으로 OS 버전에 맞게 `JobScheduler`, `AlarmManager`, `Firebase JobDispatcher` 등을 활용 → 앱 개발자는 API 하나만 사용하면 됨

- 사용 시점
  + | 상황            | 설명                         |
    | ------------- | -------------------------- |
    | **보장된 실행 필요** | 앱 종료, 재부팅 후에도 작업 실행 보장     |
    | **네트워크 필요**   | Wi-Fi / 데이터 연결 필요할 때       |
    | **주기적 동기화**   | 최소 15분 이상의 주기 작업           |
    | **조건 기반 실행**  | "충전 중일 때만", "네트워크 연결 시만" 등 |

- 특징
  + | 기능                                | 설명                     |
    | --------------------------------- | ---------------------- |
    | ✅ 작업 보장                           | 앱이 종료돼도, 재부팅돼도 실행됨     |
    | ✅ 제약 조건 지원                        | Wi-Fi, 충전 중일 때 등 설정 가능 |
    | ✅ JobScheduler, AlarmManager 등 통합 | OS 버전에 따라 자동 내부 분기     |
    | ✅ Chaining 지원                     | 작업 순서 정의 가능 (`then()`) |
    | ✅ Coroutine & RxJava 호환           | 코루틴 기반 작업 작성 가능        |

- Work 종류
  + OneTimeWorkRequest → 한 번 실행
  + PeriodicWorkRequest → 주기 실행 (최소 15분 이상, 정확한 시간 보장은 X → 최적화 고려)

- Worker 종류
  + | 클래스               | 특징                          |
    | ----------------- | --------------------------- |
    | `Worker`          | 기본 동기 실행                    |
    | `CoroutineWorker` | 코루틴 기반 (suspend fun doWork) |
    | `RxWorker`        | RxJava 기반                   |
  + ```kotlin
    class MyCoroutineWorker(appContext: Context, params: WorkerParameters) 
      : CoroutineWorker(appContext, params) {
        override suspend fun doWork(): Result {
            delay(1000) // 비동기 안전
            return Result.success()
      }
    }

    ```  

- Work 상태 관리
  + WorkInfo 객체로 상태 추적 가능
  + 상태: ENQUEUED, RUNNING, SUCCEEDED, FAILED, CANCELLED 
  + ```kotlin
    WorkManager.getInstance(context)
      .getWorkInfoByIdLiveData(workRequest.id)
      .observe(this) { workInfo ->
        if (workInfo.state == WorkInfo.State.SUCCEEDED) {
            Log.d("Work", "작업 완료")
        }
    }
    ```

- 데이터 전달
  + InputData → Worker에 값 전달
  + OutputData → Worker에서 결과 반환
  + ```kotlin
    // 전달
    val data = workDataOf("key" to "value")
    val work = OneTimeWorkRequestBuilder<MyWorker>()
      .setInputData(data)
      .build()

    // Worker 내부
    override fun doWork(): Result {
      val value = inputData.getString("key")
      val output = workDataOf("result" to "done")
      return Result.success(output)
    }
    ```

- WorkManager vs JobScheduler vs AlarmManager
  + | 항목       | WorkManager       | JobScheduler    | AlarmManager |
    | -------- | ----------------- | --------------- | ------------ |
    | 실행 보장    | ✅ 앱 종료/재부팅 후에도 실행 | ✅ (Lollipop 이상) | ❌ 보장 안 됨     |
    | 제약조건 지원  | ✅ 충전, 네트워크 등 가능   | ✅               | ❌ (시간만 가능)   |
    | 최소 주기    | 15분               | 15분             | 초 단위 가능      |
    | 사용 난이도   | 쉬움 (Jetpack 통합)   | 복잡              | 단순           |
    | 대표 사용 사례 | 데이터 동기화, 업로드      | 시스템 작업, 배치      | 알람, 푸시 시뮬레이션 |
  + 👉 즉, WorkManager는 JobScheduler + AlarmManager의 통합 고수준 API
 

- 면접 관련 질문
  + WorkManager를 언제 사용하나요?
    * 앱이 종료/재부팅돼도 반드시 실행되어야 하는 작업
    * 주기적 동기화, 파일 업로드/백업
    * 제약 조건 기반 작업
  + AlarmManager 대신 WorkManager를 써야 하는 이유는?
    * WorkManager는 작업 실행을 보장하지만 AlarmManager는 기기에 따라 누락될 수 있음
    * 다만 정확한 시간 알림은 AlarmManager가 필요
  + 주기 작업 최소 주기가 왜 15분인가요?
    * 배터리 절약을 위해 안드로이드 시스템 정책상 JobScheduler 기반 최소 간격이 15분
  + Worker / CoroutineWorker 차이는 무엇인가요?
    * Worker는 동기, CoroutineWorker는 suspend 기반 비동기 → 안드로이드에서는 CoroutineWorker 권장


---
