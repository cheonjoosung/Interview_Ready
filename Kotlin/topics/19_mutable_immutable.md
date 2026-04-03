### mutable & immutable
- 변수 (var / val)
  + | 키워드 | 의미             | 값 변경 가능 여부 | 예시 코드                     |
    |--------|------------------|------------------|------------------------------|
    | `val` | 읽기 전용 (immutable) | ❌ 불가능           | `val name = "Jane"`         |
    | `var` | 읽기/쓰기 가능 (mutable) | ✅ 가능              | `var age = 30` → `age = 31` |
  + ```koltin
    val nickname = "Sky" // ❌ nickname = "Blue" → 컴파일 에러
    var score = 100      // ✅ score = 150 → 정상 작동
    ```

- 컬렉션 (List/Set/Map)
  + 불변 컬렉션 (Immutable) 
    * 수정 불가, 읽기 전용
    * ```kotlin
      val names = listOf("A", "B", "C")
      // names.add("D") ❌ 컴파일 에러
      ``` 
  + 가변 컬렉션 (Mutable) 
    * 요소 추가/삭제 가능
    * ```kotlin
      val names = mutableListOf("A", "B")
      names.add("C") // ✅ 가능
      ``` 
  + | 컬렉션 타입                                    | 수정 가능 여부 |
    | ----------------------------------------- | -------- |
    | `List`, `Set`, `Map`                      | ❌ 불가     |
    | `MutableList`, `MutableSet`, `MutableMap` | ✅ 가능     |
  
- 클래스의 불변성 (Immutable Object)
  + ```kotlin
    // 불변 객체
    data class User(val name: String, val age: Int)

    val user = User("Alex", 20)
    // user.name = "John" ❌ 컴파일 에러
    ``` 
  + ```kotlin
    // 가변 객체
    data class MutableUser(var name: String, var age: Int)

    val mUser = MutableUser("Alex", 20)
    mUser.name = "John" // ✅ 변경 가능
    ```

- 왜 Immutable 이 더 좋은가?
  + | 장점           | 설명                           |
    | ------------ | ---------------------------- |
    | ❄️ 예측 가능성    | 값이 바뀌지 않으므로 디버깅, 테스트가 쉬움     |
    | 🔐 스레드 안전성   | 멀티스레드 환경에서 동기화 없이 안전하게 사용 가능 |
    | ♻️ 참조 공유     | 여러 곳에서 안전하게 공유 가능            |
    | 🧠 함수형 프로그래밍 | 불변 상태는 함수형 패러다임과 자연스럽게 통합됨   |

- 면접 관련 질문
  + val과 var의 차이는?
    * val: 읽기 전용. 초기화 후 값 변경 불가
    * var: 읽기/쓰기 모두 가능
  + val로 선언한 MutableList의 요소는 변경 가능할까?
    * 가능합니다.
    * val은 참조를 바꿀 수 없는 것이지, 참조된 컬렉션의 내용까지 불변이라는 의미는 아님 
    * ```kotlin
      val list = mutableListOf(1, 2, 3)
      list.add(4) // ✅ 가능
      // list = mutableListOf(9, 9, 9) ❌ 불가능 (참조 변경 금지)
      ```
  + 코틀린 `List` 와 `MutableList` 차이점?
    * | 항목      | `List` | `MutableList` |
      | ------- | ------ | ------------- |
      | 읽기      | ✅ 가능   | ✅ 가능          |
      | 쓰기 (수정) | ❌ 불가   | ✅ 가능          |
  + 언제 immutable 을 쓰고 언제 mutable 을 써야 하나요?
    * 기본은 불변으로 설계하고, 정말 필요할 때만 mutable 사용
    * 상태 변화가 불가피한 경우만 var, MutableList 사용
    * 데이터 전달, API 응답, UI 상태 표현 등은 대부분 immutable로 구성하는 것이 안전


---
