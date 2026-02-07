# 다면평가 API

## 개요

다면평가 API는 평판진단과 유사한 구조이나, **자기평가(본인 응답)** 기능이 추가됩니다.

## 주요 차이점

| 항목 | 평판진단 | 다면평가 |
|-----|---------|---------|
| 자기평가 | X | O (participant_response 설정) |
| 응시자 응답 | 평가자만 응답 | 응시자 + 평가자 모두 응답 |
| 분석 | 타인 평가만 | 자기평가 vs 타인평가 비교 |

---

## 작업 관리

### 작업 목록

```http
GET /api/multifaceted/task
Authorization: Bearer {token}
```

### 작업 생성

```http
POST /api/multifaceted/task
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body**

```json
{
  "name": "2025년 다면평가",
  "company_id": 1,
  "sheet_id": 1,
  "rater_min": 3,
  "rater_max": 10,
  "participant_response": true,
  "start_date": "2025-01-15T09:00:00",
  "end_date": "2025-01-30T18:00:00",
  "applicant_welcome_msg1": "응시자님 환영합니다.",
  "applicant_welcome_msg2": "자기평가 안내입니다.",
  "applicant_info_msg": "자기평가 진행 안내",
  "rater_welcome_msg1": "평가자님 환영합니다.",
  "rater_welcome_msg2": "타인평가 안내입니다.",
  "rater_info_msg": "타인평가 진행 안내"
}
```

**participant_response 옵션**

- `true`: 응시자도 자기평가 문항에 응답
- `false`: 평가자만 응답 (평판진단과 동일)

### 작업 상세/수정/삭제

평판진단 API와 동일한 구조

---

## 응시자 자기평가 (participant_response = true)

### 자기평가 랜딩

```http
GET /api/applicant/info
Authorization: Bearer {applicant_token}
```

**Response (다면평가 타입)**

```json
{
  "success": true,
  "data": {
    "idx": "applicant-uuid",
    "name": "홍길동",
    "type": "multifaceted",
    "task": {
      "idx": "task-uuid",
      "name": "2025년 다면평가",
      "participant_response": true
    },
    "self_assessment_complete": false,
    "raters": [
      {"idx": "rater-1", "name": "김평가", "is_complete": true},
      {"idx": "rater-2", "name": "이동료", "is_complete": false}
    ],
    "rater_count": 5,
    "completed_rater_count": 3
  }
}
```

### 자기평가 문항 조회

```http
GET /api/applicant/questions-bulk?type=self
Authorization: Bearer {applicant_token}
```

### 자기평가 답변 저장

```http
POST /api/applicant/answer
Authorization: Bearer {applicant_token}
Content-Type: application/json
```

**Request Body**

```json
{
  "question_id": 1,
  "value": 4,
  "type": "self"
}
```

### 자기평가 완료

```http
POST /api/applicant/end
Authorization: Bearer {applicant_token}
```

---

## 평가자 관리

평판진단 API와 동일

### 평가자 추가

```http
POST /api/applicant/multifaceted-rater
Authorization: Bearer {applicant_token}
```

### 평가자 목록

```http
POST /api/applicant/multifaceted-rater/list
Authorization: Bearer {applicant_token}
```

### 이메일 발송

```http
POST /api/applicant/multifaceted-rater/{raterIdx}/send-email
Authorization: Bearer {applicant_token}
```

---

## 평가자 평가 진행

평판진단과 동일한 구조 (비인증, UUID 직접 접근)

```http
GET /api/rater/{rater_idx}/info
GET /api/rater/{rater_idx}/questions/likert
POST /api/rater/{rater_idx}/answers
GET /api/rater/{rater_idx}/end
```

---

## 결과 분석

### 자기평가 vs 타인평가 비교

다면평가 결과에는 자기평가와 타인평가 비교 데이터가 포함됩니다.

```json
{
  "applicant": {
    "name": "홍길동",
    "self_score": {
      "leadership": 4.2,
      "communication": 3.8,
      "teamwork": 4.0
    },
    "others_score": {
      "leadership": 3.5,
      "communication": 4.1,
      "teamwork": 3.9
    },
    "gap_analysis": {
      "leadership": 0.7,
      "communication": -0.3,
      "teamwork": 0.1
    }
  }
}
```

---

## 다운로드

### Excel 다운로드

```http
POST /api/multifaceted/task/{idx}/download-excel
Authorization: Bearer {token}
```

### 비교 분석 보고서

```http
POST /api/multifaceted/task/{idx}/download-comparison-report
Authorization: Bearer {token}
```

**Request Body (선택)**

```json
{
  "applicant_idx": "specific-applicant-uuid",
  "format": "pdf"
}
```
