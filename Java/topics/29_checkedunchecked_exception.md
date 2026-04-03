### Checked/Unchecked Exception
- 개념
  + 자바에서 Exception은 크게 Checked Exception과 Unchecked Exception 두 가지로 나눕니다.
  + 이 둘의 가장큰 차이점은 컴파일 시점에 예외 처리 강제 여부 입니다.

- Checked Exception
  + 정의
    * java.lang.Exception 클래스를 상속받으면서, java.lang.Exception 클래스와 java.lang.Error 클래스를 상속받지 않는 모든 예외를 말합니다.
  + 특징
    * 컴파일 시점에 예외 처리를 강제합니다.
    * try-catch블록으로 감싸서 예외를 잡고 처리하는 코드를 작성합니다.
    * 메서드 선언부에 throws 키워드를 사용하여 해당 예외를 호출한쪽으로 다시 던져서 처리를 위임합니다.
    * 주된 목적 : 프로그래머가 인지하고 복구할 수 있는 예외적인 상황을 나타냅니다.
    * 개발자가 예외 상황을 예측하고 적절히 대응하도록 유도하여 프로그램의 안정성을 높이는데 기여합니다.

- Unchecked Exception
  + 정의
    * java.lang.RuntimeException 클래스와 그 하위 클래스들 그리고 java.lang.Error 클래스와 그 하위 클래스들을 말합니다.
  + 특징
    * 컴파일 시점에 예외 처리를 강제하지 않습니다. try-catch로 감싸거나 throws로 선언하지 않아도 컴파일 오류가 발생하지 않습니다.
    * 개발자가 명시적으로 처리할 수도 있지만, 필수는 아닙니다.
    * 주된 목적 (RuntimeException의 경우): 주로 프로그래밍 오류로 인해 발생하며, 예방 가능한 경우가 많습니다. 
      예를 들어, null인 객체의 멤버에 접근하려 할 때(NullPointerException), 배열의 범위를 벗어난 인덱스에 접근할 때(ArrayIndexOutOfBoundsException) 등이 있습니다.
    * 이러한 예외들은 코드 수정(버그 수정)을 통해 예방하는 것이 더 바람직하다고 간주됩니다.

- Kotlin에서의 예외 처리
  + Kotlin은 Java와 달리 모든 예외를 Unchecked Exception처럼 취급합니다. 
  + Kotlin에서는 throws 키워드로 예외를 선언할 필요가 없으며, try-catch로 예외를 처리하는 것도 필수가 아닙니다. 
    이는 Kotlin 설계자들이 Checked Exception이 때로는 과도한 보일러플레이트 코드를 유발하고, 
    실제로는 예외를 제대로 처리하지 않고 무시하는 경우가 많다고 판단했기 때문입니다. 
    하지만 Kotlin에서도 여전히 try-catch를 사용하여 예외를 처리할 수 있으며, Java 라이브러리와 상호 운용할 때는 Java의 Checked Exception을 인지하고 적절히 대응하는 것이 좋습니다.

- 요약
  + | 특징          | Checked Exception                               | Unchecked Exception (RuntimeException) |
    |--------------|-------------------------------------------------|----------------------------------------|
    | 상속 계층      | Exception의 하위 클래스 (단, RuntimeException 제외)  | RuntimeException의 하위 클래스               | 
    | 컴파일 시점 처리 | 강제 (try-catch 또는 throws 필요)                  | 강제하지 않음                                | 
    | 주된 원인      | 예측 가능하고 복구 가능한 외부 요인 (파일 없음, 네트워크 오류 등)| 프로그래밍 오류, 논리적 실수 (null 참조, 배열 범위 초과 등) | 
    | 처리 방식      | 예외를 잡아서 복구하거나, 호출한 곳으로 전파하여 처리 위임     | 주로 코드 수정을 통해 예방, 필요시 try-catch 사용      |

- 면접 질문
  + Checked Exception과 Unchecked Exception의 가장 큰 차이점은 무엇인가요?
    * 컴파일 시점의 예외 처리 강제 여부입니다. Checked Exception은 반드시 예외 처리를 해야 컴파일이 되고, Unchecked Exception은 그렇지 않습니다.
  + Checked Exception을 처리하는 두 가지 방법은 무엇인가요?
    * try-catch 블록으로 직접 처리하거나, throws 키워드를 사용하여 메서드를 호출한 쪽으로 예외 처리를 위임하는 방법이 있습니다.
  + Kotlin에서는 Checked Exception을 어떻게 다루나요? Java와 어떤 차이가 있나요?
    * Kotlin은 모든 예외를 Unchecked Exception처럼 취급합니다. 
      throws 키워드로 예외를 선언할 필요가 없으며, try-catch로 예외를 처리하는 것도 필수가 아닙니다. 
      Checked Exception이 때로는 과도한 보일러플레이트 코드를 유발한다는 판단 때문입니다.

---
