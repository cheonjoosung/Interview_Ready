# 🏗 Architecture

Android 앱 아키텍처 패턴을 정리한 문서입니다.

## 📚 목차

| # | 주제 | 파일 |
|---|------|------|
| 01 | MVVM | [01_mvvm.md](./topics/01_mvvm.md) |
| 02 | Clean Architecture | [02_clean_architecture.md](./topics/02_clean_architecture.md) |
| 03 | MVI | [03_mvi.md](./topics/03_mvi.md) |
| 04 | Repository 패턴 | [04_repository_pattern.md](./topics/04_repository_pattern.md) |

## 패턴 비교

| 패턴 | 복잡도 | 테스트 | 적합한 프로젝트 |
|------|--------|--------|-----------------|
| MVVM | 낮음 | 보통 | 일반적인 Android 앱 |
| MVVM + Clean | 높음 | 높음 | 대규모, 장기 유지보수 |
| MVI | 중간 | 높음 | 복잡한 상태 관리 필요 시 |

> Google 공식 권장: **MVVM + Repository + (필요시) UseCase**
