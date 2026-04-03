### StringBuilder VS StringBuffer
- 개념 및 정의
  + 가변(Mutable) 문자열을 다루기 위해 설계된 클래스
  + `AbstractStringBuilder`를 상속받아 구현
  + `String`과 달리 문자열 변경 시 새 객체를 생성하지 않음
  + 문자열 수정 작업이 빈번할 때 성능 우위
  * ```java
    public final class StringBuffer extends AbstractStringBuilder implements Serializable, Comparable<StringBuffer>, CharSequence {}

    public final class StringBuilder extends AbstractStringBuilder implements Serializable, Comparable<StringBuilder>, CharSequence {}
    ```

- 공통점
  + | 항목     | 설명                                                                         |
    | ------ | -------------------------------------------------------------------------- |
    | 패키지    | `java.lang` 포함                                                             |
    | 상속 구조  | `AbstractStringBuilder` 상속                                                 |
    | 가변성    | 내부 문자열 변경 가능                                                               |
    | 내부 구조  | `char[]` 배열 기반                                                             |
    | 주요 메서드 | `append()`, `insert()`, `delete()`, `replace()`, `reverse()`, `toString()` |
    | 자동 확장  | 용량 초과 시 버퍼 자동 확장                                                           |
    | 성능     | `String`보다 문자열 수정 속도 훨씬 빠름                                                 |
    | 빌더 패턴  | 자기 자신을 반환 → `.append().append()` 체이닝 가능                                    |
  + ```java
    StringBuilder sb = new StringBuilder();
    sb.append("Hello").append(" World").append("!");
    System.out.println(sb); // Hello World!
    ```

- 차이점
  + | 비교 항목     | **StringBuilder**    | **StringBuffer**          |
    | --------- | -------------------- | ------------------------- |
    | 도입 시기     | JDK 1.5              | JDK 1.0                   |
    | 스레드 안전성   | ❌ 비동기(Thread-unsafe) | ✅ 동기화(Thread-safe)        |
    | 동기화 방식    | 없음                   | 모든 메서드에 `synchronized` 적용 |
    | 단일 스레드 성능 | 빠름                   | 느림(불필요한 동기화 오버헤드)         |
    | 멀티 스레드 성능 | 데이터 충돌 위험 있음         | 안전하게 공유 가능                |
    | 권장 환경     | 단일 스레드 문자열 처리        | 멀티 스레드 공유 문자열 처리          |
  + ```java
    // StringBuffer 내부 예시
    public synchronized String toString() {
      if (toStringCache == null) {
        return toStringCache = isLatin1()
          ? StringLatin1.newString(value, 0, count)
          : StringUTF16.newString(value, 0, count);
      }
    
      return new String(toStringCache);
    }
    ```

- 성능 비교 (String vs StringBuilder vs StringBuffer)
  + | 테스트 코드                     | 실행 시간(예시) |
    | -------------------------- | --------- |
    | `String` (`str += "a"`)    | 약 12초     |
    | `StringBuilder` (`append`) | 약 0.11초   |
    | `StringBuffer` (`append`)  | 약 0.12초   |
  + 단일 스레드 → StringBuilder
  + 멀티 스레드 → StringBuffer

- 주요 메서드
  + | 메서드                | 설명           |
    | ------------------ | ------------ |
    | `append()`         | 문자열 추가       |
    | `insert()`         | 특정 위치에 삽입    |
    | `replace()`        | 특정 구간 문자열 교체 |
    | `delete()`         | 특정 구간 삭제     |
    | `reverse()`        | 문자열 뒤집기      |
    | `capacity()`       | 현재 버퍼 용량 반환  |
    | `ensureCapacity()` | 최소 버퍼 용량 보장  |
    | `charAt()`         | 특정 인덱스 문자 반환 |
    | `substring()`      | 부분 문자열 추출    |

- 성능 최적화 팁
  + 초기 용량(capacity) 설정으로 불필요한 배열 복사 방지
  + 내부적으로 char[] 배열 사용 → 용량 초과 시 배열 복사 발생 → 예상 크기를 미리 설정하면 성능 향상
  + ```java
    StringBuilder sb = new StringBuilder(1000); // 초기 용량 지정
    ```

- 면접 관련 질문
  + StringBuilder와 StringBuffer 차이는?
    * 기능은 동일하지만, StringBuffer는 모든 메서드가 synchronized되어 있어 멀티 스레드 환경에서 안전합니다. 반면 StringBuilder는 동기화가 없어 단일 스레드에서 빠릅니다.
  + StringBuilder와 String의 차이는?
    * String은 불변(Immutable) → 변경 시 새 객체 생성,
    * StringBuilder는 가변(Mutable) → 동일 객체 내 수정.
  + StringBuffer를 꼭 써야 하는 경우는?
    * 멀티 스레드 환경에서 여러 스레드가 동시에 같은 문자열 객체를 수정하는 경우.
  + 성능 최적화를 위해 무엇을 할 수 있나?
    * 예상 문자열 길이만큼 초기 용량을 설정해 불필요한 배열 확장을 줄입니다.


---
