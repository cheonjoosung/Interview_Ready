### inline 키워드
- 정의
  + `inline` 함수란 **호출되는 대신 그 본문이 호출 위치에 '직접 복사'되어 삽입**되는 함수
  + 주로 **고차 함수**에서 사용되며, **람다 객체 생성을 피하고 성능을 최적화**하는 목적

- 사용하는 이유?
  + | 목적           | 설명                                      |
    | ------------ | --------------------------------------- |
    | ✅ 성능 최적화     | 함수 호출 오버헤드 제거, 특히 람다 객체 생성을 생략          |
    | ✅ return 제어  | 람다 내부에서 return 사용 가능 (non-local return) |
    | ✅ reified 지원 | 제네릭 타입을 런타임에도 유지할 수 있음                  |
    | ❌ 코드 팽창 주의   | 반복 사용 시 바이너리 크기 증가 가능성 있음               |

- 기본 동작
  + ```kotlin
    inline fun log(block: () -> Unit) {
      println("== Start ==")
      block()
      println("== End ==")
    }

    fun main() {
      log {
         println("Hello!")
      }
    }
    
    // 컴파일 시 (개념상)
    fun main() {
       println("== Start ==")
       println("Hello!")
       println("== End ==")
    }  
    ```
  + log() 함수는 호출이 아니라 코드로 펼쳐져 들어감 (인라인화)

- inline 효과
  + | 항목             | 설명                              |
    | -------------- | ------------------------------- |
    | ✅ 성능           | 함수 호출 비용 제거 (특히 람다 객체 생성 안 함)   |
    | ✅ return 흐름 제어 | 람다 내부에서 `return` 가능             |
    | ❌ 코드 크기        | 너무 많이 쓰면 바이너리 커짐 (주의)           |
    | ✅ 제네릭 람다       | `inline` 덕분에 람다에서 reified 사용 가능 |

- 보조 키워드 : noinline, crossinline
  + `noinline` : 인라인 안하고 객체로 넘기고 싶을 때
    * ```kotlin
      inline fun test(a: () -> Unit, noinline b: () -> Unit) {
        a() // 인라인됨
        b() // 인라인되지 않음
      }
      ```
    * 람다 여러 개 중 일부만 인라인하고 싶을 때 유용
  + `crossinline` : 람다 안에서 non-local return 못하게 막기
    * ```kotlin
      inline fun doSomething(crossinline action: () -> Unit) {
         Thread {
            action() // 여기서 return 하면 컴파일 에러
         }.start()
      }
      ```
    * 다른 스레드/콜백에서 실행될 람다에는 return 쓰면 위험하므로 금지

- 실무 사용 예시
  + 성능 측정
    * ```kotlin
      inline fun measure(block: () -> Unit) {
        val start = System.currentTimeMillis()
        block()
        println("Elapsed: ${System.currentTimeMillis() - start}ms")
      }
      ```
  + 제네릭 타입 유지(`reified`)
    * ```kotlin
      inline fun <reified T> Gson.fromJson(json: String): T { 
         return this.fromJson(json, T::class.java)
      }
      ```
    * 일반 함수에서는 제네릭 타입 T는 런타임에 소멸되지만, inline + reified 덕분에 런타임에도 타입 정보를 유지 가능

- 면접 관련 질문
  + `inline` 함수 장점
    * 고차 함수 호출 시 발생하는 람다 객체 생성을 피해서 성능 향상
    * return 문법 유연화, reified 타입 활용 가능
  + `noinline` 이 필요한가?
    * 인라인 함수 내 람다 중 일부를 객체로 유지하고 싶을 때 사용
    * 예: 람다를 다른 함수에 전달하거나 저장하려는 경우 인라인하면 안 되기 때문
  + `reified` 는 왜 `inline` 함수에서만 사용할 수 있는가?
    * Kotlin의 일반 제네릭은 타입 소거되지만, inline 함수는 컴파일 시점에 T의 실제 타입 정보를 알 수 있으므로 이를 런타임에도 유지하게 하기 위해 reified와 함께 사용함


---
