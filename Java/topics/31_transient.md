### transient
- 개념/정의
  + 직렬화(Serialization) 시 해당 필드를 직렬화 대상에서 제외시키기 위함
  + 해당 키워드로 선언된 필드는 ObjectOutputStream 으로 저장되지 않음
  + 직렬화(복습)
    * 객체를 바이트 스트림으로 변환해 파일 저장 및 네트워크 전송 가능하게 만드는 과정
    * Serializable 인터페이스 구현해야 사용 가능 (코틀린에서는 Parcelize)

- 왜 필요한가?
  + | 필요 이유       | 설명                      |
    | ----------- | ----------------------- |
    | 민감 정보 보호    | 비밀번호, 토큰 등 직렬화에서 제외     |
    | 불필요한 데이터 제외 | DB 커넥션, 스레드 등 직렬화 의미 없음 |
    | 캐시 / 임시 변수  | 실행 중 사용되지만 복원 불필요       |

- 코드
  + ```java
    import java.io.Serializable;

    public class User implements Serializable {
        private String username;
        private transient String password; // 저장 X

        public User(String username, String password) {
            this.username = username;
            this.password = password;
        } 
    }
    
    User user = new User("admin", "1234");
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.ser"));
    oos.writeObject(user);  // password는 저장되지 않음
    ```
    
- 면접 관련 질문
  + `transient` 키워드 언제 사용?
    * 민감 정보(비밀번호 등) or 직렬화가 의미 없는 필드(DB 커넥션, 캐시)등을 직렬화 대상에서 제외하기 위해 사용
  + `transient` & `static` 직렬화 차이?
    * `static` 클래스 단위 변수라 직렬화 대상이 아님
    * `transient` 일판 필드 중에서 직렬화를 제외하고 싶을 때 명시적으로 지정
  + `transient` 필드 복원 시 어떤 값?
    * `null`, `0`, `false` 등 해당 타입의 초기값으로 복권


---
