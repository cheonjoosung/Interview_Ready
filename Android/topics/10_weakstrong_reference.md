### Weak/Strong Reference
- 개요
  + `WeakReference` 와 `StrongReference`는 GC(가비지 컬렉션) 과 객체 생명주기 관리에서 중요한 개념
  + Android에서는 메모리 누수 방지, 캐시 최적화, Context 참조 관리 등에서 많이 활용됨

- Strong Reference (강한 참조)
  + 일반적으로 사용하는 기본 참조 방식 default 
  + ```kotlin
    val activity = MainActivity()
    ```
  + 특징
    * 참조가 남아 있는 한, GC 대상이 아님
    * 명시적으로 null 처리하거나, 스코프에서 벗어나야 GC 가능
    * 객체 생명주기를 강하게 유지 → 일반적인 코드 대부분 Strong Reference 
  
  
- Weak Reference (약한 참조)
  + GC가 객체를 자유롭게 수거할 수 있는 참조
  + ```kotlin
    val activityRef = WeakReference(acitivty)
    ```
  + 특징
    * 참조는 유지하지만 GC가 필요시 수거 가능
    * 메모리 누수 방지, 캐시, Context 참조, Listener/Handler에서 활용 

- Java 4가지 참조 타입 비교
  + | 참조 타입       | GC 수거 여부                          | 특징/사용처                                 |
    | ----------- | --------------------------------- | -------------------------------------- |
    | **Strong**  | 참조 남아 있으면 절대 수거 안 함               | 일반적인 객체 참조 방식                          |
    | **Soft**    | 메모리 부족할 때만 GC가 수거                 | 캐시 구현 시 사용 (메모리가 충분하면 유지)              |
    | **Weak**    | 언제든 GC가 수거 가능                     | Handler → Activity 참조 방지, 메모리 민감 객체 참조 |
    | **Phantom** | 객체가 **finalize 후**, 수거 직전에만 참조 가능 | 리소스 해제 추적, 정리 작업 (거의 실무에서는 잘 안 씀)      |

- 비교
  + | 항목         | Strong Reference  | WeakReference                      |
    | ---------- | ----------------- | ---------------------------------- |
    | GC 대상 여부   | 참조가 존재하면 GC 대상 아님 | GC는 자유롭게 수거 가능                     |
    | 참조 유지      | 강하게 유지 (직접 소유)    | 약하게 유지 (GC 허용)                     |
    | 사용 예       | 대부분의 변수, 클래스 속성 등 | 캐시, Context 참조, 리스너, View 등        |
    | Android 활용 | 일반적인 코드           | Handler → Activity 참조 방지, LruCache |

- 실전 예제
  + 잘못된 사용 예제
    * ```kotlin
      class MyHandler(val activity: Activity) : Handler() {
          override fun handleMessage(msg: Message) {
              activity.doSomething()
          }
      }
      ```
    * Activity가 끝나도 Handler가 내부에서 강하게 참조 중이면 GC 불가 → 메모리 누수
    * ```kotlin
      class MyHandler(activity: Activity) : Handler() {
         private val activityRef = WeakReference(activity)

         override fun handleMessage(msg: Message) {
             val activity = activityRef.get()
             activity?.doSomething()
         }
      }
      ```
    * Activity가 끝나면 GC가 수거 가능하도록 WeakReference 로 해결

- 면접 관련 질문
  + WeakReference는 언제 사용하나요? 실무에서의 예시도 함께 설명해주세요.
    * WeakReference는 객체를 참조하면서도 GC가 해당 객체를 수거할 수 있도록 허용할 때 사용
    * Android에서는 Activity나 Context처럼 생명주기가 명확하고 무거운 객체를 오래 참조하면 메모리 누수 위험이 있음
    * Handler나 Listener 내부에서 Activity를 참조할 때 WeakReference를 사용
    * 오래 실행되는 타이머나 핸들러가 Activity를 직접 참조하면 화면이 닫힌 뒤에도 메모리가 회수되지 않기 때문에 WeakReference(activity)로 감싸서 GC가 수거할 수 있게 합니다."
  + WeakReference 객체에서 실제 객체를 사용할 때 주의해야 할 점은?
    * WeakReference.get()은 반환값이 nullable이기 때문에, 항상 null 체크 후 사용
    * 이 참조는 언제든 무효화될 수 있으므로, 장시간 신뢰 가능한 객체 상태가 필요하다면 별도의 강한 참조 유지가 필요할 수 있음
  + GC 캐시 정책 관련 참조 타입
    * Strong/Soft/Weak/Phantom 4가지의 참조타입이 존재
    * soft 는 캐시용으로 적합하며 메모리 부족할때만 수거 (LRU캐시처럼 동작)
    * phantom 객체가 finalize 된 이후 추적용으로 쓰임. 리소스 해제 후 처리 작업 추적 용도이나 복잡해서 거의 쓰이질 않음
  + Soft / Weak / Phantom 차이는?
    * Soft → 캐시용 (메모리 부족 시에만 정리)
    * Weak → 즉시 수거 가능 (안정적 누수 방지)
    * Phantom → 수거 이후 정리 추적 (거의 안 씀) 


---
