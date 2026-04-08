### 와일드카드
- 개념 및 정의
  + 와일드카드 `?`는 Java 제네릭에서 정확한 타입을 명시하지 않고 어떤 타입이든
  허용되지만 제약은 있다의 의미로 사용하는 타입 대체제
  
- 와일드카드 형태
  + ```java
    <?>           // 모든 타입 허용
    <? extends T> // T or T 하위 타입만 허용 (upper bound)
    <? super T>   // T or T 상위 타입만 허용 (lower bound)
    ```
  + | 형태              | 의미                     | 읽는 법           |
    | --------------- | ---------------------- | -------------- |
    | `<?>`           | 어떤 타입이든 허용 (비한정 와일드카드) | "unknown type" |
    | `<? extends T>` | T 및 T의 **하위 타입만** 허용   | "T의 자식만 OK"    |
    | `<? super T>`   | T 및 T의 **상위 타입만** 허용   | "T의 부모만 OK"    |

- 예시 코드
  + ```java
    class Animal {}
    class Dog extends Animal {}
    class Cat extends Animal {}
    
    public void printAnimals(List<? extends Animal> animals) {
        for (Animal a : animals) {
            System.out.println(a);
        }
    }
    
    public void addAnimals(List<? super Dog> list) {
        list.add(new Dog()); // OK
        // list.add(new Animal()) // error
    }
    ```
   
- 와일드카드가 필요한 이유
  + 제네릭은 타입 안정성을 주지만, 구체적인 타입 지정 시 유연성 떨어짐
  + 다형성을 제네릭에서도 활용가능 함
    * ```java
      public void process(List<Dog> dogs) {}  // Cat은 못 넣음
      public void process(List<? extends Animal> animals) {}  // Dog, Cat 모두 가능
      ```
      
- 면접 관련 질문
  + Java 제네릭에서 <? extends T> 와 <? super T> 차이는?
    * <? extends T>는 T의 하위 타입만 허용하고 값을 읽을 수 있지만, 값을 추가할 수 없습니다. 
    * <? super T>는 T의 상위 타입만 허용하며, 값을 추가할 수 있지만 읽을 때는 Object로 받아야 합니다. 
    * 이를 구분하는 기준은 "PECS – Producer Extends, Consumer Super" 입니다.
  + List<?>와 List<Object>는 같은가요?
    * List<?> 어떤 타입이든 상관없다는 것이고 List<Object<> Object 타입만 가능
  + 와일드카드 사용하지 않고 제네릭을 쓰면 어떤 문제가 발생하나요?
    * 타입 고정 시 다형성 제한, List<Dog> 는 List<Animal> 서브타입 아니기에 Dog를 받으려면 오버러드 필요하기에
    재사용성과 유연성이 떨어지는 문제가 발생


---
