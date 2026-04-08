### 반공변성
- 개념 및 정의
  + 타입 계층 구조에서 하위 타입 대신 상위 타입을 받도록 허용하는 제네릭 타입 제약 방식
  + 제네릭 타입 매개변수 앞에 `in` 키워드를 붙여 선언
    * 해당 타입이 입력(소비) 전용 명시
    * T의 부모 타입까지 허용하게 만들 수 있음
  + Int 타입이 Number 하위 자료형일 때 Class Box<in T> 선언 시
    * Class Box<Number>는 Class Box<Int> 의 하위 자료형이 됨
  + ![out_in](../_assets/variance.png)

- 예시
  + ```kotlin
    open class Animal
    class Dog : Animal()
    
    class AnimalConsumer<in T> {
        fun consume(animal: T) {
            println("Consumed: $animal")
        }
    }
    
    val dogConsumer: AnimalConsumer<Dog> = AnimalConsumer<Animal>()  // OK (in 사용 시)
    ```
  + `AnimalConsumer<in T>` T의 부모 타입도 안전하게 대입 가능하도록 만듬
  + 'AnimalConsumer<Animal>' 은 `AnimalConsumer<Dog>`로 대입될 수 있음 -> 반공변성

- 필요한 이유?
  + 함수를 인자로 받는 경우, 그 함수가 더 일반적인(상위) 타입을 소비할 수 있어야 유연한 설계가 가능
  + 반공변성은 이런 상황에서 타입 안정성을 유지하면서도 다형성을 ㄷ하기도ㅇ시에 유지할 수 있게 도와줌
  + ```kotlin
    fun feedDogs(consumer:AnimalConsumer<Dog>) {
        consumer.consume(Dog())
    }

    // AnimalConsumer<Animal>을 넣을 수 있음 (Dog를 포함하므로)
    feedDogs(AnimalConsumer<Animal>())
    ```
    * 만약 `AnimalConsumer<Dog>` 대신 `AnimalConsumer<Animal>` 을 넘길수 없다면 유연한 구조 불가

- 공변성과의 비교
  * | 개념       | 키워드   | 의미                 | 예시 타입 방향                                   |
    | -------- | ----- | ------------------ | ------------------------------------------ |
    | 공변성      | `out` | 생산 전용 (출력만)        | `List<out Animal>` → `List<Dog>` 허용        |
    | **반공변성** | `in`  | 소비 전용 (입력만)        | `Consumer<in Dog>` → `Consumer<Animal>` 허용 |
    | 무공변      | 없음    | 기본값, in/out 둘 다 불가 | `MutableList<T>` 등                         |

- Java의 `? super T`와의 비교
  + | 언어     | 표현          | 의미                     |
    | ------ | ----------- | ---------------------- |
    | Kotlin | `in T`      | `T`의 상위 타입도 허용 (입력 전용) |
    | Java   | `? super T` | `T`의 상위 타입도 허용         |

- 면접 관련 질문
  + Kotlin에서 in 키워드는 어떤 의미인가요?
    * 제네릭 타입이 입력(소비) 전용으로 상위 타입으로 대체 가능
  + Kotlin의 in과 Java의 ? super T는 어떤 관계인가요?
    * 동일한 의미로 사용
  + 반공변성과 공변성은 서로 반대 방향이라고 보아도 되나요?
    * 공변성(out) 은 생산(출력) 하기에 하위 타입 허용
    * 반공변성(in) 은 소비(입력) 하기에 상위 타입 허용


---
