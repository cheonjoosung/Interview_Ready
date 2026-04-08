### String 문자열
- 개념 및 정의
  + 문자열을 다루는 가장 기본적이고 중요한 참조형 타입 클래스
  + `final` 클래스이므로 상속 불가
  + 내부 값은 `final byte[] value`로 저장 -> 불변(Immutable)
  문자열을 다루는 가장 기본적이고 중요한 참조형 타입 클래스
  + ```java
    public final class String
      implements java.io.Serializable, Comparable<String>,  CharSequence, Constable, ConstantDesc {
    
      @Stable
      private final byte[] value;
    }
    ```

- 특징
  + Immutable class (final 키워드 사용)
    * 문자열 값은 한 번 생성되면 변경 불가
    * 변경 연산(+, concat) 시 새로운 객체 생성 후 참조를 변경
    * ```java
      String a = "test";
      a += "test"; 
      // 기존 test 위치에 추가되는 것이 아니라 testtest 를 만들고 참조하는 값의 위치를 바꾼다
      ```
    * 불변성 장점
      1. 멀티스레드 환경에서 안전(Thread-safe)
      2. 해시값 캐싱 가능 (hashCode() 결과 재사용)  
  + Method Area 생성 (String Pool)
    * 문자열 리터럴은 String Constant Pool에서 관리
    * 동일한 문자열 리터럴은 재사용
    * new 키워드 사용 시 Heap 영역에 별도 객체 생성
    * intern() 메서드 → Heap 객체를 Pool 참조로 변경
    * ```java
      String a = "test";
      String b = "test";
      String c = new String("test");
      String d = new String("test");
      System.out.println(a == b); // true
      System.out.println(a == c); // false
      System.out.println(b == c); // false
      System.out.println(c == d); // false
      
      System.out.println(c.intern() == a); // true (intern() → Pool 참조)
      ```
    * | 생성 방식               | 저장 위치       | 동일 값 재사용 여부                  |
      | ------------------- | ----------- | ---------------------------- |
      | `"abc"` (리터럴)       | String Pool | O                            |
      | `new String("abc")` | Heap        | X (intern() 호출 시 Pool 참조 가능) |
  + 형변환
    * ```java
      // 기본 타입 -> String 
      String s1 = String.valueOf(123);
      String s2 = String.valueOf(true);

      // String -> 기본 타입
      String str = "4885";
      int num = Integer.parseInt(str);
      double d = Double.parseDouble(str);
      float f = Float.parseFloat(str);
      char c = str.charAt(1);
      ``` 

- 자주 쓰이는 메서드
  + | 기능      | 메서드 예시                                  |
    | ------- | --------------------------------------- |
    | 대소문자 변환 | `toLowerCase()`, `toUpperCase()`        |
    | 포함 여부   | `contains("abc")`                       |
    | 내용 비교   | `contentEquals("abc")`, `equals("abc")` |
    | 시작/끝 확인 | `startsWith("pre")`, `endsWith("txt")`  |
    | 공백 제거   | `trim()`                                |
    | 분리      | `split(",")`                            |

- 관련 심화 주제
  + StringBuilder / StringBuffer
    * StringBuilder: 가변 문자열, Thread-unsafe
    * StringBuffer: 가변 문자열, Thread-safe
    * 문자열 잦은 변경 시 String보다 효율적
  + JVM 구조
    * String Pool은 JVM의 Method Area(Java 8부터는 Metaspace) 관리 영역에 위치

- 면접 관련 질문
  + String이 불변(Immutable)하게 설계된 이유는 무엇인가요?
    * 보안성, 스레드 안전성, 해시 캐싱 최적화를 위해서입니다.
    * 예를 들어, URL, 파일 경로, 키 값 같은 문자열은 변경되면 보안 위험과 오류를 유발할 수 있습니다.
  + ==과 equals()의 차이는 무엇인가요?
    * ==: 참조 비교 (같은 객체 주소인지 확인)
    * equals(): 문자열 내용 비교 (값이 동일한지 확인)
  + String, StringBuilder, StringBuffer 차이는?
    * String: 불변 객체, 변경 시 새 객체 생성
    * StringBuilder: 가변 객체, Thread-unsafe, 빠름
    * StringBuffer: 가변 객체, Thread-safe, 상대적으로 느림
  + intern() 메서드는 언제 사용하나요?
    * Heap에 생성된 String 객체를 String Pool에 넣어 리터럴 참조와 동일하게 사용하고 싶을 때 사용합니다.


---
