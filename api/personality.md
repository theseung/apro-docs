# 인성검사 API

## 검사 관리

### 검사 목록 조회

```http
GET /api/personality
Authorization: Bearer {token}
```

**Query Parameters**

| 파라미터 | 타입 | 설명 |
|---------|-----|------|
| page | int | 페이지 번호 (기본: 1) |
| per_page | int | 페이지당 항목 수 (기본: 20) |
| company_id | int | 회사 ID 필터 |
| status | string | 상태 필터 (pending/active/completed/stopped) |
| search | string | 검색어 (검사명) |

**Response (200)**

```json
{
  "success": true,
  "data": [
    {
      "idx": "550e8400-e29b-41d4-a716-446655440000",
      "name": "2025년 상반기 인성검사",
      "company_id": 1,
      "company_name": "테크솔루션",
      "sheet_name": "MBTI 기본형",
      "online": true,
      "start_date": "2025-01-15T09:00:00",
      "end_date": "2025-01-20T18:00:00",
      "is_stop": false,
      "status": "active",
      "applicant_count": 50,
      "completed_count": 30
    }
  ],
  "meta": {
    "current_page": 1,
    "last_page": 5,
    "per_page": 20,
    "total": 100
  }
}
```

---

### 검사 생성

```http
POST /api/personality
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body**

```json
{
  "name": "2025년 상반기 인성검사",
  "company_id": 1,
  "personality_sheet_id": 1,
  "online": true,
  "start_date": "2025-01-15T09:00:00",
  "end_date": "2025-01-20T18:00:00",
  "reliability": 0.8,
  "s_score": 90,
  "a_score": 80,
  "b_score": 70,
  "c_score": 60,
  "welcome_msg1": "환영합니다.",
  "welcome_msg2": "검사 안내입니다.",
  "info_msg": "주의사항 1&&&주의사항 2"
}
```

**Response (201)**

```json
{
  "success": true,
  "message": "검사가 생성되었습니다.",
  "data": {
    "idx": "550e8400-e29b-41d4-a716-446655440000",
    "name": "2025년 상반기 인성검사"
  }
}
```

---

### 검사 상세 조회

```http
GET /api/personality/task/{idx}
Authorization: Bearer {token}
```

**Response (200)**

```json
{
  "success": true,
  "data": {
    "idx": "550e8400-e29b-41d4-a716-446655440000",
    "name": "2025년 상반기 인성검사",
    "company": {
      "id": 1,
      "name": "테크솔루션"
    },
    "sheet": {
      "id": 1,
      "name": "MBTI 기본형",
      "solve_time": 3600
    },
    "online": true,
    "start_date": "2025-01-15T09:00:00",
    "end_date": "2025-01-20T18:00:00",
    "is_stop": false,
    "status": "active",
    "reliability": 0.8,
    "s_score": 90,
    "a_score": 80,
    "b_score": 70,
    "c_score": 60,
    "welcome_msg1": "환영합니다.",
    "welcome_msg2": "검사 안내입니다.",
    "info_msg": "주의사항 1&&&주의사항 2",
    "applicant_count": 50,
    "completed_count": 30
  }
}
```

---

### 검사 수정

```http
PUT /api/personality/task/{idx}
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body**

```json
{
  "name": "수정된 검사명",
  "end_date": "2025-01-25T18:00:00"
}
```

---

### 검사 삭제

```http
DELETE /api/personality/task/{idx}
Authorization: Bearer {token}
```

---

### 검사 상태 토글

```http
PATCH /api/personality/task/{idx}/toggle-status
Authorization: Bearer {token}
```

**Response (200)**

```json
{
  "success": true,
  "message": "검사가 중지되었습니다.",
  "data": {
    "is_stop": true
  }
}
```

---

## 응시자 관리

### 응시자 목록

```http
GET /api/personality/task/{idx}/applicants
Authorization: Bearer {token}
```

**Query Parameters**

| 파라미터 | 타입 | 설명 |
|---------|-----|------|
| page | int | 페이지 번호 |
| per_page | int | 페이지당 항목 수 |
| is_complete | bool | 완료 여부 필터 |
| search | string | 검색어 (이름, 번호) |

**Response (200)**

```json
{
  "success": true,
  "data": [
    {
      "idx": "applicant-uuid",
      "name": "홍길동",
      "applicant_number": "A-001",
      "id": "applicant01",
      "email": "hong@example.com",
      "phone": "010-1234-5678",
      "field": "개발",
      "is_complete": true,
      "start_date": "2025-01-15T10:00:00",
      "end_date": "2025-01-15T11:30:00",
      "avg_score": 85.5,
      "grade": "A"
    }
  ]
}
```

---

### 응시자 대량 등록

```http
POST /api/personality/task/{idx}/applicants/bulk
Authorization: Bearer {token}
Content-Type: multipart/form-data
```

**Request Body**

```
file: (Excel 파일)
send_email: true  (선택)
```

**Excel 형식**

| 이름 | 수험번호 | 이메일 | 전화번호 | 분야 |
|-----|---------|-------|---------|-----|
| 홍길동 | A-001 | hong@example.com | 010-1234-5678 | 개발 |
| 김철수 | A-002 | kim@example.com | 010-2345-6789 | 마케팅 |

**Response (201)**

```json
{
  "success": true,
  "message": "50명의 응시자가 등록되었습니다.",
  "data": {
    "total": 50,
    "success": 48,
    "failed": 2,
    "errors": [
      {"row": 5, "message": "이메일 형식이 올바르지 않습니다."},
      {"row": 12, "message": "중복된 수험번호입니다."}
    ]
  }
}
```

---

### 응시자 채점 초기화

```http
POST /api/personality/task/{idx}/applicants/{applicantIdx}/reset
Authorization: Bearer {token}
```

---

## 검사지 관리

### 검사지 목록

```http
GET /api/personality/sheet
Authorization: Bearer {token}
```

---

### 검사지 생성

```http
POST /api/personality/sheet
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body**

```json
{
  "name": "MBTI 기본형",
  "type": "MBTI",
  "company_id": 1,
  "solve_time": 3600,
  "answerType": "likert5",
  "code": ["EI", "SN", "TF", "JP"],
  "problem": [1, 2, 3, 4, 5],
  "settlement": {
    "EI": {"formula": "sum", "threshold": 50}
  }
}
```

---

### 검사지 복제

```http
POST /api/personality/sheet/{id}/copy
Authorization: Bearer {token}
```

---

## 다운로드

### 응시자 Excel 다운로드

```http
POST /api/personality/task/{idx}/download-excel
Authorization: Bearer {token}
```

### 답안 다운로드

```http
GET /api/personality/task/{idx}/download-answer
Authorization: Bearer {token}
```
