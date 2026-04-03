### 고차 함수
- 개념 및 정의
  + 함수를 인자로 받거나 함수를 반환하는 함수
  + 함수는 일급 객체이므로 변수처럼 전달 가능 -> 함수도 데이터처럼 다룰 수 있음
  + ```kotlin
    fun orderFunction (action : () -> Unit) {
        action()    // 전달받은 함수를 실행 
    }
    ```

- 고차함수의 장점
  + | 항목                | 설명                                           |
    | ----------------- | -------------------------------------------- |
    | 🎯 코드 재사용         | 반복되는 로직을 추상화하고, 로직을 외부에서 주입 받음               |
    | 🎯 가독성 향상         | 람다와 함께 사용 시 코드가 간결하고 의도 파악 쉬움                |
    | 🎯 유연한 동작 주입      | 동작을 파라미터로 받아서 다르게 정의 가능 (e.g., 클릭 리스너, 콜백 등) |
    | 🎯 코루틴/Flow/Rx 기반 | Kotlin 비동기 API 대부분이 고차 함수로 구성됨               |

- 고차함수 주의사항
  + | 이슈      | 설명                                    | 해결법                            |
    | ------- | ------------------------------------- | ------------------------------ |
    | 성능      | 함수/람다는 객체 생성 발생 → 힙 메모리 사용            | `inline` 키워드 사용 (컴파일 시 코드로 치환) |
    | 메모리 누수  | 람다가 외부 컨텍스트(Activity 등)를 캡처할 경우 GC 불가 | Lifecycle-aware 또는 WeakRef 사용  |
    | 디버깅 어려움 | 람다 중첩 시 콜스택 추적 어려움                    | 람다를 이름 있는 함수로 분리               |

- 사용 예제
  + 콜백 패턴
    * ```kotlin
      fun requestPermission(onGranted: () -> Unit, onDenied: () -> Unit) {
        val granted = true // 예시
        if (granted) onGranted() else onDenied()
      }

      requestPermission(
          onGranted = { println("허용됨") },
          onDenied = { println("거부됨") }
      )
      ```
  + 리스트 처리
    * ```kotlin
      val names = listOf("Alice", "Bob", "Charlie")

      val upper = names.map { it.uppercase() }
      val filtered = names.filter { it.length <= 4 }

      println(upper)      // [ALICE, BOB, CHARLIE]
      println(filtered)   // [Bob]
      ``` 
  + inline 과 고차 함수
    * inline 키워드를 쓰면 람다 함수 객체가 생성되지 않고, 호출 위치에 코드가 직접 삽입됨 → 성능 최적화
    * ```kotlin
      inline fun log(action: () -> Unit) {
        println("시작")
        action()
        println("끝")
      }
      ```  

- 면접 질문
  + 고차 함수를 사용하는 이유는?
    * 콜백 처리나 UI 이벤트 처리에 유연하게 대응 가능
    * 로직 추상화를 통해 중복 제거
    * 컬렉션 연산 (map/filter 등)을 간결하게 표현 가능
  + 고차 함수에서 inline을 쓰는 이유는?
    * 함수 객체 생성을 피하고, 성능 향상을 위해서입니다. 
    * 람다가 인라인 되면 호출 오버헤드와 힙 할당을 줄일 수 있습니다.
  + 고차 함수 사용 시 주의할 점?
    * 람다에서 외부 컨텍스트를 참조하면 GC 대상에서 제외될 수 있어 메모리 누수 발생 가능
    * 람다 중첩이 심해지면 가독성 저하 → 별도 함수로 추출 권장


---
