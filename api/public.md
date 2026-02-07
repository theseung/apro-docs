# 외부 공개 API

## 개요

외부 시스템과 연동하기 위한 공개 API입니다.
API Key 인증을 사용합니다.

## 인증

모든 요청에 API Key 필요:

```http
X-API-Key: {your-api-key}
```

또는 쿼리 파라미터:

```
?api_key={your-api-key}
```

---

## 보고서 URL 생성

특정 응시자의 보고서 접근 URL을 생성합니다.

```http
POST /api/public/report/url
Content-Type: application/json
X-API-Key: {api_key}
```

**Request Body**

```json
{
  "company_code": "TS001",
  "personality_idx": "550e8400-e29b-41d4-a716-446655440000",
  "applicant_idx": "applicant-uuid",
  "blind_name": true,
  "blind_field": false,
  "blind_identifier": false,
  "blind_exam_number": false,
  "population_source": true,
  "field_rank": true
}
```

**Parameters**

| 필드 | 타입 | 필수 | 설명 |
|-----|-----|-----|------|
| company_code | string | O | 회사 코드 |
| personality_idx | string | O | 검사 UUID |
| applicant_idx | string | O | 응시자 UUID |
| blind_name | bool | X | 이름 블라인드 처리 |
| blind_field | bool | X | 분야 블라인드 처리 |
| blind_identifier | bool | X | 식별자 블라인드 처리 |
| blind_exam_number | bool | X | 수험번호 블라인드 처리 |
| population_source | bool | X | 모집단 통계 표시 |
| field_rank | bool | X | 분야별 순위 표시 |

**Response (200)**

```json
{
  "success": true,
  "data": {
    "report_url": "https://report.aprofiler.co.kr/view/abc123def456...",
    "token": "abc123def456...",
    "expires_at": "2025-01-16T10:00:00",
    "expires_in": 86400
  }
}
```

**Error Responses**

| 코드 | 상황 | 응답 |
|-----|-----|-----|
| 401 | API Key 없음 | `{"error_message": "API Key가 필요합니다."}` |
| 401 | 잘못된 API Key | `{"error_message": "유효하지 않은 API Key입니다."}` |
| 404 | 회사 없음 | `{"error_message": "존재하지 않는 회사 코드입니다."}` |
| 404 | 검사 없음 | `{"error_message": "존재하지 않는 검사입니다."}` |
| 404 | 응시자 없음 | `{"error_message": "존재하지 않는 응시자입니다."}` |

---

## 프로젝트 목록 조회

회사의 검사 프로젝트 목록을 조회합니다.

```http
POST /api/public/project/list
Content-Type: application/json
X-API-Key: {api_key}
```

**Request Body**

```json
{
  "company_code": "TS001",
  "status": "completed",
  "page": 1,
  "per_page": 20
}
```

**Parameters**

| 필드 | 타입 | 필수 | 설명 |
|-----|-----|-----|------|
| company_code | string | O | 회사 코드 |
| status | string | X | 상태 필터 (pending/active/completed/stopped) |
| page | int | X | 페이지 번호 (기본: 1) |
| per_page | int | X | 페이지당 항목 수 (기본: 20) |

**Response (200)**

```json
{
  "success": true,
  "data": {
    "projects": [
      {
        "idx": "project-uuid",
        "name": "2025년 상반기 인성검사",
        "start_date": "2025-01-15",
        "end_date": "2025-01-20",
        "status": "completed",
        "applicant_count": 50,
        "completed_count": 48
      }
    ],
    "meta": {
      "current_page": 1,
      "last_page": 3,
      "total": 45
    }
  }
}
```

---

## 지원자 목록 조회

특정 검사의 지원자 목록을 조회합니다.

```http
POST /api/public/applicant/list
Content-Type: application/json
X-API-Key: {api_key}
```

**Request Body**

```json
{
  "company_code": "TS001",
  "personality_idx": "project-uuid",
  "is_complete": true,
  "page": 1,
  "per_page": 50
}
```

**Parameters**

| 필드 | 타입 | 필수 | 설명 |
|-----|-----|-----|------|
| company_code | string | O | 회사 코드 |
| personality_idx | string | O | 검사 UUID |
| is_complete | bool | X | 완료 여부 필터 |
| page | int | X | 페이지 번호 |
| per_page | int | X | 페이지당 항목 수 |

**Response (200)**

```json
{
  "success": true,
  "data": {
    "applicants": [
      {
        "idx": "applicant-uuid",
        "name": "홍길동",
        "applicant_number": "A-001",
        "field": "개발",
        "is_complete": true,
        "completed_at": "2025-01-15T11:30:00",
        "avg_score": 85.5,
        "grade": "A"
      }
    ],
    "meta": {
      "current_page": 1,
      "last_page": 2,
      "total": 48
    }
  }
}
```

---

## 보고서 토큰 사용

### 토큰 특성

- 유효 기간: 24시간
- 일회성 사용 가능 (설정에 따라)
- 만료 후 재발급 필요

### 보고서 접근

생성된 URL로 직접 접근:

```
https://report.aprofiler.co.kr/view/{token}
```

### 토큰 검증 (내부 사용)

```php
// ReportAccessToken 모델
$token = ReportAccessToken::verifyToken($tokenString);

if (!$token) {
    // 토큰 없음 또는 만료
}

if ($token->isUsed()) {
    // 이미 사용된 토큰
}

// 사용 처리
$token->markAsUsed();
```

---

## CORS 설정

외부 공개 API는 CORS가 허용됩니다:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type, X-API-Key
```

---

## Rate Limiting

| 제한 | 값 |
|-----|---|
| 분당 요청 수 | 60 |
| 시간당 요청 수 | 1000 |

초과 시 `429 Too Many Requests` 응답

---

## 연동 예시 (cURL)

```bash
# 보고서 URL 생성
curl -X POST https://api.aprofiler.co.kr/api/public/report/url \
  -H 'Content-Type: application/json' \
  -H 'X-API-Key: your-api-key' \
  -d '{
    "company_code": "TS001",
    "personality_idx": "550e8400-e29b-41d4-a716-446655440000",
    "applicant_idx": "applicant-uuid",
    "blind_name": true
  }'

# 프로젝트 목록 조회
curl -X POST https://api.aprofiler.co.kr/api/public/project/list \
  -H 'Content-Type: application/json' \
  -H 'X-API-Key: your-api-key' \
  -d '{
    "company_code": "TS001",
    "status": "completed"
  }'
```
