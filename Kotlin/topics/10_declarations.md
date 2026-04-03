### 구조 분해 선언(Destructuring Declarations)
- 정의
  + 객체의 프로퍼티들을 하ㄴ 번에 여러 변수로 분해해서 할당할 수 있게 해주는 문법
  + `data class`, `Map`, `List`, `Pair`, `Triple` 등 사용

- 기본 개념
  + ```kotlin
    val (a, b) = someObject  // someObject.component1(), component2() 호출
    ```
  + 구조 분해가 가능하려면 해당 클래스가 `componentN()` 연산자 함수 제공
  + 코틀린 컴파일러는 구조 분해 선언을 만나면 내부적으로 `component1()`, `component2()` 형태로 메소드 호출

- 주요 사용 사례
  + Data Class
    * ```kotlin
      data class User(val name: String, val age: Int)

      val user = User("Alice", 30)
      val (name, age) = user
      println("Name: $name, Age: $age")  // 출력: Name: Alice, Age: 30
      ```

  + Map.Entry
    * ```kotlin
      val map = mapOf("A" to 1, "B" to 2)

      for ((key, value) in map) {
        println("Key: $key, Value: $value")
      }

      // 출력:
      // Key: A, Value: 1
      // Key: B, Value: 2
      ```

  + 리스트 or 배열
    * ```kotlin
      val list = listOf("Apple", "Banana", "Cherry")
      val (first, second) = list

      println("First: $first, Second: $second")  // 출력: First: Apple, Second: Banana
      ```

  + 여러 값 반환
    * ```kotlin
      data class Result(val value: Int, val status: String)

      fun process(): Result = Result(42, "Success")

      val (value, status) = process()
      println("Value: $value, Status: $status")
      ```

  + 일부 값 무시
    * ```kotlin
      data class Point(val x: Int, val y: Int, val z: Int)

      val point = Point(10, 20, 30)
      val (x, _, z) = point

      println("X: $x, Z: $z")
      ```

  + 람다 파라메터
    * ```kotlin
      val users = listOf(User("Bob", 25), User("Charlie", 35))

      users.forEach { (name, age) ->
          println("$name is $age years old")
      }

      val map = mapOf(1 to "one", 2 to "two")
      map.forEach { (key, value) ->
          println("$key -> $value")
      }
      ```

- 성능 및 주의사항
  + 구조 분해는 컴파일 시 componentN 호출로 변환되므로 성능 부담은 거의 없음
  + 다만, 지나치게 많은 변수 사용은 가독성 저하나 실수를 유발할 수 있음
  + 컴파일 시점에 componentN() 함수가 정의되어 있어야 컴파일 가능 

- 질문
  + 구조 분해 선언시 componentN() 함수는 어떤 역할을 하나요?
    * 구조 분해 선언시 내부적으로 호출되는 메서드 입니다. 
      예를 들어 val (x, y) = point는 point.component1()과 point.component2()를 호출합니다. 이는 컴파일러가 자동으로 변환해주는 형태입니다.
  + 구조 분해 선언이 성능에 영향을 줄 수 있나요?
    * 아니요. 구조 분해는 컴파일 타임에 componentN() 메서드 호출로 변환되므로 오버헤드는 거의 없습니다. 
    * 다만, 너무 많은 분해 변수는 코드 가독성을 해칠 수 있어 주의가 필요합니다.
  + 구조 분해 선언이 가능하려면 어떤 조건이 필요한가?
    * 해당 클래스가 componentN() 함수를 제공해야 합니다.
    * data class는 이 함수를 자동으로 생성하며, Map.Entry, Pair, Triple 등 일부 클래스는 표준 라이브러리에서 제공됩니다.  


---
