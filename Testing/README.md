# 🧪 Testing

Android 앱 테스트 전략과 방법을 정리합니다.

## 테스트 피라미드

```
        /\
       /UI\          <- Espresso, Compose UI Test (느림, 비쌈)
      /----\
     / 통합  \        <- Repository, Room, API (중간)
    /--------\
   /  단위    \       <- ViewModel, UseCase, Logic (빠름, 많이)
  /____________\
```

## 📚 목차

| # | 주제 | 파일 |
|---|------|------|
| 01 | Unit Testing (단위 테스트) | [01_unit_testing.md](./topics/01_unit_testing.md) |

## 🔖 추가 학습 권장 주제
- Espresso (UI 테스트)
- Compose UI Testing
- Room 인메모리 DB 테스트
- MockWebServer (네트워크 테스트)
- TDD (테스트 주도 개발)
