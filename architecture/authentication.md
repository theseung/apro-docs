# 인증 시스템

## 듀얼 인증 아키텍처

시스템은 두 가지 독립적인 인증 시스템을 운영합니다.

```
┌─────────────────────────────────────────────────────────────┐
│                     인증 시스템 구조                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────┐    ┌─────────────────────┐        │
│  │   관리자 인증         │    │    응시자 인증        │        │
│  │   (auth:sanctum)    │    │  (applicant.auth)   │        │
│  ├─────────────────────┤    ├─────────────────────┤        │
│  │ 대상: User 모델      │    │ 대상:               │        │
│  │ 토큰: Sanctum       │    │ - PersonalityApplicant│       │
│  │ 저장: personal_     │    │ - ReputationApplicant │       │
│  │       access_tokens │    │ - MultifacetedApplicant│      │
│  │                     │    │ - ReputationRater     │       │
│  │                     │    │ - MultifacetedRater   │       │
│  └─────────────────────┘    └─────────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 1. 관리자 인증 (auth:sanctum)

### 로그인 플로우

```
POST /api/auth/login
├── 이메일/비밀번호 검증
├── Sanctum 토큰 발급
└── 응답: { access_token, user }
```

### 사용 예시

```php
// 라우트 정의
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user/me', [UserController::class, 'me']);
});

// 컨트롤러에서 사용자 접근
$user = $request->user();
$companyId = $user->company_id;
```

### 토큰 갱신

```
POST /api/auth/refresh-token
└── 기존 토큰 폐기 → 새 토큰 발급
```

### 관련 엔드포인트

| 엔드포인트 | 메서드 | 설명 |
|-----------|-------|------|
| /api/auth/login | POST | 로그인 |
| /api/auth/refresh-token | POST | 토큰 갱신 |
| /api/auth/logout | POST | 로그아웃 |
| /api/auth/forgot-password | POST | 비밀번호 초기화 |

## 2. 응시자 인증 (applicant.auth)

### 로그인 플로우

```
POST /api/applicant/login
Headers: { company: "{회사코드}" }
Body: { id, pw }
├── 회사 코드로 Company 조회
├── 3가지 응시자 테이블에서 순차 검색
│   ├── PersonalityApplicant
│   ├── ReputationApplicant (task 기반)
│   └── MultifacetedApplicant (task 기반)
├── 로그인 횟수 검증 (최대 2회)
├── 테스트 기간 검증
├── Sanctum 토큰 발급 (applicant ability)
└── 응답: { applicant, access_token, type }
```

### 미들웨어 동작

`ApplicantAuthenticate.php`:

```php
public function handle(Request $request, Closure $next)
{
    // 1. Authorization 헤더에서 토큰 추출
    $token = $request->bearerToken();

    // 2. Sanctum PersonalAccessToken 검증
    $accessToken = PersonalAccessToken::findToken($token);

    // 3. applicant ability 확인
    if (!$accessToken->can('applicant')) {
        return response()->json(['error' => 'Unauthorized'], 403);
    }

    // 4. tokenable 모델이 Applicant 타입인지 확인
    $applicant = $accessToken->tokenable;

    // 5. request에 applicant 주입
    $request->attributes->set('applicant', $applicant);
    $request->attributes->set('applicant_type', $type);

    return $next($request);
}
```

### 컨트롤러에서 응시자 접근

```php
public function info(Request $request)
{
    $applicant = $request->attributes->get('applicant');
    $type = $request->attributes->get('applicant_type');

    return response()->json([
        'success' => true,
        'data' => $applicant
    ]);
}
```

### 관련 엔드포인트

| 엔드포인트 | 메서드 | 설명 |
|-----------|-------|------|
| /api/applicant/login | POST | 응시자 로그인 |
| /api/applicant/logout | POST | 응시자 로그아웃 |
| /api/applicant/info | GET | 응시자 정보 |

## 3. 평가자 인증 (rater_idx 기반)

평가자(Rater)는 별도 인증 없이 UUID로 직접 접근합니다.

```
GET /api/rater/{rater_idx}/info
├── rater_idx UUID로 ReputationRater 또는 MultifacetedRater 조회
└── 인증 없이 접근 가능 (URL 기반)
```

### 보안 고려사항

- UUID는 추측하기 어려움 (36자 랜덤 문자열)
- 이메일 발송된 링크로만 접근
- 완료된 평가자는 수정 불가

## 4. 외부 API 인증 (api.key)

외부 시스템 연동을 위한 API Key 방식:

```
POST /api/public/report/url
Headers: { X-API-Key: "{API_KEY}" }
```

### 미들웨어 동작

`VerifyApiKey.php`:

```php
public function handle(Request $request, Closure $next)
{
    // OPTIONS 요청 통과 (CORS preflight)
    if ($request->isMethod('OPTIONS')) {
        return $next($request);
    }

    // API Key 검증
    $apiKey = $request->header('X-API-Key') ?? $request->query('api_key');

    if ($apiKey !== env('PUBLIC_API_KEY')) {
        return response()->json(['error' => 'Invalid API Key'], 401);
    }

    return $next($request);
}
```

## 토큰 저장 (프론트엔드)

### apro (관리자)

```javascript
// lib/api-client.js
export function setAccessToken(token) {
    localStorage.setItem('access_token', token);
}

export function getAccessToken() {
    return localStorage.getItem('access_token');
}
```

### apro_user (응시자)

```javascript
// api-client.js
// 동일한 방식으로 localStorage 사용
```

## 세션 관리

- 토큰 만료: 설정에 따름 (`config/sanctum.php`)
- 갱신: `/api/auth/refresh-token` 호출
- 로그아웃: 토큰 삭제 및 서버측 무효화
