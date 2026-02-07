# 시스템 개요

## 전체 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│                        클라이언트                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │    apro     │    │  apro_user  │    │  외부 API   │         │
│  │  (관리자)    │    │   (응시자)   │    │   연동      │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                   │                   │               │
└─────────┼───────────────────┼───────────────────┼───────────────┘
          │                   │                   │
          ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                      htdocs (Laravel API)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ auth:sanctum │  │applicant.auth│  │   api.key    │          │
│  │  (관리자)     │  │   (응시자)    │  │  (외부 API)  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         │                   │                   │               │
│         ▼                   ▼                   ▼               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    company.access                        │   │
│  │                   (멀티테넌시 미들웨어)                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     Controllers                          │   │
│  │  Personality | Reputation | Multifaceted | Dashboard    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                       Models                             │   │
│  │  Company | User | Personality* | Reputation* | Multi*   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
└──────────────────────────────┼───────────────────────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │      SQLite/MySQL    │
                    │      Database        │
                    └─────────────────────┘
```

## 기술 스택

### 백엔드

| 기술 | 버전 | 용도 |
|-----|-----|-----|
| PHP | 8.2+ | 서버 사이드 |
| Laravel | 12.x | 웹 프레임워크 |
| Sanctum | 4.x | API 인증 |
| Inertia.js | 2.x | SPA 프레임워크 |
| DOMPDF | 3.x | PDF 생성 |
| PhpSpreadsheet | 5.x | Excel 처리 |
| mPDF | 8.x | PDF 생성 (대안) |
| Browsershot | 5.x | 스크린샷/PDF |

### 프론트엔드 (apro)

| 기술 | 버전 | 용도 |
|-----|-----|-----|
| React | 18.x | UI 라이브러리 |
| Material-UI | 7.x | UI 컴포넌트 |
| React Router | 7.x | 라우팅 |
| React Hook Form | 7.x | 폼 관리 |
| Zod | 3.x | 스키마 검증 |
| Day.js | 1.x | 날짜 처리 |
| Recharts | 2.x | 차트 |
| i18next | 25.x | 다국어 |

### 프론트엔드 (apro_user)

| 기술 | 버전 | 용도 |
|-----|-----|-----|
| React | 19.x | UI 라이브러리 |
| TypeScript | 5.x | 타입 안전성 |
| Vite | 7.x | 빌드 도구 |
| Material-UI | 7.x | UI 컴포넌트 |
| React Router | 7.x | 라우팅 |

## 주요 디자인 패턴

### 1. 멀티테넌시 (Multi-tenancy)

모든 데이터는 `company_id`로 격리됩니다.
- `CheckCompanyAccess` 미들웨어가 모든 요청을 검증
- 슈퍼 관리자(company_id=1,2)는 모든 회사 접근 가능
- 대행사 구조로 계층적 접근 제어

### 2. 듀얼 인증 (Dual Authentication)

두 가지 인증 시스템 운영:
- **관리자**: Laravel Sanctum (`auth:sanctum`)
- **응시자**: 커스텀 토큰 (`applicant.auth`)

### 3. UUID 기반 식별자

외부 노출 엔티티는 UUID 사용:
- `personalities.idx`
- `personality_applicants.idx`
- `reputation_tasks.idx`
- `reputation_raters.idx`
- 등

### 4. JSON 필드 활용

유연한 데이터 저장:
- `personality_sheets.code` - 코드 배열
- `personality_sheets.problem` - 문항 배열
- `personality_applicants.answer` - 응답 데이터
- `personality_applicants.scoring_result` - 채점 결과

## 폴더 구조

```
xampp/
├── htdocs/                 # Laravel 백엔드
│   ├── app/
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   │   ├── Api/    # API 컨트롤러
│   │   │   │   └── Auth/   # 인증 컨트롤러
│   │   │   └── Middleware/ # 커스텀 미들웨어
│   │   └── Models/         # Eloquent 모델
│   ├── config/             # 설정 파일
│   ├── database/
│   │   ├── migrations/     # 마이그레이션
│   │   └── seeders/        # 시더
│   ├── routes/
│   │   ├── api.php         # API 라우트
│   │   └── web.php         # 웹 라우트
│   └── resources/          # 뷰/에셋
│
├── apro/                   # 관리자 React 앱
│   └── src/
│       ├── components/     # 컴포넌트
│       ├── pages/          # 페이지
│       ├── hooks/          # 커스텀 훅
│       ├── lib/            # 유틸리티
│       ├── config/         # 설정
│       └── routes/         # 라우팅
│
└── apro_user/              # 응시자 React 앱
    └── src/
        ├── pages/          # 페이지
        ├── components/     # 컴포넌트
        ├── hooks/          # 커스텀 훅
        └── assets/         # 정적 에셋
```
