### 객체 직렬화
- **Serializable**
  + 개념 및 정의
    * 자바 표준 직렬화 방식
    * JVM 레벨에서 사용 (파일 저장, 네트워크 전송)
    * 마커 인터페이스 (Serializable)만 구현하면 됨
    * 내부적으로 **리플렉션(Reflection)**을 사용 → 느림, 메모리 사용량 큼
  + 특징
    * serialVersionUID 필요 (클래스 변경 시 버전 불일치 방지)
    * transient 키워드 → 직렬화에서 제외할 필드
    * static 필드 → 직렬화 대상 아님
    * 직렬화 대상 객체의 모든 필드가 Serializable 가능해야 함  
  + 직렬화 사용 이유
    * 저장 : 객체 상태를 파일에 저장하거나 DB에 저장
    * 전송 : 네트워크를 통해 객체를 다른 JVM으로 전달
    * 캐싱 : 객체를 메모리/디스크에 저장해 재사용
    * RPC : 원격 호출에서 객체 데이터를 주고 받을때 사용
  + 사용방법
    * **Serializable** 인터페이스 구현
      ```java
          import java.io.Serializable;
       
          public class User implements Serializable {
              private static final long serialVersionUID = 1L;

              private String name;
              private int age;
           
              // 생성자, getter, setter 등
          }
      ```
    * Serializable은 마커 인터페이스로, 메서드가 하나도 없고 단지 직렬화 대상이라는 표시만 함
    * 직렬화 예제 (저장)
      ```java
          ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.dat"));
          User user = new User("철수", 25);
          out.writeObject(user);
          out.close();
      ```
    * 역질렬화 예제 (복원)
      ```java
          ObjectInputStream in = new ObjectInputStream(new FileInputStream("user.dat"));
          User user = (User) in.readObject();
          in.close();
      ```
    + 주의할 점
      * serialVersionUID : 클래스 버전 관리를 위한 고유 ID (버전이 불일치할 경우 오류 발생 가능)
      * transient 키워드 : 직렬화에서 제외할 필드에 사용 (transient String password;)
      * static 필드 : 클래스에 속한 값이므로 직렬화 되지 않음
      * 직렬화 대상 객체의 모든 필드 : 직렬화가 가능해야 함 (안될경우 NotSerializableException 발생)

- **Parcelable**
  + 개념 및 정의
    * 안드로이드 전용 직렬화 방식
    * Intent, Bundle 등에서 객체 전달에 최적화
    * 객체를 바이트 배열로 직접 변환 → 성능 우수
    * Android Studio + Kotlin → @Parcelize 덕분에 구현 간단해짐
  + 특징
    * 수동 구현 시 writeToParcel() & CREATOR 필요
    * Kotlin에서는 @Parcelize로 자동 처리
    * 성능 최적화 : GC 적게 발생, 메모리 사용 효율적
  + 예제
    * 데이터 클래스 만들기
    ```kotlin
        import android.os.Parcelable
        import kotlinx.parcelize.Parcelize
        
        @Parcelize
        data class User (val name: String, val age: Int) : Parcelable
    ```
    * Intent에 담아서 전달
    ```kotlin
        val user = User("철수", 25)
        val intent = Intent(this, DetailActivity::class.java)
        
        intent.putExtra("user", user)
        startActivity(intent)
    ```
    * 받은 쪽에서 꺼내기
    ```kotlin
        val user = intent.getParcelableExtra<User>("user")
    ```

- Parcelable vs Serializable
  + | 항목         | Parcelable 🚀      | Serializable 🐢 |
    | ---------- | ------------------ | --------------- |
    | **속도**     | 빠름 (메모리 직접 처리)     | 느림 (리플렉션 기반)    |
    | **용도**     | Android IPC 전용     | JVM 전반 (범용)     |
    | **구현 난이도** | 다소 복잡 (직접 구현 필요)   | 쉬움 (인터페이스만 추가)  |
    | **성능**     | 고성능, 모바일 최적화       | 저성능, 메모리 낭비     |
    | **권장 상황**  | 안드로이드 컴포넌트 간 객체 전달 | 파일 저장, 네트워크 전송  |
 
- 실무 팁
  + Intent/Bundle → Parcelable
    * 성능 차이가 크므로 Android에서는 Parcelable 권장
  + 네트워크 전송, 파일 저장 → Serializable/Gson/Protobuf
    * Parcelable은 안드로이드 전용이므로 범용 저장/전송은 다른 직렬화 방식 사용사용
  + 대용량 객체 전달 X
    * Bundle/Intent 크기 제한 (보통 1MB 내외) → TransactionTooLargeException 발생 가능
    * DB, SharedPreferences, File 저장 후 경로만 전달하는 방식이 안정적 

- 면접 관련 질문
  + Parcelable이 Serializable보다 빠른 이유는?
    * Serializable은 리플렉션 기반 (많은 오버헤드)
    * Parcelable은 직접 메모리에 write/read (GC 발생 적음)
  + Parcelable과 Serializable 언제 각각 쓰나요?
    * 안드로이드 내부 데이터 전달 (Intent, Bundle) → Parcelable
    * 범용 저장/전송 (파일, 네트워크, 캐싱) → Serializable 또는 JSON, ProtoBuf
  + Bundle에 큰 데이터 넣으면 어떻게 되나요?
    * TransactionTooLargeException 발생 가능
    * 해결책: DB/파일에 저장 후 key만 전달


---
