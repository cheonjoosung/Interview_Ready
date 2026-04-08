### Iterable, Sequence
- Iterable
  + `List`, `Set`, `Map` 등 대부분의 컬렉션이 `Iterable<T>` 구현
  + 즉시 계산 방식(Eager evaluation)
  + 각 연산 시 중간 컬렉션 생성
  + 특징
    * `map`, `filter`, `forEach` 등 다양한 확장 함수 제공
    * 컬렉션 전체를 한 연산씩 처리 (ex: `map` 전체 -> `filter` 전체)
    * ```kotlin
      interface Iterable<out T> {
          operator fun iterator(): Iterator<T>
      }

      interface Iterator<out T> {
          operator fun next(): T
          operator fun hasNext(): Boolean
      }

      val numbersList: List<Int> = listOf(1, 2, 3, 4, 5)

      // Iterable의 확장 함수 사용
      val doubledAndEven = numbersList
        .map { it * 2 }       // [2, 4, 6, 8, 10] - 중간 리스트 생성
        .filter { it % 2 == 0 } // [2, 4, 6, 8, 10] - 또 다른 중간 리스트 생성 (이 경우엔 동일)

      // for 루프를 통한 반복 (내부적으로 iterator 사용)
      for (number in numbersList) {
          println(number)
      }  
      ```
  + 장단점
    * | 장점               | 단점                         |
      | ---------------- | -------------------------- |
      | 간단하고 직관적인 구조     | 큰 데이터셋에서 성능 저하 가능          |
      | 컬렉션 전체를 쉽게 반복 가능 | 연산마다 새로운 리스트(중간 컬렉션) 생성됨   |
      | 결과를 즉시 확인 가능     | 연산 체인이 길어질수록 메모리/CPU 부담 증가 |

- Sequence
  + `Sequence<T>`는 지연 계산 방식(Lazy evaluation)
  + `map`, `filter` 등 중간 연산은 실행하지 않고 계획만 수립
    * ```kotlin
      val sequence = sequenceOf(1, 2, 3, 4, 5)

      val result = sequence
          .map {
              println("map: $it")
              it * 2
          }
          .filter {
              println("filter: $it")
              it % 2 == 0
          }

      println("정의만 되었고 아직 실행 안됨")

      val first = result.first() // 연산 시작! 1개만 처리
      ``` 
  + 특징
    * 요소 단위로 전체 파이프라인을 순차 처리
    * 중간 컬렉션을 만들지 않음
    * 최종 연산 (first(), toList(), count() 등) 호출 시 연산이 시작됨
    ```kotlin
    val numbersSequence: Sequence<Int> = sequenceOf(1, 2, 3, 4, 5)
    // 또는 기존 컬렉션을 Sequence로 변환
    // val numbersSequence = listOf(1, 2, 3, 4, 5).asSequence()
    ```
  + 장단점
    * | 장점                 | 단점                                  |
      | ------------------ | ----------------------------------- |
      | 성능 효율적 (중간 컬렉션 없음) | 소규모 연산에서는 약간의 오버헤드 존재               |
      | 큰 데이터, 복잡 연산에서 유리  | 디버깅이 어려울 수 있음                       |
      | **무한 시퀀스 처리 가능**   | 모든 API에서 지원되지 않음 (`randomAccess` 등) |
\
- Iterable vs. Sequence 정리
  + | 항목        | `Iterable`                          | `Sequence`                     |
    | --------- | ----------------------------------- | ------------------------------ |
    | 계산 방식     | 즉시 계산 (Eager)                       | 지연 계산 (Lazy)                   |
    | 중간 컬렉션 생성 | 매 연산마다 생성됨                          | 최종 연산까지 생성되지 않음                |
    | 연산 처리 순서  | 전체 컬렉션 → 각 연산별 처리                   | 각 요소 → 전체 파이프라인 순차 처리          |
    | 예시 메서드    | `.map()`, `.filter()`, `.forEach()` | `.asSequence().map().filter()` |
    | 사용 추천 상황  | 적은 데이터, 결과 즉시 필요할 때                 | 대용량, 복잡한 체인, 무한 처리 필요할 때       |
    | 성능        | 상대적으로 낮음                            | 성능 최적화 가능                      |

- 언제 무엇을 써야 할까?
  + | 상황                             | 추천 타입        |
    | ------------------------------ | ------------ |
    | 단순 변환, 소규모 컬렉션                 | ✅ `Iterable` |
    | 수십만 건 이상, 다단계 연산               | ✅ `Sequence` |
    | 무한 시퀀스 (`generateSequence {}`) | ✅ `Sequence` |
    | 연산 결과를 바로 써야 함                 | ✅ `Iterable` |
    | `first()`, `find()`로 빠르게 추출    | ✅ `Sequence` |

- 면접 관련 질문
  + Iterable과 Sequence의 차이점은?
    * Iterable은 즉시 계산, Sequence는 지연 계산
    * Iterable은 중간 컬렉션 생성, Sequence는 요소 단위 처리로 메모리 효율적
  + Sequence는 언제 쓰는 게 좋은가요?
    * 큰 데이터셋이나 무한 스트림 처리, 또는 중간 리스트 생성을 피하고 싶을 때
    * 예: asSequence().map().filter().first() 


---
