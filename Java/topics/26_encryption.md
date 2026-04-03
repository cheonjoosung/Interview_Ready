### 암복호화 방식 및 샘플
- 개념/정의
    + 암호화(Encryption)
        * 데이터를 알아볼 수 ㅇ벗는 형태로 변환하여 제3자가 내용을 확인하지 못하도록 보호하는 기술
    + 복호화(Decryption)
        * 암호화된 데이터를 원래 상태로 되돌리는 과정

- 왜 필요한가?
    + 기밀성 : 외부에 노출되어도 내용 보호
    + 무결성 : 데이터 변경 여부 탐지 가능
    + 인증 : 사용자/서버의 신원 확인
    + 법/정책 준수 : GDPR, 개인정보보호법 등 대응 필요
    + 로그인 비밀번호, 결제정보, API 통신(개인정보), DB저장 등

- 암복호화 종류 및 방식
    + 대칭키 암호화(Symmetric Encryption)
        * 하나의 키로 암호화/복호화 모두 수행
        * 빠르고 사용이 간단
        * 키 노출 시 보안 취약
        * 알고리즘 : AES, DES, 3DES, Blowfish
    + 비대칭키 암호화(Asymmetric Encryption)
        * 공개키(Public Key) 암호화
        * 개인키(Private Key) 복호화
        * 키가 다르기에 안전환 통신 가능함
        * 속도 느림, 주로 키 교환/인증/서명에 사용
        * 알고리즘 : RSA, DSA, ECC
    + 해시(Hashing) 단방향
        * 복호화 불가능, 무결성 확인 / 비밀번호 저장 등 사용
        * SHA-256, MD5(비추천)

- 샘플 코드(RSA)
    * ```java
      import javax.crypto.Cipher;
      import java.security.KeyPair;
      import java.security.KeyPairGenerator;
      import java.security.PrivateKey;
      import java.security.PublicKey;
      import java.util.Base64;

      public class RSASample {
      public static void main(String[] args) {
          String plainText = "Hello RSA";

          try {
              // 키쌍 생성
              KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
      
              // RSA 2048 사용
              keyPairGenerator.initialize(2048);
              KeyPair keyPair = keyPairGenerator.generateKeyPair();

              PublicKey publicKey = keyPair.getPublic();
              PrivateKey privateKey = keyPair.getPrivate();

              // 암호화
              Cipher encryptCipher = Cipher.getInstance("RSA");
              encryptCipher.init(Cipher.ENCRYPT_MODE, publicKey);

              byte[] encryptedBytes = encryptCipher.doFinal(plainText.getBytes());
              String encryptedBase64 = Base64.getEncoder().encodeToString(encryptedBytes);
              System.out.println("암호화 결과: " + encryptedBase64);

              // 복호화
              Cipher decryptCipher = Cipher.getInstance("RSA");    
              decryptCipher.init(Cipher.DECRYPT_MODE, privateKey);

              byte[] decryptedBytes = decryptCipher.doFinal(Base64.getDecoder().decode(encryptedBase64));
              String decryptedTest = new String(decryptedBytes);
              System.out.println("복호화 결과: " + decryptedTest);
          } catch (Exception e) {
            System.out.println(e);
          }
      }
      }
      ```

- 면접 관련 질문
    + 대칭키와 비대칭키의 차이
        * 대칭키는 하나의 키로 암호화와 복호화를 모두 수행하고 비대칭키는 공개키/개인키 사용
    + 해시와 암호화의 차이
        * 해시는 복호화가 불가능한 단방향 변환이고 무결성 확인 및 비빌먼호 저장에 사용
        * 암호화는 복호화를 전제로 하며 데이터 보호를 위해 사용
    + 실무에서 대칭/비대칭 어떻게 조합
        * 비대칭키로 대칭키를 안전하게 전달하고 이후 대칭키를 실제 데이터를 빠르게 암호화 복호화


---
