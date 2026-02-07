# 백엔드 디렉토리 구조

## htdocs 폴더 구조

```
htdocs/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/                    # API 컨트롤러
│   │   │   │   ├── ApplicantController.php      # 응시자 (72KB)
│   │   │   │   ├── PersonalityController.php    # 인성검사 (141KB)
│   │   │   │   ├── CompanyController.php        # 기업 관리 (21KB)
│   │   │   │   ├── UserController.php           # 사용자 (18KB)
│   │   │   │   ├── DashboardController.php      # 대시보드 (25KB)
│   │   │   │   ├── RaterController.php          # 평가자 (33KB)
│   │   │   │   ├── ReputationTaskController.php # 평판진단 (79KB)
│   │   │   │   ├── MultifacetedTaskController.php # 다면평가 (71KB)
│   │   │   │   ├── NoticeController.php         # 공지사항 (12KB)
│   │   │   │   ├── PublicReportController.php   # 외부 API (11KB)
│   │   │   │   └── ...
│   │   │   └── Auth/                   # 인증 컨트롤러 (Laravel 기본)
│   │   └── Middleware/
│   │       ├── CheckCompanyAccess.php  # 멀티테넌시
│   │       ├── ApplicantAuthenticate.php # 응시자 인증
│   │       └── VerifyApiKey.php        # 외부 API 인증
│   │
│   └── Models/                         # Eloquent 모델
│       ├── User.php
│       ├── Company.php
│       ├── Personality.php
│       ├── PersonalityApplicant.php
│       ├── PersonalitySheet.php
│       ├── PersonalityCode.php
│       ├── PersonalityQuestion.php
│       ├── PersonalityData.php
│       ├── PersonalityDataStatistics.php
│       ├── ReputationTask.php
│       ├── ReputationApplicant.php
│       ├── ReputationRater.php
│       ├── ReputationData.php
│       ├── MultifacetedTask.php
│       ├── MultifacetedApplicant.php
│       ├── MultifacetedRater.php
│       ├── ReportAccessToken.php
│       └── Notice.php
│
├── config/
│   ├── app.php             # 앱 설정
│   ├── auth.php            # 인증 설정
│   ├── cors.php            # CORS 설정
│   ├── database.php        # DB 설정
│   ├── sanctum.php         # Sanctum 설정
│   └── ...
│
├── database/
│   ├── migrations/         # 마이그레이션 (50+개)
│   ├── seeders/            # 시더
│   │   ├── DatabaseSeeder.php
│   │   ├── CompanySeeder.php
│   │   ├── UserSeeder.php
│   │   ├── PersonalityTestTypeSeeder.php
│   │   ├── PersonalityCodeSeeder.php
│   │   ├── PersonalityQuestionSeeder.php
│   │   └── ...
│   └── seeders/            # DB 시더
│
├── routes/
│   ├── api.php             # API 라우트 (25KB)
│   └── web.php             # 웹 라우트 (40KB)
│
├── resources/
│   ├── js/                 # 프론트엔드 (미사용)
│   └── views/              # Blade 템플릿
│
├── storage/
│   ├── app/                # 업로드 파일
│   ├── framework/          # 캐시, 세션
│   └── logs/               # 로그 파일
│
├── tests/
│   ├── Feature/            # 기능 테스트
│   └── Unit/               # 유닛 테스트
│
├── vendor/                 # Composer 패키지
├── node_modules/           # npm 패키지
│
├── .env                    # 환경 설정
├── composer.json           # PHP 의존성
├── package.json            # Node 의존성
├── artisan                 # CLI
├── phpunit.xml             # PHPUnit 설정
└── vite.config.ts          # Vite 설정
```

## 주요 파일 역할

### 컨트롤러

| 파일 | 크기 | 주요 역할 |
|-----|-----|---------|
| PersonalityController | 141KB | 인성검사 CRUD, 응시자 관리, 코드/문항/검사지 관리 |
| ReputationTaskController | 79KB | 평판진단 CRUD, 피평가자/평가자 관리 |
| MultifacetedTaskController | 71KB | 다면평가 CRUD |
| ApplicantController | 72KB | 3가지 타입 응시자 통합 로그인/응시 처리 |
| RaterController | 33KB | 평가자 평가 진행 (인증 없음) |
| DashboardController | 25KB | 대시보드 통계 |
| CompanyController | 21KB | 기업 관리, 대행사 |
| UserController | 18KB | 관리자 인증, 마이페이지 |

### 모델

| 파일 | 주요 역할 |
|-----|---------|
| Company | 기업 정보, 대행사 관계 |
| User | 관리자 사용자 |
| Personality | 인성검사 (UUID idx) |
| PersonalityApplicant | 인성검사 응시자 (UUID idx) |
| PersonalitySheet | 검사지 템플릿 |
| PersonalityData | 통계용 개별 데이터 |
| ReputationTask | 평판진단 작업 (UUID idx) |
| ReputationApplicant | 피평가자 (UUID idx) |
| ReputationRater | 평가자 (UUID idx) |
| MultifacetedTask | 다면평가 작업 (UUID idx) |
| ReportAccessToken | 외부 보고서 접근 토큰 |

### 미들웨어

| 파일 | 별칭 | 역할 |
|-----|-----|-----|
| CheckCompanyAccess | company.access | 멀티테넌시 데이터 격리 |
| ApplicantAuthenticate | applicant.auth | 응시자 토큰 검증 |
| VerifyApiKey | api.key | 외부 API Key 검증 |

## 네이밍 컨벤션

### 파일명

- 컨트롤러: `{Resource}Controller.php`
- 모델: `{Model}.php` (단수형)
- 마이그레이션: `{date}_create_{table}_table.php`
- 시더: `{Model}Seeder.php`

### 메서드명 (컨트롤러)

```php
index()      // 목록 조회
store()      // 생성
show($id)    // 단일 조회
update($id)  // 수정
destroy($id) // 삭제

// 커스텀 액션
toggleStatus($id)     // 상태 토글
downloadExcel($id)    // Excel 다운로드
uploadResult($id)     // 결과 업로드
```

### 라우트 네이밍

```php
// RESTful 패턴
GET    /api/{resource}           // index
POST   /api/{resource}           // store
GET    /api/{resource}/{id}      // show
PUT    /api/{resource}/{id}      // update
DELETE /api/{resource}/{id}      // destroy

// 중첩 리소스
GET    /api/personality/task/{idx}/applicants
POST   /api/personality/task/{idx}/applicants/bulk
```
