# 인증 API

## 관리자 인증

### 로그인

관리자 로그인 및 토큰 발급

```http
POST /api/auth/login
Content-Type: application/json
```

**Request Body**

```json
{
  "email": "admin@example.com",
  "password": "password"
}
```

**Response (200)**

```json
{
  "success": true,
  "message": "로그인이 완료되었습니다.",
  "data": {
    "user": {
      "id": 1,
      "name": "관리자",
      "email": "admin@example.com",
      "company_id": 1,
      "role": "admin",
      "level": 3
    },
    "access_token": "1|abc123..."
  }
}
```

**Error Response (401)**

```json
{
  "success": false,
  "error_message": "이메일 또는 비밀번호가 일치하지 않습니다."
}
```

---

### 토큰 갱신

```http
POST /api/auth/refresh-token
Authorization: Bearer {access_token}
```

**Response (200)**

```json
{
  "success": true,
  "data": {
    "access_token": "2|xyz789..."
  }
}
```

---

### 로그아웃

```http
POST /api/auth/logout
Authorization: Bearer {access_token}
```

**Response (200)**

```json
{
  "success": true,
  "message": "로그아웃이 완료되었습니다."
}
```

---

### 비밀번호 초기화 요청

```http
POST /api/auth/forgot-password
Content-Type: application/json
```

**Request Body**

```json
{
  "email": "admin@example.com"
}
```

**Response (200)**

```json
{
  "success": true,
  "message": "비밀번호 재설정 링크가 이메일로 발송되었습니다."
}
```

---

## 응시자 인증

### 응시자 로그인

3가지 타입(Personality, Reputation, Multifaceted) 응시자 통합 로그인

```http
POST /api/applicant/login
Content-Type: application/json
company: {회사코드}
```

**Request Body**

```json
{
  "id": "applicant01",
  "pw": "password123",
  "device_name": "web"
}
```

**Response (200)**

```json
{
  "success": true,
  "message": "로그인이 완료되었습니다.",
  "data": {
    "applicant": {
      "idx": "550e8400-e29b-41d4-a716-446655440000",
      "name": "홍길동",
      "applicant_number": "A-001",
      "id": "applicant01",
      "email": "hong@example.com",
      "phone": "010-1234-5678",
      "field": "개발",
      "is_complete": false,
      "start_date": null,
      "end_date": null
    },
    "access_token": "3|token...",
    "type": "personality"
  }
}
```

**Error Responses**

| 코드 | 상황 | 응답 |
|-----|-----|-----|
| 400 | company 헤더 누락 | `{"error_message": "회사 코드가 필요합니다."}` |
| 401 | 잘못된 자격증명 | `{"error_message": "아이디 또는 비밀번호가 일치하지 않습니다."}` |
| 401 | 로그인 횟수 초과 | `{"error_message": "로그인 횟수를 초과했습니다. (최대 2회)"}` |
| 403 | 검사 기간 외 | `{"error_message": "검사 기간이 아닙니다."}` |
| 404 | 회사 코드 없음 | `{"error_message": "존재하지 않는 회사 코드입니다."}` |

---

### 응시자 정보 조회

```http
GET /api/applicant/info
Authorization: Bearer {access_token}
```

**Response (200)**

```json
{
  "success": true,
  "data": {
    "idx": "550e8400-e29b-41d4-a716-446655440000",
    "name": "홍길동",
    "applicant_number": "A-001",
    "is_complete": false,
    "start_date": "2025-01-15T09:00:00",
    "total_questions": 100,
    "answered_count": 45,
    "remaining_time": 3600
  }
}
```

---

### 응시자 로그아웃

```http
POST /api/applicant/logout
Authorization: Bearer {access_token}
```

**Response (200)**

```json
{
  "success": true,
  "message": "로그아웃이 완료되었습니다."
}
```

---

## 외부 API 인증

### API Key 인증

외부 시스템 연동용 API Key 방식

```http
POST /api/public/report/url
Content-Type: application/json
X-API-Key: {api_key}
```

또는 쿼리 파라미터:

```http
POST /api/public/report/url?api_key={api_key}
```

**Error Response (401)**

```json
{
  "success": false,
  "error_message": "유효하지 않은 API Key입니다."
}
```

---

## 인증 헤더 요약

| 엔드포인트 유형 | 헤더 |
|---------------|------|
| 관리자 API | `Authorization: Bearer {sanctum_token}` |
| 응시자 API | `Authorization: Bearer {applicant_token}` |
| 응시자 로그인 | `company: {회사코드}` |
| 외부 공개 API | `X-API-Key: {api_key}` |
| 평가자 API | 인증 없음 (UUID로 직접 접근) |
