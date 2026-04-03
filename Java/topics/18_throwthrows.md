### throw/throws
- throw vs throws 의 차이
  + | 구분     | **throw**             | **throws**           |
    | ------ | --------------------- | -------------------- |
    | **역할** | 예외 **객체를 직접 발생**      | 예외를 **메서드 선언부에서 위임** |
    | **위치** | 메서드 **본문 내부**         | 메서드 **선언부 (매개변수 뒤)** |
    | **개수** | 한 번에 **하나만**          | 여러 개 나열 가능 (`,`로 구분) |
    | **대상** | Exception **객체 인스턴스** | Exception **클래스 타입** |

- throw (예외 발생 시키기)
  + 예외 객체를 new 해서 직접 던짐
  + 반드시 Throwable 하위 클래스만 가능
  + ```java
    public void checkAge(int age) {
      if (age < 18) {
        throw new IllegalArgumentException("나이는 18세 이상이어야 합니다.");
      }
    }
    ``` 
  
- throws (예외 위임하기)
  + throws (예외 위임하기)
  + 해당 메서드에서 발생 가능한 예외를 호출자에게 책임 전가
  + Checked Exception의 경우 필수적으로 명시해야 함
  + ```java
    public void readFile(String path) throws IOException {
      FileReader fr = new FileReader(path);
    }
    ```

- 함께 사용하는 예
  + ```java
    public void read(String path) throws IOException {
      if (path == null) {
        throw new IllegalArgumentException("경로는 null일 수 없습니다."); // Unchecked
      }
      throw new IOException("파일을 읽을 수 없습니다."); // Checked
    }
    ``` 
  + throw -> 실제 예외 객체 던짐
  + throws -> 메서드가 해당 예외를 던질 수 있음을 명시
  
- checked vs Unchecked
  + | 구분                      | 설명                                | 예시                                                 |
    | ----------------------- | --------------------------------- | -------------------------------------------------- |
    | **Checked Exception**   | 반드시 `throws` 또는 `try-catch` 필요    | `IOException`, `SQLException`                      |
    | **Unchecked Exception** | `RuntimeException` 계열 → 명시 안 해도 됨 | `NullPointerException`, `IllegalArgumentException` |

- 면접 관련 질문
  + throw와 throws 차이는?
    * throw는 예외 객체를 던지는 것,
    * throws는 예외를 메서드 선언부에서 위임하는 것
  + Checked Exception과 Unchecked Exception 차이는?
    * Checked는 컴파일 시점에 강제 처리, throws 필요
    * Unchecked는 런타임 시 발생, 선택적 처리
  + 왜 Checked Exception이 존재할까?
    * 반드시 예외 처리를 강제해 안정성 확보
    * 단, 과도하면 코드 복잡성 증가 → 실무에서는 Unchecked(Exception wrapping) 방식 많이 씀 (Spring도 대부분 런타임 예외로 변환) 


---
