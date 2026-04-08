### Handler, Looper, MessageQueue
- 개념 및 정ㅢ
  + Handler: 메시지 전송자 → Runnable/Message를 Queue에 넣어 Looper에게 전달
  + MessageQueue: 메시지 저장소 (FIFO) → `enqueueMessage()`, `next()`로 꺼냄
  + Looper: 메시지 처리기 → Queue에서 메시지를 꺼내 `Handler.handleMessage()` 실행
  + 즉, Handler.post() → MessageQueue 저장 → Looper.loop() → Handler 실행 

- 비교표
  + | 구성요소         | 역할      | 설명                                   |
    |--------------|---------|--------------------------------------|
    | Handler      | 메시지 전송자 | 작업에서 메시지를 생성에서 Looper에게 전달           |
    | MessageQueue | 메시지 저장소 | 전송된 메시지를 시간순으로 저장 (FIFO 큐)           |
    | Looper       | 메시지 처리기 | MessageQueue를 무한히 돌면서 메시지를 하나씩 꺼내 실행 |

- 전체 구조 흐름
  + ```markdown
    - Thread
        └── Looper
              └── MessageQueue
                       ↑
                   Handler.post()
    ```
  + `Handler`가 `Message` 또는 `Runnable`을 생성하여 `MessageQueue`에 넣음
  + `Looper`는 `MessageQueue`를 계속 돌면서 메시지를 꺼냄
  + 해당 메시지를 처리 (`Handler.handleMessage()` 또는 `Runnable.run()` 호출)

- Handler
  + **Runnable** 또는 **Message**를 전달할 수 있는 인터페이스
  + `post(Runnable)` 또는 `sendMessage(Messgae)` 사용
  + `handlerMessage(Message msg)`를 override 하여 메시지 처리 가능

- Looper
  + `Thread`에 붙어서 해당 쓰레드가 이벤트 루프를 돌도록 만듬
  + 기본적으로 UI 메인 쓰레드에는 `Looper.getMainLooper()`가 존재
  + 새 쓰레드에서 쓰고 싶다면 `Looper.prepare()` -> `Looper.loop()` 호출 필요

- MessageQueue
  + 메시지들이 `enqueueMessage()`로 들어와 저장됨
  + `next()` 메서드로 메시지를 하나씩 꺼냄
  + `Looper.loop()`이 이 `next()`를 무한 반복

- 코드 예시
  + 메인 쓰레드에서 Handler 사용
    ```kotlin
        val handler = Handler(Looper.getMainLooper())
    
        handler.post {
            // UI 쓰레드에서 실행
            textView.text = "작업 완료"
        }
    ```
    * `post()`는 내부적으로 Message를 만들어 MessageQueue에 등록
    * Main Looper는 이걸 꺼내서 실행
  + 새 Thread에서 Handler 만들기
    ```kotlin
        class MyThread : Thread() {
            lateinit var handler : Handler
    
            override fun run() {
                Looper.prepare()    // Looper를 이 Thread에 바인딩
                handler = Handler(Looper.myLooper()!!) {
                    println("새 쓰레드에서 메시지 처리 됨")
                    true
                }
                Looper.loop()   // 메시지 루프 시작 
            }
        }
    
        val t = MyThread()
        t.start()
    
        // 나중에 다른 쓰레드에서 호출
        t.handler.post {
            println(" 이 코드는 MyThread 안에서 실행됨")
        }
    ```
  + 딜레이, 반복 작업 가능
    ```kotlin
        val handler = Handler(Looper.getMainLooper())
        val runnable = object : Runnable {
            override fun run() {
                println("1초마다 반복")
                handler.postDelayed(this, 1000)
            }
        }
        handler.post(runnable)
    ```
    - `postDelayed()`로 딜레이 실행 가능
    - `removeCallbacks(runnable)`로 중단 가능

- Handler 직접 써야하는 이유
  + | 상황                   | 이유                                    |
    |----------------------|---------------------------------------|
    | 다른 쓰레드에서 UI를 바꿔야 할 떄 | UI는 오직 메인 쓰레드에서만 수정 가능 -> Handler로 전달 |
    | 쓰레드 간 작업 분리          | 백그라운드 쓰레드에서 계산하고 -> 메인 쓰레드에서 결과 반영    |
    | 일정 시간 후 작업 실행        | `postDelayed`로 지연 실행 가능               |
    | 주기적 작업 처리            | `Runnable` 재전송 방식으로 반복 구현 가능          |

- 면접 관련 질문
  + Handler란 무엇인가요?
    * `Message` 또는 `Runnable`을 다른 스레드로 전달하고, Looper를 통해 큐에 등록된 작업을 실행하게 해주는 클래스.
    * UI 쓰레드로 작업을 전달하거나 딜레이/반복 작업에도 사용
  + Looper란 무엇인가요? 왜 필요한가요?
    * 해당 쓰레드에 연결되어 있는 이벤트 루프 처리기
    * `MessageQueue`를 반복해서 확인하며 메시지를 하나씩 꺼내서 실행
    * `UI 쓰레드`에는 기본적으로 `Looper`가 존재
  + Handler는 어떤 쓰레드에서 동작하나요?
    * 생성 시 지정한 Looper에 따라 동작하므로 메인 쓰레드에서 실행
    * 직접 만든 쓰레드에 사용하려면 Looper.prepare()로 초기화 해야 함
  + Handler 대신 Coroutines를 써도 되지 않나요?
    * Coroutine은 Handler보다 추상화된 방식으로 스레드 전환을 처리 가능
    * `withContext(Dispatchers.Main)`으로 메인쓰레드 전환이 가능하며, **코드 가독성과 에러 처리 측면에서 유리**
    * 하지만 내부적으로 Dispatchers는 `Handler`를 기반으로 동작하는 경우가 많다


---
