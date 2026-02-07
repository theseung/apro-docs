# 미들웨어

## 커스텀 미들웨어 목록

| 미들웨어 | 별칭 | 위치 | 역할 |
|---------|-----|-----|-----|
| CheckCompanyAccess | company.access | app/Http/Middleware/ | 멀티테넌시 데이터 격리 |
| ApplicantAuthenticate | applicant.auth | app/Http/Middleware/ | 응시자 토큰 검증 |
| VerifyApiKey | api.key | app/Http/Middleware/ | 외부 API Key 검증 |

---

## CheckCompanyAccess

### 역할

다중 테넌트 환경에서 회사별 데이터 격리를 강제합니다.

### 전체 코드

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use App\Models\Company;

class CheckCompanyAccess
{
    public function handle(Request $request, Closure $next)
    {
        $user = $request->user();

        if (!$user) {
            return response()->json([
                'success' => false,
                'error_message' => '인증이 필요합니다.'
            ], 401);
        }

        // 슈퍼 관리자 바이패스
        if (in_array($user->company_id, [1, 2])) {
            return $next($request);
        }

        // 요청에서 company_id 추출
        $requestedCompanyId = $this->extractCompanyId($request);

        if ($requestedCompanyId) {
            $accessibleCompanyIds = self::getAccessibleCompanyIds($user);

            if (!in_array($requestedCompanyId, $accessibleCompanyIds)) {
                return response()->json([
                    'success' => false,
                    'error_message' => '해당 회사에 접근 권한이 없습니다.'
                ], 403);
            }
        }

        return $next($request);
    }

    private function extractCompanyId(Request $request): ?int
    {
        // 1. URL 파라미터
        if ($request->route('id') && is_numeric($request->route('id'))) {
            return (int) $request->route('id');
        }

        // 2. 쿼리 파라미터
        if ($request->query('company_id')) {
            return (int) $request->query('company_id');
        }

        // 3. Request Body
        if ($request->input('company_id')) {
            return (int) $request->input('company_id');
        }

        return null;
    }

    public static function getAccessibleCompanyIds($user): array
    {
        // 슈퍼 관리자: 모든 회사
        if (in_array($user->company_id, [1, 2])) {
            return Company::pluck('id')->toArray();
        }

        // 본인 회사
        $companyIds = [$user->company_id];

        // 대행하는 회사들
        $agencyCompanies = Company::where('agency_id', $user->company_id)
            ->pluck('id')
            ->toArray();

        return array_merge($companyIds, $agencyCompanies);
    }
}
```

### 사용 예시

```php
// routes/api.php
Route::middleware(['auth:sanctum', 'company.access'])->group(function () {
    Route::get('/company/{id}', [CompanyController::class, 'get']);
    Route::get('/personality', [PersonalityController::class, 'index']);
});
```

---

## ApplicantAuthenticate

### 역할

응시자(Applicant) 전용 토큰을 검증하고, 요청에 응시자 정보를 주입합니다.

### 전체 코드

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Laravel\Sanctum\PersonalAccessToken;
use App\Models\PersonalityApplicant;
use App\Models\ReputationApplicant;
use App\Models\MultifacetedApplicant;

class ApplicantAuthenticate
{
    public function handle(Request $request, Closure $next)
    {
        $token = $request->bearerToken();

        if (!$token) {
            return response()->json([
                'success' => false,
                'error_message' => '인증 토큰이 없습니다.'
            ], 401);
        }

        // Sanctum 토큰 검증
        $accessToken = PersonalAccessToken::findToken($token);

        if (!$accessToken) {
            return response()->json([
                'success' => false,
                'error_message' => '토큰이 유효하지 않거나 만료되었습니다.'
            ], 401);
        }

        // applicant ability 확인
        if (!$accessToken->can('applicant')) {
            return response()->json([
                'success' => false,
                'error_message' => '응시자 권한이 없습니다.'
            ], 403);
        }

        // tokenable 모델 확인
        $applicant = $accessToken->tokenable;
        $type = null;

        if ($applicant instanceof PersonalityApplicant) {
            $type = 'personality';
        } elseif ($applicant instanceof ReputationApplicant) {
            $type = 'reputation';
        } elseif ($applicant instanceof MultifacetedApplicant) {
            $type = 'multifaceted';
        } else {
            return response()->json([
                'success' => false,
                'error_message' => '유효하지 않은 응시자 타입입니다.'
            ], 403);
        }

        // 요청에 응시자 정보 주입
        $request->attributes->set('applicant', $applicant);
        $request->attributes->set('applicant_type', $type);

        return $next($request);
    }
}
```

### 컨트롤러에서 사용

```php
public function info(Request $request)
{
    $applicant = $request->attributes->get('applicant');
    $type = $request->attributes->get('applicant_type');

    return response()->json([
        'success' => true,
        'data' => [
            'applicant' => $applicant,
            'type' => $type
        ]
    ]);
}
```

### 라우트 등록

```php
// routes/api.php
Route::middleware('applicant.auth')->group(function () {
    Route::get('/applicant/info', [ApplicantController::class, 'info']);
    Route::post('/applicant/answer', [ApplicantController::class, 'answerStore']);
    Route::post('/applicant/end', [ApplicantController::class, 'end']);
});
```

---

## VerifyApiKey

### 역할

외부 시스템 연동을 위한 API Key 검증을 수행합니다.

### 전체 코드

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class VerifyApiKey
{
    public function handle(Request $request, Closure $next)
    {
        // CORS preflight 요청 통과
        if ($request->isMethod('OPTIONS')) {
            return $next($request);
        }

        // API Key 추출 (헤더 또는 쿼리)
        $apiKey = $request->header('X-API-Key')
            ?? $request->query('api_key');

        if (!$apiKey) {
            return response()->json([
                'success' => false,
                'error_message' => 'API Key가 필요합니다.'
            ], 401);
        }

        // 환경 변수의 키와 비교
        if ($apiKey !== env('PUBLIC_API_KEY')) {
            return response()->json([
                'success' => false,
                'error_message' => '유효하지 않은 API Key입니다.'
            ], 401);
        }

        // CORS 헤더 추가
        $response = $next($request);

        return $response->header('Access-Control-Allow-Origin', '*')
            ->header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
            ->header('Access-Control-Allow-Headers', 'Content-Type, X-API-Key');
    }
}
```

### 라우트 등록

```php
// routes/api.php
Route::middleware('api.key')->prefix('public')->group(function () {
    Route::post('/report/url', [PublicReportController::class, 'getReportUrl']);
    Route::post('/project/list', [PublicReportController::class, 'getProjectList']);
    Route::post('/applicant/list', [PublicReportController::class, 'getApplicantList']);
});
```

---

## 미들웨어 등록

### bootstrap/app.php

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'company.access' => \App\Http\Middleware\CheckCompanyAccess::class,
            'applicant.auth' => \App\Http\Middleware\ApplicantAuthenticate::class,
            'api.key' => \App\Http\Middleware\VerifyApiKey::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

---

## 미들웨어 조합 패턴

### 관리자 전용 (회사 제한 없음)

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user/me', [UserController::class, 'me']);
});
```

### 관리자 + 회사 접근 제어

```php
Route::middleware(['auth:sanctum', 'company.access'])->group(function () {
    Route::get('/company/{id}', [CompanyController::class, 'get']);
    Route::get('/personality', [PersonalityController::class, 'index']);
});
```

### 응시자 전용

```php
Route::middleware('applicant.auth')->group(function () {
    Route::get('/applicant/info', [ApplicantController::class, 'info']);
    Route::post('/applicant/end', [ApplicantController::class, 'end']);
});
```

### 외부 API

```php
Route::middleware('api.key')->group(function () {
    Route::post('/public/report/url', [PublicReportController::class, 'getReportUrl']);
});
```

### 인증 없음 (공개)

```php
// 미들웨어 없이 직접 등록
Route::post('/applicant/login', [ApplicantController::class, 'login']);
Route::get('/company-info', [CompanyController::class, 'info']);
Route::get('/rater/{rater_idx}/info', [RaterController::class, 'info']);
```
