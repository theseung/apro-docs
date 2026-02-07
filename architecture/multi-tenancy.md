# 멀티테넌시

## 개요

시스템은 회사(Company) 기반의 멀티테넌시를 구현하여 각 기업의 데이터를 격리합니다.

## 데이터 격리 구조

```
┌─────────────────────────────────────────────────────────────┐
│                      Companies 테이블                         │
├─────────────────────────────────────────────────────────────┤
│  ID=1 │ 테크솔루션   │ 슈퍼관리자 (전체 접근)               │
│  ID=2 │ 디지털마케팅  │ 슈퍼관리자 (전체 접근)               │
│  ID=3 │ A기업       │ 일반 기업                            │
│  ID=4 │ B기업       │ 일반 기업 (agency_id=3, A기업 대행)   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
         ┌─────────────────────────────────────┐
         │        company_id로 모든 데이터 격리   │
         ├─────────────────────────────────────┤
         │  users.company_id                   │
         │  personalities.company_id           │
         │  personality_applicants.company_id  │
         │  reputation_tasks.company_id        │
         │  multifaceted_tasks.company_id      │
         │  notices.company_id                 │
         │  ...                                │
         └─────────────────────────────────────┘
```

## CheckCompanyAccess 미들웨어

### 위치

`app/Http/Middleware/CheckCompanyAccess.php`

### 동작 원리

```php
public function handle(Request $request, Closure $next)
{
    $user = $request->user();

    // 1. 슈퍼 관리자 바이패스 (company_id = 1, 2)
    if (in_array($user->company_id, [1, 2])) {
        return $next($request);
    }

    // 2. 요청에서 company_id 추출
    $requestedCompanyId = $this->extractCompanyId($request);

    // 3. 접근 가능한 회사 목록 조회
    $accessibleCompanyIds = self::getAccessibleCompanyIds($user);

    // 4. 접근 권한 검증
    if (!in_array($requestedCompanyId, $accessibleCompanyIds)) {
        return response()->json([
            'success' => false,
            'error_message' => '해당 회사에 접근 권한이 없습니다.'
        ], 403);
    }

    return $next($request);
}
```

### 접근 가능한 회사 조회

```php
public static function getAccessibleCompanyIds($user): array
{
    // 슈퍼 관리자: 모든 회사
    if (in_array($user->company_id, [1, 2])) {
        return Company::pluck('id')->toArray();
    }

    // 일반 사용자: 본인 회사 + 대행 회사
    $companyIds = [$user->company_id];

    // 대행하는 회사들 추가
    $agencyCompanies = Company::where('agency_id', $user->company_id)
        ->pluck('id')
        ->toArray();

    return array_merge($companyIds, $agencyCompanies);
}
```

### company_id 추출 위치

미들웨어는 다음 순서로 company_id를 찾습니다:

1. URL 파라미터: `/company/{id}`
2. 쿼리 파라미터: `?company_id=123`
3. Request Body: `{ "company_id": 123 }`
4. 관련 엔티티에서 추출 (예: personality_idx로 조회 후 company_id 확인)

## 대행사 구조

### 계층 관계

```
Company A (agency_id: null)
├── Company B (agency_id: A)
├── Company C (agency_id: A)
└── Company D (agency_id: A)
    └── Company E (agency_id: D)  // 2단계 대행
```

### 모델 관계

```php
// Company.php
public function agency()
{
    return $this->belongsTo(Company::class, 'agency_id');
}

public function agencies()
{
    return $this->hasMany(Company::class, 'agency_id');
}

public function agencyName()
{
    return $this->agency ? $this->agency->name : null;
}
```

### 대행사 접근 권한

A기업 사용자가 접근 가능한 데이터:
- A기업의 모든 데이터
- B, C, D기업(A가 대행)의 모든 데이터
- E기업(D가 대행)은 접근 불가 (직접 대행 관계 아님)

## 라우트에 미들웨어 적용

```php
// routes/api.php

// 회사 접근 제어가 필요한 라우트
Route::middleware(['auth:sanctum', 'company.access'])->group(function () {
    Route::get('/company/{id}', [CompanyController::class, 'get']);
    Route::get('/personality/task/{idx}', [PersonalityController::class, 'show']);
});

// 회사 접근 제어 없이 사용자 회사만
Route::middleware(['auth:sanctum'])->group(function () {
    Route::get('/user/me', [UserController::class, 'me']);
});
```

## 컨트롤러에서 활용

### 자동 필터링

```php
public function index(Request $request)
{
    $user = $request->user();
    $accessibleCompanyIds = CheckCompanyAccess::getAccessibleCompanyIds($user);

    $personalities = Personality::whereIn('company_id', $accessibleCompanyIds)
        ->paginate(20);

    return response()->json([
        'success' => true,
        'data' => $personalities
    ]);
}
```

### 생성 시 회사 자동 할당

```php
public function store(Request $request)
{
    $validated = $request->validate([...]);

    // 요청에 company_id가 없으면 사용자 회사로 설정
    if (!isset($validated['company_id'])) {
        $validated['company_id'] = $request->user()->company_id;
    }

    $personality = Personality::create($validated);

    return response()->json([
        'success' => true,
        'data' => $personality
    ]);
}
```

## 보안 고려사항

1. **모든 데이터 쿼리에 company_id 필터링** 필수
2. **관계를 통한 접근도 검증** (예: applicant → personality → company)
3. **슈퍼 관리자 권한 최소화** (실제 운영에서는 제한적 사용)
4. **대행 관계 변경 시 기존 데이터 접근 검토**
