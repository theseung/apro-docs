# 응시자 API

## 검사 진행

### 검사 시작

응시 시작 시간 기록

```http
POST /api/applicant/start
Authorization: Bearer {applicant_token}
```

**Response (200)**

```json
{
  "success": true,
  "message": "검사가 시작되었습니다.",
  "data": {
    "start_date": "2025-01-15T10:00:00",
    "end_time": "2025-01-15T11:00:00",
    "total_questions": 100
  }
}
```

---

### 문항 조회 (일괄)

10개씩 문항 조회

```http
GET /api/applicant/questions-bulk
Authorization: Bearer {applicant_token}
```

**Query Parameters**

| 파라미터 | 타입 | 설명 |
|---------|-----|------|
| start | int | 시작 인덱스 (기본: 0) |
| count | int | 조회 개수 (기본: 10, 최대: 20) |

**Response (200)**

```json
{
  "success": true,
  "data": {
    "questions": [
      {
        "id": 1,
        "question_number": "EI01",
        "content": "사람들과 함께 있을 때 에너지를 얻는다.",
        "code": "EI",
        "direction": "positive"
      },
      {
        "id": 2,
        "question_number": "EI02",
        "content": "혼자 있을 때 더 편안함을 느낀다.",
        "code": "EI",
        "direction": "negative"
      }
    ],
    "has_more": true,
    "total": 100,
    "current": 10
  }
}
```

---

### 답변 저장

```http
POST /api/applicant/answer
Authorization: Bearer {applicant_token}
Content-Type: application/json
```

**Request Body**

```json
{
  "question_id": 1,
  "value": 4
}
```

**Response (200)**

```json
{
  "success": true,
  "data": {
    "answered_count": 45,
    "total_questions": 100
  }
}
```

---

### 임시 저장

```http
POST /api/applicant/save-temp
Authorization: Bearer {applicant_token}
Content-Type: application/json
```

**Request Body**

```json
{
  "answers": {
    "1": 4,
    "2": 3,
    "3": 5
  }
}
```

---

### 서술형 답변 제출

```http
POST /api/applicant/submit-descriptive
Authorization: Bearer {applicant_token}
Content-Type: application/json
```

**Request Body**

```json
{
  "question_id": 101,
  "answer": "서술형 답변 내용입니다..."
}
```

---

### 검사 종료

```http
POST /api/applicant/end
Authorization: Bearer {applicant_token}
```

**Response (200)**

```json
{
  "success": true,
  "message": "검사가 완료되었습니다.",
  "data": {
    "is_complete": true,
    "end_date": "2025-01-15T10:45:00"
  }
}
```

---

## 평가자 관리 (다면평가/평판진단)

### 평가자 추가

```http
POST /api/applicant/{type}-rater
Authorization: Bearer {applicant_token}
Content-Type: application/json
```

**URL Parameters**

- `type`: `reputation` 또는 `multifaceted`

**Request Body**

```json
{
  "rater_name": "김평가",
  "rater_email": "rater@example.com",
  "rater_company": "테크솔루션",
  "rater_position": "팀장",
  "rater_relationship": "상사",
  "rater_exchange": false
}
```

**Response (201)**

```json
{
  "success": true,
  "message": "평가자가 추가되었습니다.",
  "data": {
    "idx": "rater-uuid",
    "name": "김평가",
    "email": "rater@example.com",
    "is_complete": false
  }
}
```

---

### 평가자 목록 조회

```http
POST /api/applicant/{type}-rater/list
Authorization: Bearer {applicant_token}
```

**Response (200)**

```json
{
  "success": true,
  "data": [
    {
      "idx": "rater-uuid-1",
      "name": "김평가",
      "email": "rater1@example.com",
      "relationship": "상사",
      "is_complete": true,
      "email_sent_at": "2025-01-10T09:00:00"
    },
    {
      "idx": "rater-uuid-2",
      "name": "이동료",
      "email": "rater2@example.com",
      "relationship": "동료",
      "is_complete": false,
      "email_sent_at": null
    }
  ]
}
```

---

### 평가자 수정

```http
PUT /api/applicant/{type}-rater/{raterIdx}
Authorization: Bearer {applicant_token}
Content-Type: application/json
```

**Request Body**

```json
{
  "rater_name": "수정된 이름",
  "rater_email": "new-email@example.com"
}
```

---

### 평가자 삭제

```http
DELETE /api/applicant/{type}-rater/{raterIdx}
Authorization: Bearer {applicant_token}
```

---

### 평가 요청 이메일 발송

```http
POST /api/applicant/{type}-rater/{raterIdx}/send-email
Authorization: Bearer {applicant_token}
```

**Response (200)**

```json
{
  "success": true,
  "message": "이메일이 발송되었습니다.",
  "data": {
    "email_sent_at": "2025-01-15T10:00:00"
  }
}
```

**Error Response (429)**

```json
{
  "success": false,
  "error_message": "5분 이내에 재발송할 수 없습니다."
}
```

---

## 에러 응답

### 공통 에러

| 코드 | 상황 |
|-----|-----|
| 401 | 토큰 없음 또는 만료 |
| 403 | 검사 기간이 아님 |
| 404 | 리소스 없음 |
| 422 | 유효성 검사 실패 |

### 검사 관련 에러

| 에러 메시지 | 상황 |
|-----------|-----|
| "검사 기간이 종료되었습니다." | end_date 초과 |
| "검사가 시작되지 않았습니다." | start_date 이전 |
| "이미 완료된 검사입니다." | is_complete = true |
| "로그인 횟수를 초과했습니다." | login_count >= 2 |
