### reified & 제네릭
- 정의
  + 제네릭
    * 타입에 의존하지 않고 유연하고 재사용 가능한 코드를 작성할 수 있게 해주는 문법
    * ```kotlin
      fun <T> printItem(item: T) {
        println(item)
      }
      ```
    * ⚠️ 타입 소거(Type Erasure) 발생
      1. 컴파일 후 T의 실제 타입 정보가 런타임에 사라짐
      2. 예: T::class, T::class.java, is T, as T 사용 ❌불가
  + reified
    * reified: "구체화된"이라는 의미. 제네릭 타입 T를 런타임에도 타입 정보로 유지
    * 사용 조건:
      1. inline 함수의 타입 파라미터에만 사용 가능
      2. 컴파일 시 실제 타입으로 치환(inline + 타입 구체화)  
    * ```kotlin
      inline fun <reified T> getType(): Class<T> {
        return T::class.java // ✅ 가능!
      }
      ``` 

- 일반 제네릭과의 차이
  + | 기능                             | 일반 제네릭    | reified 제네릭      |
    | ------------------------------ | --------- | ---------------- |
    | `T::class`, `T::class.java` 사용 | ❌         | ✅ 가능             |
    | `is T`, `as T` 타입 검사           | ❌         | ✅ 가능             |
    | 런타임 타입 유지                      | ❌ (타입 소거) | ✅ (컴파일 시 타입 삽입)  |
    | 사용 가능 위치                       | 어디든       | `inline` 함수 안에서만 |
   
- 주요 활용 예시
  + Gson JSON 파싱
    * ```kotlin
      inline fun <reified T> parseJson(json: String): T {
        return Gson().fromJson(json, T::class.java)
      }

      val user: User = parseJson(jsonString)
      ``` 
    * 일반 제네릭이라면 TypeToken<T>().type을 써야 해서 복잡함
  + 타입 검사/필터링
    * ```kotlin
      inline fun <reified T> isOfType(value: Any): Boolean {
        return value is T
      }
      
      val result = isOfType<String>("Hello") // true
      ```
  + ViewModel Factory 
    * ```kotlin
      inline fun <reified VM : ViewModel> createViewModel(): VM {
        return ViewModelProvider(this).get(VM::class.java)
      }
      ```

- 상황별 사용 정리
  + | 상황                                   | `reified` 필요 여부 |
    | ------------------------------------ | --------------- |
    | 런타임 타입 검사 (`is T`, `as T`)           | ✅ 필요            |
    | `T::class`, `T::class.java` 사용       | ✅ 필요            |
    | 단순한 타입 재사용 / 전달                      | ❌ 불필요           |
    | Gson/Retrofit/뷰모델 등 런타임 타입 반영 필요한 경우 | ✅ 매우 유용         |
       
- 면접 관련 질문
  + `reified` 키워드 필요한 이유?
    * Kotlin의 제네릭은 타입 소거 때문에 T::class, is T 같은 연산이 불가
    * reified는 실제 타입 정보를 코드에 삽입해서 런타임에도 활용 가능하게 해줌
  + `inline` 함수에서만 `reified`를 쓸 수 있는가?
    * inline 함수는 컴파일 시 함수 본문이 호출 위치에 복사됨
    * 이때 T가 실제 타입으로 치환되므로 타입 정보 유지 가능
    * 일반 함수는 이 치환이 안 되기 때문에 타입 소거 발생 → reified 사용 불가


---
