### JVM (Java Virtual Machine)
- 개념 및 정의
  + 자바 바이트코드(.class) 실행을 위한 가상 컴퓨터
  + 플랫폼 독립성 제공 → "Write Once, Run Anywhere"
  + 메모리 관리, 실행 최적화(JIT), 자동 GC 수행

- JVM 주요 역할
  + 바이트코드 → 기계어 변환 및 실행
  + 메모리 관리 (힙, 스택, 메타스페이스 등)
  + GC(가비지 컬렉션) → 사용하지 않는 객체 회수

- 구조
  + ```text
    JVM
    ├── Class Loader (클래스 로딩)
    ├── Execution Engine (실행 엔진)
    │     ├─ Interpreter (한 줄씩 해석)
    │     ├─ JIT Compiler (자주 쓰는 코드 → 네이티브 변환)
    │     └─ GC (Garbage Collector)
    └── Runtime Data Areas
          ├─ Method Area (MetaSpace) → 클래스 정보, static 변수
          ├─ Heap → new 객체 저장
          ├─ Stack → 메서드 호출, 지역 변수 (스레드별)
          ├─ PC Register → 현재 실행 주소
          └─ Native Method Stack → JNI, 네이티브 호출

    ```

- JVM 동작 흐름
  + 소스.java → javac → 바이트코드.class
  + Class Loader 가 JVM 메모리에 적재
  + Bytecode Verifier 안전성 검증
    * Execution Engine 실행
    * 인터프리터로 해석
  + JIT으로 자주 실행되는 코드 최적화 → 네이티브 코드 변환

- JVM 내부구조
  + | 영역                      | 설명                        |
    | ----------------------- | ------------------------- |
    | Method Area (MetaSpace) | 클래스, static, 상수 풀         |
    | Heap                    | 객체(new) 저장                |
    | Stack                   | 메서드 호출, 지역 변수 (스레드별)      |
    | PC Register             | 현재 실행 중인 바이트코드 주소         |
    | Native Method Stack     | JNI, C/C++ 같은 네이티브 메서드 호출 |

- JVM 장점
  + 플랫폼 독립성: OS와 무관
  + 메모리 자동 관리 (GC)
  + 안정성 (검증된 바이트코드만 실행)
  + 최적화 (JIT, GC 튜닝)

- 면접 관련 질문
  + JVM 메모리 구조 설명해보세요. (힙/스택/메타스페이스 차이 강조)
  + JIT 컴파일러는 무엇이고 왜 필요한가요?
    * 인터프리터 단점 보완, 자주 실행되는 코드 최적화
  + GC는 어떻게 동작하나요? (Stop-the-world, Young/Old Gen 개념)
  + JVM과 JRE, JDK 차이점은?
    * JDK = 개발 키트 (JRE + 컴파일러)
    * JRE = 실행 환경 (JVM + 라이브러리)
    * JVM = 실행 엔진


---
