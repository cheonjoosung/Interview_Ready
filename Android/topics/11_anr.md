### ANR(Application Not Responding)
- 정의
  + 안드로이드 앱에서 메인(UI) 스레드가 장시간(blocking) 멈춰 이벤트나 화면 갱신을 처리하지 못하면 발생
  + 시스템이 5초 이상 입력 이벤트 처리 못하면 → ANR 대화상자 표시

- ANR 발생 조건
  + | 유형                       | 제한 시간 | 설명                                                     |
    | ------------------------ | ----- | ------------------------------------------------------ |
    | 입력 이벤트 처리 지연             | 5초    | 터치, 키 입력 이벤트 처리 못할 때                                   |
    | Foreground Service 시작 지연 | 5초    | `startForegroundService()` 호출 후 `startForeground()` 지연 |
    | Service 실행 지연            | 수 초   | `onCreate()`, `onStartCommand()` 등 오래 걸릴 때             |
    | BroadcastReceiver 지연     | 10초 내 | `onReceive()`가 오래 걸릴 때                                 |
    | JobService 지연            | 수 초   | `onStartJob()`, `onStopJob()` 반환 지연                    |

- 주요 원인
  + 메인 스레드 블로킹
    * 네트워크 작업, 대용량 파일 I/O
    * 복잡한 연산 (긴 루프, 알고리즘)
    * 무거운 UI 갱신 (RecyclerView 과도한 bind 등)
  + 동기화 문제
    * 락 경쟁, 교착 상태(Deadlock)
  + 잘못된 컴포넌트 사용
    * BroadcastReceiver에서 긴 작업 수행
    * Service에서 무거운 로직 직접 처리
  + 시스템/기기 부하
    * 시스템 리소스 부족, 다른 프로세스 간섭  

- 예방 방법
  + 백그라운드 처리
    * 네트워크, 파일 I/O, DB → 별도 스레드 / Coroutine / WorkManager
  + StrictMode 활용
    * 개발 중 메인 스레드에서 잘못된 I/O 탐지
  + UI 최적화
    * RecyclerView, DiffUtil, 비동기 렌더링
  + 락/동기화 최소화
    * 락 범위 최소화, 순서 일관성 유지
  + 서비스/리시버 최적화
    * BroadcastReceiver에서는 빠른 처리만 → 나머지는 JobScheduler/WorkManager

- 면접 관련 질문
  + ANR이란 무엇인가요?
    * 메인 스레드가 입력 이벤트를 5초 이상 처리 못해 사용자에게 "응답 없음" 대화상자가 뜨는 현상
  + 주요 원인?
    * 메인 스레드에서 네트워크/IO 수행, 긴 루프, Deadlock, 무거운 UI 작업
  + 해결 방법?
    * 모든 무거운 작업은 백그라운드 스레드로
    * StrictMode, WorkManager, Coroutine, 최적화된 UI 사용
  + ANR 발생 시 분석 방법은?
    * /data/anr/traces.txt 확인
    * Perfetto/Systrace로 스레드 상태 및 부하 분석 


---
