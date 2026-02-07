# 컨트롤러

## 주요 컨트롤러 목록

### API 컨트롤러 (`app/Http/Controllers/Api/`)

| 컨트롤러 | 크기 | 주요 기능 |
|---------|-----|---------|
| PersonalityController | 141KB | 인성검사 전체 관리 |
| ReputationTaskController | 79KB | 평판진단 관리 |
| ApplicantController | 72KB | 응시자 인증/응시 |
| MultifacetedTaskController | 71KB | 다면평가 관리 |
| RaterController | 33KB | 평가자 평가 진행 |
| DashboardController | 25KB | 대시보드 통계 |
| CompanyController | 21KB | 기업 관리 |
| UserController | 18KB | 사용자 인증/관리 |
| NoticeController | 12KB | 공지사항 |
| PublicReportController | 11KB | 외부 공개 API |

---

## PersonalityController

인성검사의 모든 기능을 담당하는 핵심 컨트롤러입니다.

### 검사 관리

```php
// 목록 조회 (페이지네이션, 필터링)
public function index(Request $request)

// 검사 생성
public function store(Request $request)

// 검사 상세
public function show($idx)

// 검사 수정
public function update(Request $request, $idx)

// 검사 삭제
public function destroy($idx)

// 상태 토글 (중지/재개)
public function toggleStatus($idx)
```

### 응시자 관리

```php
// 응시자 목록
public function applicants(Request $request, $idx)

// 단일 응시자 추가
public function applicantStore(Request $request, $idx)

// 대량 응시자 추가 (Excel)
public function applicantBulkStore(Request $request, $idx)

// 응시자 상세
public function applicantShow($idx, $applicantIdx)

// 응시자 수정
public function applicantUpdate(Request $request, $idx, $applicantIdx)

// 응시자 삭제
public function applicantDestroy($idx, $applicantIdx)

// 채점 상태 초기화
public function applicantReset($idx, $applicantIdx)

// 전체 삭제
public function applicantRemoveAll($idx)
```

### 코드/문항/검사지 관리

```php
// 코드 CRUD
public function code(Request $request)
public function codeStore(Request $request)
public function codeDetail($id)
public function codeUpdate(Request $request, $id)
public function codeDelete($id)

// 문항 CRUD
public function question(Request $request)
public function questionStore(Request $request)
public function questionDetail($id)
public function questionUpdate(Request $request, $id)
public function questionDelete($id)

// 검사지 CRUD
public function sheet(Request $request)
public function sheetStore(Request $request)
public function sheetDetail($id)
public function sheetUpdate(Request $request, $id)
public function sheetDelete($id)
public function sheetActive($id)  // 활성화 토글
public function sheetCopy($id)    // 복제
```

### 다운로드/업로드

```php
// 엑셀 다운로드
public function downloadExcel($idx)
public function downloadAnswer($idx)
public function sheetDownload($id)

// 결과 업로드
public function resultUpload(Request $request, $idx)
```

---

## ApplicantController

3가지 타입의 응시자를 통합 처리합니다.

### 인증

```php
public function login(Request $request)
{
    // 1. company 헤더로 회사 조회
    $companyCode = $request->header('company');
    $company = Company::where('code', $companyCode)->first();

    // 2. 3가지 타입에서 응시자 검색
    $applicant = null;
    $type = null;

    // PersonalityApplicant 검색
    $applicant = PersonalityApplicant::where('company_id', $company->id)
        ->where('id', $request->id)
        ->where('pw', $request->pw)
        ->first();
    if ($applicant) $type = 'personality';

    // ReputationApplicant 검색 (task 기반)
    if (!$applicant) {
        $applicant = ReputationApplicant::whereHas('task', function($q) use ($company) {
            $q->where('company_id', $company->id);
        })->where('id', $request->id)->where('pw', $request->pw)->first();
        if ($applicant) $type = 'reputation';
    }

    // MultifacetedApplicant 검색
    if (!$applicant) {
        // 동일한 패턴
        $type = 'multifaceted';
    }

    // 3. 검증
    if ($applicant->login_count >= 2) {
        return response()->json(['error_message' => '로그인 횟수 초과'], 401);
    }

    // 4. 토큰 발급
    $token = $applicant->createToken('applicant_' . $type, ['applicant']);

    return response()->json([
        'success' => true,
        'data' => [
            'applicant' => $applicant,
            'access_token' => $token->plainTextToken,
            'type' => $type
        ]
    ]);
}
```

### 응시 진행

```php
// 응시자 정보
public function info(Request $request)

// 검사 시작
public function start(Request $request)

// 문항 조회 (10개씩)
public function questionsBulk(Request $request)

// 답변 저장
public function answerStore(Request $request)

// 임시 저장
public function saveTemp(Request $request)

// 서술형 제출
public function submitDescriptive(Request $request)

// 검사 종료
public function end(Request $request)
```

---

## RaterController

평가자(Rater)의 평가 진행을 처리합니다. **인증 없이** UUID로 직접 접근합니다.

```php
// 평가자 정보 조회
public function info($rater_idx)

// 평가자 정보 수정
public function updateInfo(Request $request, $rater_idx)

// Likert 척도 문항 조회
public function getLikertQuestions($rater_idx)

// 서술형 문항 조회
public function getDescriptiveQuestions($rater_idx)

// 객관식 답변 저장
public function saveAnswers(Request $request, $rater_idx)

// 서술형 답변 저장
public function saveDescriptiveAnswers(Request $request, $rater_idx)

// 답변 조회
public function getAnswers($rater_idx)

// 평가 상태
public function getStatus($rater_idx)

// 평가 완료
public function end($rater_idx)
```

---

## UserController

### 인증

```php
// 로그인
public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required'
    ]);

    if (!Auth::attempt($credentials)) {
        return response()->json([
            'success' => false,
            'error_message' => '이메일 또는 비밀번호가 일치하지 않습니다.'
        ], 401);
    }

    $user = Auth::user();
    $token = $user->createToken('auth_token')->plainTextToken;

    return response()->json([
        'success' => true,
        'data' => [
            'user' => $user,
            'access_token' => $token
        ]
    ]);
}

// 토큰 갱신
public function refreshToken(Request $request)

// 로그아웃
public function logout(Request $request)
```

### 마이페이지

```php
// 내 정보
public function me(Request $request)

// 마이페이지
public function mypage(Request $request)

// 정보 수정
public function update(Request $request)

// 비밀번호 변경
public function updatePassword(Request $request)
```

### 회사 관리자 관리

```php
// 관리자 추가
public function store(Request $request)

// 관리자 목록
public function getCompanyUsers(Request $request)

// 관리자 수정
public function updateCompanyUser(Request $request, $id)

// 관리자 삭제
public function deleteCompanyUser($id)
```

---

## PublicReportController

외부 시스템 연동용 공개 API입니다.

```php
// 보고서 URL 생성
public function getReportUrl(Request $request)
{
    // API Key 검증은 VerifyApiKey 미들웨어에서 처리

    $validated = $request->validate([
        'company_code' => 'required|string',
        'personality_idx' => 'required|string',
        'applicant_idx' => 'required|string',
        // 블라인드 옵션 (선택)
        'blind_name' => 'boolean',
        'blind_field' => 'boolean',
    ]);

    // 토큰 생성
    $token = ReportAccessToken::createToken([
        'company_id' => $company->id,
        'personality_idx' => $validated['personality_idx'],
        'applicant_idx' => $validated['applicant_idx'],
        'blind_name_fg' => $validated['blind_name'] ?? false,
        // ...
    ]);

    return response()->json([
        'success' => true,
        'data' => [
            'report_url' => "https://report.aprofiler.co.kr/view/{$token->token}",
            'expires_at' => $token->expires_at
        ]
    ]);
}

// 프로젝트 목록
public function getProjectList(Request $request)

// 지원자 목록
public function getApplicantList(Request $request)
```

---

## 응답 형식

### 성공 응답

```php
return response()->json([
    'success' => true,
    'message' => '저장되었습니다.',  // 선택
    'data' => $data                  // 선택
]);
```

### 에러 응답

```php
return response()->json([
    'success' => false,
    'error_message' => '에러 메시지'
], 400);  // 적절한 HTTP 상태 코드
```

### 페이지네이션

```php
return response()->json([
    'success' => true,
    'data' => $items,
    'meta' => [
        'current_page' => $paginator->currentPage(),
        'last_page' => $paginator->lastPage(),
        'per_page' => $paginator->perPage(),
        'total' => $paginator->total()
    ]
]);
```
