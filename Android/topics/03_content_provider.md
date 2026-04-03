### Content Provider
- 정의
  + 안드로이드 4대 컴포넌트 중 하나
  + 앱 간 데이터 공유 표준 인터페이스
  + SQLite, 파일, 네트워크 등 내부 저장소 구조를 숨기고 통일ㅚㄴ CRUD API로 제공

- 주요 역할
  + 데이터 공유: 다른 앱이 표준 인터페이스로 데이터 접근
  + 데이터 캡슐화: 내부 저장 방식(파일/DB 등) 노출 X
  + 접근 제어: 퍼미션 기반 권한 관리 (READ_CONTACTS, WRITE_CONTACTS 등)
  + CRUD 제공: query, insert, update, delete, getType 
  
- 핵심 구성 요소
  + Content URI
    * content://<authority>/<path>/<id>
      1. authority: ContentProvider의 고유 이름 (AndroidManifest에 등록)
      2. path: 테이블/데이터 종류
      3. id(optional): 특정 row 식별
    * content://contacts/people/3
      1. contacts authority의 people 테이블에서 ID=3 레코드 
  + ContentResolver
    * 다른 앱에서 Content Provider 접근 시 사용하는 클라이언트 API 
    * ```kotlin
      val cursor = contentResolver.query(
        ContactsContract.Contacts.CONTENT_URI,
        null, null, null, null
      )
      ```
    * 앱은 직접 Provider에 접근하지 않고 ContentResolver → ContentProvider 경유
 
- 표준 CRUD 메서드
  + | 메서드         | 역할                   |
    | ----------- | -------------------- |
    | `query()`   | 데이터 조회 (`Cursor` 반환) |
    | `insert()`  | 새로운 데이터 추가           |
    | `update()`  | 기존 데이터 수정            |
    | `delete()`  | 데이터 삭제               |
    | `getType()` | 특정 URI의 MIME 타입 반환   |
  
- 실무 예시
  + 안드로이드 기본 앱
    * 연락처 (ContactsContract)
    * 캘린더 (CalendarContract)
    * 미디어 파일 (MediaStore)
  + 우리 앱에서 직접 구현
    * DB 데이터를 외부 앱과 공유해야 할 때
    * 예: 메신저 앱이 content://messenger/chat/1 URI로 채팅 데이터 노출

- 비교
  + | 항목 | Activity/Fragment | ContentProvider |
    | -- | ----------------- | --------------- |
    | 목적 | UI 제공             | 데이터 공유          |
    | 실행 | 명시적/암시적 Intent    | Content URI     |
    | 권한 | Intent-filter 기반  | 퍼미션 기반 접근 제어    |
 

- 표준 CRUD 인터페이스
  + 데이터를 조작하기 위한 표준 메서드 집합을 제공합니다.
    * query() : 데이터 검색
    * insert() : 새 데이터 삽입
    * update() : 기존 데이터 수정
    * delete() : 데이터 삭제
    * getType() : 특정 URI에 해당하는 데이터의 MIME 타입을 반환

- 콘텐트 URI
  + Content Provider 내의 특정 데이터를 식별하기 위한 고유한 URI 체계를 사용합니다. 
  + content:// : 스킴, 콘텐츠 프로바이더임을 나타냅니다.
  + <authority>: 콘텐츠 프로바이더를 식별하는 고유한 이름
  + <path>: 프로아비더가 제공하는 데이터의 종류를 나타내는 경로
  + <optional_id>: 특정 레코드를 식별하기 위한 ID

- 면접 관련 질문
  + Content Provider는 언제 직접 구현하나요?
    * 내 앱 데이터를 다른 앱과 공유할 필요가 있을 때만
    * 자기 앱 내부에서만 쓰면 Room/DB/Repository로 충분
  + ContentResolver와 직접 DB 접근 차이는?
    * ContentResolver는 앱 간 표준 인터페이스
    * DB 접근은 내부 전용
    * 즉, ContentProvider는 앱 간 데이터 추상화 계층
  + Content Provider 성능 이슈 방지 방법은?
    * query는 반드시 백그라운드 스레드에서 처리
    * Cursor는 다 쓴 후 close()
    * 필요한 최소한의 데이터만 projection


---
