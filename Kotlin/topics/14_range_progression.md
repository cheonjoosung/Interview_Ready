### Range & Progression
- 개요
  + Range : 정해진 시작갑소가 끝값 사이의 연속된 값 집합 (ex: 1,2,3,4)
  + Progression(수열) : 일정한 간격(step)을 가진 수열 표현. 증/감소 모두 가능
  + 반복문, 조건문, 데이터 필터링 등 코드 가독성과 표현력 향상시킴

- Range
  + 닫힌 범위
    * ```kotlin
      val range = 1..5         // 1, 2, 3, 4를 포함하는 IntRange
      val charRange = 'a'..'d' // 'a', 'b', 'c', 'd'를 포함하는 CharRange
      ```
  + 반 열림 범위
    * ```kotlin
      val range = 1..<5 // 1 to 4 (5제외)
      ```
  + 포함 여부 체크
    * ```kotlin
      val x = 3
      if (x in 1..5) println("x는 1~5안에 있음")
      ```   
- Progression
  + step 키워드
    * ```kotlin
      val even = 0..10 step 2
      println(even.toList())  // [0, 2, 4, 6, 8, 10]
      ``` 
  + downTo
    * ```kotlin
      val countdown = 10 downTo 0
      println(countdown.toList())  // [10, 9, ..., 0]

      val customStep = 10 downTo 0 step 2
      println(customStep.toList())  // [10, 8, 6, 4, 2, 0]\
      ``` 
  + 반목문 활용
    * ```kotlin
      for (i in 1..5) print("$i ")         // 1 2 3 4 5
      for (i in 5 downTo 1 step 2) print("$i ") // 5 3 1
      ``` 
  + reversed()
    * ```kotlin
      val range = 1..5
      println(range.reversed().toList()) // [5, 4, 3, 2, 1]
      ```

- 구성요소 비교
  + | 구분      | Range (`..`, `..<`) | Progression (`step`, `downTo`) |
    | ------- | ------------------- | ------------------------------ |
    | 포함 여부   | 끝값 포함 / 미포함 선택 가능   | 자동 포함 (값 생성 규칙 따름)             |
    | step 설정 | 기본값 1 (증가)          | 자유롭게 `step` 지정 가능 (증가/감소 모두)   |
    | 방향      | 증가만 가능              | 증가/감소 모두 지원 (`downTo`, `step`) |
    | 활용도     | 범위 검사, 조건문, 반복문     | 커스텀 반복 조건, 특정 간격 수열 생성 등       |

- 면접 관련 질문
  + `..`와 `..<`의 차이점은?
    * `..`는 끝 값을 포함하는 닫힌 범위이고, `..<`는 끝 값을 포함하지 않는 반 열린 범위입니다.
    * 반복 조건이나 인덱스 조작에서 실수를 줄이기 위해 ..<를 자주 사용합니다.
  + Progression과 Range의 가장 큰 차이는?
    * Range는 기본적으로 1씩 증가하는 값의 집합이며 방향과 간격이 고정되어 있습니다.
    * 반면 Progression은 증가/감소 방향과 step 조절이 자유롭습니다.
  + Range와 함께 step을 사용하는 이유는?
    * 일정 간격의 값만 반복하고 싶을 때 사용합니다.
    * 예: 짝수만 반복, 역순 반복, 3단계 건너뛰기 등.       


---
