# 라우팅

## 라우트 파일

| 파일 | 크기 | 역할 |
|-----|-----|-----|
| routes/api.php | 25KB | API 엔드포인트 |
| routes/web.php | 40KB | 웹 라우트 (테스트용) |

---

## API 라우트 구조 (`routes/api.php`)

### 인증 라우트 (비보호)

```php
// 관리자 인증
Route::prefix('auth')->group(function () {
    Route::post('/login', [UserController::class, 'login']);
    Route::post('/refresh-token', [UserController::class, 'refreshToken']);
    Route::post('/logout', [UserController::class, 'logout']);
    Route::post('/forgot-password', [UserController::class, 'forgotPassword']);
});

// 응시자 로그인
Route::post('/applicant/login', [ApplicantController::class, 'login']);

// 기업 정보 (로고, 이름)
Route::get('/company-info', [CompanyController::class, 'info']);
```

### 관리자 보호 라우트

```php
Route::middleware('auth:sanctum')->group(function () {

    // 사용자 정보
    Route::prefix('user')->group(function () {
        Route::get('/me', [UserController::class, 'me']);
        Route::get('/mypage', [UserController::class, 'mypage']);
        Route::post('/update', [UserController::class, 'update']);
        Route::post('/update-password', [UserController::class, 'updatePassword']);
    });

    // 기업 관리 (회사 접근 제어 포함)
    Route::middleware('company.access')->group(function () {

        // 기업 CRUD
        Route::prefix('company')->group(function () {
            Route::get('/', [CompanyController::class, 'index']);
            Route::post('/', [CompanyController::class, 'store']);
            Route::get('/{id}', [CompanyController::class, 'get']);
            Route::post('/{id}', [CompanyController::class, 'update']);
            Route::delete('/{id}', [CompanyController::class, 'destroy']);
            Route::post('/{id}/logo', [CompanyController::class, 'companyLogo']);
        });

        // 인성검사
        Route::prefix('personality')->group(function () {
            // 검사 CRUD
            Route::get('/', [PersonalityController::class, 'index']);
            Route::post('/', [PersonalityController::class, 'store']);
            Route::get('/task/{idx}', [PersonalityController::class, 'show']);
            Route::put('/task/{idx}', [PersonalityController::class, 'update']);
            Route::delete('/task/{idx}', [PersonalityController::class, 'destroy']);
            Route::patch('/task/{idx}/toggle-status', [PersonalityController::class, 'toggleStatus']);

            // 응시자 관리
            Route::get('/task/{idx}/applicants', [PersonalityController::class, 'applicants']);
            Route::post('/task/{idx}/applicants', [PersonalityController::class, 'applicantStore']);
            Route::post('/task/{idx}/applicants/bulk', [PersonalityController::class, 'applicantBulkStore']);
            Route::get('/task/{idx}/applicants/{applicantIdx}', [PersonalityController::class, 'applicantShow']);
            Route::put('/task/{idx}/applicants/{applicantIdx}', [PersonalityController::class, 'applicantUpdate']);
            Route::delete('/task/{idx}/applicants/{applicantIdx}', [PersonalityController::class, 'applicantDestroy']);
            Route::post('/task/{idx}/applicants/{applicantIdx}/reset', [PersonalityController::class, 'applicantReset']);

            // 다운로드
            Route::post('/task/{idx}/download-excel', [PersonalityController::class, 'downloadExcel']);
            Route::get('/task/{idx}/download-answer', [PersonalityController::class, 'downloadAnswer']);

            // 코드
            Route::get('/code', [PersonalityController::class, 'code']);
            Route::post('/code', [PersonalityController::class, 'codeStore']);
            Route::get('/code/{id}', [PersonalityController::class, 'codeDetail']);
            Route::put('/code/{id}', [PersonalityController::class, 'codeUpdate']);
            Route::delete('/code/{id}', [PersonalityController::class, 'codeDelete']);

            // 문항
            Route::get('/question', [PersonalityController::class, 'question']);
            Route::post('/question', [PersonalityController::class, 'questionStore']);
            Route::get('/question/{id}', [PersonalityController::class, 'questionDetail']);
            Route::put('/question/{id}', [PersonalityController::class, 'questionUpdate']);
            Route::delete('/question/{id}', [PersonalityController::class, 'questionDelete']);

            // 검사지
            Route::get('/sheet', [PersonalityController::class, 'sheet']);
            Route::post('/sheet', [PersonalityController::class, 'sheetStore']);
            Route::get('/sheet/{id}', [PersonalityController::class, 'sheetDetail']);
            Route::put('/sheet/{id}', [PersonalityController::class, 'sheetUpdate']);
            Route::delete('/sheet/{id}', [PersonalityController::class, 'sheetDelete']);
            Route::patch('/sheet/{id}/active', [PersonalityController::class, 'sheetActive']);
            Route::post('/sheet/{id}/copy', [PersonalityController::class, 'sheetCopy']);
        });

        // 평판진단 (유사 구조)
        Route::prefix('reputation')->group(function () {
            // ...
        });

        // 다면평가 (유사 구조)
        Route::prefix('multifaceted')->group(function () {
            // ...
        });
    });

    // 대시보드
    Route::prefix('dashboard')->group(function () {
        Route::get('/overview', [DashboardController::class, 'overview']);
        Route::get('/all-tests', [DashboardController::class, 'allTests']);
        Route::get('/company-stats', [DashboardController::class, 'companyStats']);
    });

    // 공지사항
    Route::prefix('notice')->group(function () {
        Route::get('/', [NoticeController::class, 'index']);
        Route::post('/', [NoticeController::class, 'store']);
        Route::get('/{id}', [NoticeController::class, 'show']);
        Route::put('/{id}', [NoticeController::class, 'update']);
        Route::delete('/{id}', [NoticeController::class, 'destroy']);
        Route::patch('/{id}/toggle-status', [NoticeController::class, 'toggleStatus']);
    });
});
```

### 응시자 보호 라우트

```php
Route::middleware('applicant.auth')->prefix('applicant')->group(function () {
    Route::post('/logout', [ApplicantController::class, 'logout']);
    Route::get('/info', [ApplicantController::class, 'info']);
    Route::post('/start', [ApplicantController::class, 'start']);
    Route::get('/questions-bulk', [ApplicantController::class, 'questionsBulk']);
    Route::post('/answer', [ApplicantController::class, 'answerStore']);
    Route::post('/save-temp', [ApplicantController::class, 'saveTemp']);
    Route::post('/submit-descriptive', [ApplicantController::class, 'submitDescriptive']);
    Route::post('/end', [ApplicantController::class, 'end']);

    // 평가자 관리 (응시자가 평가자 추가)
    Route::post('/reputation-rater', [ApplicantController::class, 'addReputationRater']);
    Route::get('/reputation-rater/list', [ApplicantController::class, 'listReputationRaters']);
    Route::put('/reputation-rater/{idx}', [ApplicantController::class, 'updateReputationRater']);
    Route::delete('/reputation-rater/{idx}', [ApplicantController::class, 'deleteReputationRater']);
    Route::post('/reputation-rater/{idx}/send-email', [ApplicantController::class, 'sendRaterEmail']);
});
```

### 평가자 라우트 (비인증)

```php
// rater_idx UUID로 직접 접근
Route::prefix('rater/{rater_idx}')->group(function () {
    Route::get('/info', [RaterController::class, 'info']);
    Route::put('/info', [RaterController::class, 'updateInfo']);
    Route::get('/questions/likert', [RaterController::class, 'getLikertQuestions']);
    Route::get('/questions/descriptive', [RaterController::class, 'getDescriptiveQuestions']);
    Route::post('/answers', [RaterController::class, 'saveAnswers']);
    Route::post('/descriptive-answers', [RaterController::class, 'saveDescriptiveAnswers']);
    Route::get('/answers', [RaterController::class, 'getAnswers']);
    Route::get('/status', [RaterController::class, 'getStatus']);
    Route::get('/end', [RaterController::class, 'end']);
});
```

### 외부 공개 API

```php
Route::middleware('api.key')->prefix('public')->group(function () {
    Route::post('/report/url', [PublicReportController::class, 'getReportUrl']);
    Route::post('/project/list', [PublicReportController::class, 'getProjectList']);
    Route::post('/applicant/list', [PublicReportController::class, 'getApplicantList']);
});
```

---

## 라우트 확인 명령어

```bash
# 모든 라우트 목록
php artisan route:list

# API 라우트만
php artisan route:list --path=api

# 특정 패턴 필터
php artisan route:list --path=api/personality

# JSON 형식
php artisan route:list --json
```

---

## URL 패턴

### RESTful 패턴

```
GET    /api/{resource}              # index (목록)
POST   /api/{resource}              # store (생성)
GET    /api/{resource}/{id}         # show (상세)
PUT    /api/{resource}/{id}         # update (수정)
DELETE /api/{resource}/{id}         # destroy (삭제)
```

### 중첩 리소스

```
GET    /api/personality/task/{idx}/applicants           # 응시자 목록
POST   /api/personality/task/{idx}/applicants           # 응시자 추가
GET    /api/personality/task/{idx}/applicants/{appIdx}  # 응시자 상세
```

### 액션 라우트

```
PATCH  /api/personality/task/{idx}/toggle-status  # 상태 토글
POST   /api/personality/task/{idx}/download-excel # 다운로드
POST   /api/sheet/{id}/copy                       # 복제
```

---

## 라우트 네이밍 (선택사항)

```php
Route::get('/personality', [PersonalityController::class, 'index'])
    ->name('personality.index');

Route::post('/personality', [PersonalityController::class, 'store'])
    ->name('personality.store');
```

사용:
```php
route('personality.index');  // /api/personality
```
