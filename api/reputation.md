# 평판진단 API

## 작업 관리

### 작업 목록

```http
GET /api/reputation/task
Authorization: Bearer {token}
```

### 작업 생성

```http
POST /api/reputation/task
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body**

```json
{
  "name": "2025년 평판진단",
  "company_id": 1,
  "sheet_id": 1,
  "rater_min": 3,
  "rater_max": 10,
  "start_date": "2025-01-15T09:00:00",
  "end_date": "2025-01-30T18:00:00",
  "welcome_msg1": "환영합니다.",
  "welcome_msg2": "평가 안내입니다.",
  "rater_welcome_msg1": "평가자님 환영합니다.",
  "info_msg": "안내사항"
}
```

### 작업 상세

```http
GET /api/reputation/task/{idx}
Authorization: Bearer {token}
```

### 작업 수정

```http
PUT /api/reputation/task/{idx}
Authorization: Bearer {token}
```

### 작업 삭제

```http
DELETE /api/reputation/task/{idx}
Authorization: Bearer {token}
```

---

## 피평가자 관리

### 피평가자 목록

```http
GET /api/reputation/task/{idx}/applicants
Authorization: Bearer {token}
```

**Response**

```json
{
  "success": true,
  "data": [
    {
      "idx": "applicant-uuid",
      "name": "홍길동",
      "applicant_number": "A-001",
      "email": "hong@example.com",
      "is_complete": false,
      "raters": [
        {
          "idx": "rater-uuid",
          "name": "김평가",
          "relationship": "상사",
          "is_complete": true
        }
      ],
      "rater_count": 5,
      "completed_rater_count": 3
    }
  ]
}
```

### 피평가자 추가

```http
POST /api/reputation/task/{idx}/applicants
Authorization: Bearer {token}
```

### 피평가자 대량 등록

```http
POST /api/reputation/task/{idx}/applicants/bulk
Authorization: Bearer {token}
Content-Type: multipart/form-data
```

---

## 평가자 관리 (관리자)

### 평가자 추가

```http
POST /api/reputation/task/{idx}/applicants/{applicantIdx}/rater
Authorization: Bearer {token}
```

**Request Body**

```json
{
  "name": "김평가",
  "email": "rater@example.com",
  "company": "테크솔루션",
  "position": "팀장",
  "relationship": "상사",
  "exchange": false
}
```

### 평가자 이메일 발송

```http
POST /api/reputation/task/{idx}/applicants/{applicantIdx}/rater/{raterIdx}/send-email
Authorization: Bearer {token}
```

### 미완료 평가자 일괄 발송

```http
POST /api/reputation/task/{idx}/applicants/{applicantIdx}/rater/send-all
Authorization: Bearer {token}
```

---

## 평가자 평가 진행 (비인증)

평가자는 이메일 링크로 접근하며, 별도 인증 없이 UUID로 직접 접근합니다.

### 평가자 정보 조회

```http
GET /api/rater/{rater_idx}/info
```

**Response**

```json
{
  "success": true,
  "data": {
    "idx": "rater-uuid",
    "name": "김평가",
    "applicant_name": "홍길동",
    "task_name": "2025년 평판진단",
    "relationship": "상사",
    "is_complete": false,
    "questions_count": 50
  }
}
```

### Likert 척도 문항 조회

```http
GET /api/rater/{rater_idx}/questions/likert
```

### 서술형 문항 조회

```http
GET /api/rater/{rater_idx}/questions/descriptive
```

### 객관식 답변 저장

```http
POST /api/rater/{rater_idx}/answers
Content-Type: application/json
```

**Request Body**

```json
{
  "answers": {
    "1": 4,
    "2": 5,
    "3": 3
  }
}
```

### 서술형 답변 저장

```http
POST /api/rater/{rater_idx}/descriptive-answers
Content-Type: application/json
```

**Request Body**

```json
{
  "question_id": 101,
  "answer": "서술형 답변 내용..."
}
```

### 평가 완료

```http
GET /api/rater/{rater_idx}/end
```

**Response**

```json
{
  "success": true,
  "message": "평가가 완료되었습니다.",
  "data": {
    "is_complete": true,
    "complete_date": "2025-01-15T14:30:00"
  }
}
```

---

## 다운로드

### 결과 Excel 다운로드

```http
POST /api/reputation/task/{idx}/download-excel
Authorization: Bearer {token}
```

### 답안 다운로드

```http
GET /api/reputation/task/{idx}/download-answer
Authorization: Bearer {token}
```

### PDF 보고서 다운로드

```http
POST /api/reputation/task/{idx}/download-pdf
Authorization: Bearer {token}
```
