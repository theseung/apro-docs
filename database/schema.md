# 데이터베이스 스키마

## 개요

- **개발 환경**: SQLite
- **운영 환경**: MySQL 또는 PostgreSQL
- **Primary Key**: 대부분 UUID (`idx` 필드)
- **마이그레이션**: Laravel Migration 사용

## 테이블 목록 (30개)

### 핵심 시스템

| 테이블 | 설명 |
|-------|------|
| companies | 기업/회사 정보 |
| users | 관리자 사용자 |
| personal_access_tokens | Sanctum 인증 토큰 |
| notices | 공지사항 |
| cache | Laravel 캐시 |
| sessions | Laravel 세션 |

### 인성검사 (Personality)

| 테이블 | 설명 |
|-------|------|
| personality_test_types | 검사 종류 (MBTI, Big5 등) |
| personality_codes | 검사 코드 |
| personality_questions | 검사 문항 |
| personality_sheets | 검사지 템플릿 |
| personalities | 인성검사 (UUID) |
| personality_applicants | 응시자 (UUID) |
| personality_data_statistics | 통계 데이터 |
| personality_grade_criteria | 등급 기준 |

### 평판진단 (Reputation)

| 테이블 | 설명 |
|-------|------|
| reputation_test_types | 진단 종류 |
| reputation_codes | 진단 코드 |
| reputation_questions | 진단 문항 |
| reputation_sheets | 진단지 템플릿 |
| reputation_tasks | 평판진단 작업 (UUID) |
| reputation_applicants | 피평가자 (UUID) |
| reputation_raters | 평가자 (UUID) |
| reputation_data | 평가 데이터 |
| reputation_data_statistics | 통계 |

### 다면평가 (Multifaceted)

| 테이블 | 설명 |
|-------|------|
| multifaceted_test_types | 평가 종류 |
| multifaceted_codes | 평가 코드 |
| multifaceted_questions | 평가 문항 |
| multifaceted_sheets | 평가지 템플릿 |
| multifaceted_tasks | 다면평가 작업 (UUID) |
| multifaceted_applicants | 응시자 (UUID) |
| multifaceted_raters | 평가자 (UUID) |

### 기타

| 테이블 | 설명 |
|-------|------|
| report_access_tokens | 보고서 접근 토큰 |

---

## 주요 테이블 스키마

### companies

```sql
CREATE TABLE companies (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(100),
    code VARCHAR(50) UNIQUE,
    logo VARCHAR(500),
    talent TEXT,
    agency_id BIGINT NULL,              -- 대행사 (self-reference)
    agency_date DATE NULL,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (agency_id) REFERENCES companies(id)
);
```

### users

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    phone VARCHAR(50),
    position VARCHAR(100),
    company_id BIGINT NOT NULL,
    role VARCHAR(50) DEFAULT 'user',
    level INT DEFAULT 1,                -- 1: 일반, 2: 팀장, 3: 관리자
    remember_token VARCHAR(100),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

### personalities

```sql
CREATE TABLE personalities (
    idx CHAR(36) PRIMARY KEY,           -- UUID
    name VARCHAR(255) NOT NULL,
    online BOOLEAN DEFAULT true,
    company_id BIGINT NOT NULL,
    personality_sheet_id BIGINT NOT NULL,
    start_date DATETIME,
    end_date DATETIME,
    is_stop BOOLEAN DEFAULT false,
    reliability DECIMAL(5,2),
    s_score DECIMAL(5,2),               -- S등급 기준
    a_score DECIMAL(5,2),               -- A등급 기준
    b_score DECIMAL(5,2),               -- B등급 기준
    c_score DECIMAL(5,2),               -- C등급 기준
    welcome_msg1 TEXT,
    welcome_msg2 TEXT,
    info_msg TEXT,                      -- &&& 구분자로 다중 메시지
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (company_id) REFERENCES companies(id),
    FOREIGN KEY (personality_sheet_id) REFERENCES personality_sheets(id)
);
```

### personality_applicants

```sql
CREATE TABLE personality_applicants (
    idx CHAR(36) PRIMARY KEY,           -- UUID
    personality_idx CHAR(36) NOT NULL,
    company_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    applicant_number VARCHAR(100),
    id VARCHAR(100) NOT NULL,           -- 로그인 ID
    pw VARCHAR(100) NOT NULL,           -- 로그인 PW
    email VARCHAR(255),
    phone VARCHAR(50),
    field VARCHAR(100),                 -- 분야/직군
    additional_id VARCHAR(100),         -- 추가 식별자
    answer JSON,                        -- 응답 데이터
    scoring_result JSON,                -- 채점 결과
    report_value JSON,                  -- 보고서 데이터
    is_complete BOOLEAN DEFAULT false,
    start_date DATETIME,
    end_date DATETIME,
    avg_score DECIMAL(5,2),
    reliability_score DECIMAL(5,2),
    statistics_reflected BOOLEAN DEFAULT false,
    is_scoring_started BOOLEAN DEFAULT false,
    is_scoring_completed BOOLEAN DEFAULT false,
    login_count INT DEFAULT 0,          -- 최대 2회
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (personality_idx) REFERENCES personalities(idx),
    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

### personality_sheets

```sql
CREATE TABLE personality_sheets (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(100),
    company_id BIGINT NOT NULL,
    solve_time INT,                     -- 소요 시간 (초)
    answerType VARCHAR(50),
    display_codes JSON,                 -- 표시할 코드들
    code JSON,                          -- 코드 배열
    problem JSON,                       -- 문항 배열
    settlement JSON,                    -- 정산식
    problem_order JSON,                 -- 문항 순서
    active BOOLEAN DEFAULT false,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

### reputation_tasks

```sql
CREATE TABLE reputation_tasks (
    idx CHAR(36) PRIMARY KEY,           -- UUID
    name VARCHAR(255) NOT NULL,
    company_id BIGINT NOT NULL,
    sheet_id BIGINT NOT NULL,
    rater_min INT DEFAULT 3,            -- 최소 평가자 수
    rater_max INT DEFAULT 10,           -- 최대 평가자 수
    start_date DATETIME,
    end_date DATETIME,
    is_stop BOOLEAN DEFAULT false,
    welcome_msg1 TEXT,
    welcome_msg2 TEXT,
    applicant_welcome_msg1 TEXT,
    applicant_welcome_msg2 TEXT,
    applicant_info_msg TEXT,
    rater_welcome_msg1 TEXT,
    rater_welcome_msg2 TEXT,
    rater_info_msg TEXT,
    info_msg TEXT,
    guide_msg TEXT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (company_id) REFERENCES companies(id),
    FOREIGN KEY (sheet_id) REFERENCES reputation_sheets(id)
);
```

### reputation_raters

```sql
CREATE TABLE reputation_raters (
    idx CHAR(36) PRIMARY KEY,           -- UUID
    task_idx CHAR(36) NOT NULL,
    applicant_idx CHAR(36) NOT NULL,
    company_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    company VARCHAR(255),               -- 평가자 소속 회사
    position VARCHAR(100),              -- 직책
    relationship VARCHAR(50),           -- 관계 (상사/동료/부하)
    exchange BOOLEAN DEFAULT false,     -- 상호 평가 여부
    is_complete BOOLEAN DEFAULT false,
    answer JSON,
    complete_date DATETIME,
    email_sent_at DATETIME,
    scoring_record JSON,
    statistics_updated BOOLEAN DEFAULT false,
    statistics_updated_at DATETIME,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (task_idx) REFERENCES reputation_tasks(idx),
    FOREIGN KEY (applicant_idx) REFERENCES reputation_applicants(idx),
    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

### report_access_tokens

```sql
CREATE TABLE report_access_tokens (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    token CHAR(64) UNIQUE NOT NULL,     -- 랜덤 토큰
    company_id BIGINT NOT NULL,
    personality_idx CHAR(36) NOT NULL,
    applicant_idx CHAR(36) NOT NULL,
    blind_name_fg BOOLEAN DEFAULT false,
    blind_field_fg BOOLEAN DEFAULT false,
    blind_identifier_fg BOOLEAN DEFAULT false,
    blind_exam_number_fg BOOLEAN DEFAULT false,
    population_source_fg BOOLEAN DEFAULT false,
    field_rank_fg BOOLEAN DEFAULT false,
    expires_at DATETIME NOT NULL,       -- 만료 시간
    used_at DATETIME,                   -- 사용 시간
    created_at TIMESTAMP,
    updated_at TIMESTAMP,

    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

---

## 인덱스

### 성능 최적화 인덱스

```sql
-- 회사별 조회
CREATE INDEX idx_personalities_company ON personalities(company_id);
CREATE INDEX idx_personality_applicants_company ON personality_applicants(company_id);
CREATE INDEX idx_reputation_tasks_company ON reputation_tasks(company_id);

-- 검사/작업별 조회
CREATE INDEX idx_personality_applicants_personality ON personality_applicants(personality_idx);
CREATE INDEX idx_reputation_raters_task ON reputation_raters(task_idx);
CREATE INDEX idx_reputation_raters_applicant ON reputation_raters(applicant_idx);

-- 상태 조회
CREATE INDEX idx_personality_applicants_complete ON personality_applicants(is_complete);
CREATE INDEX idx_reputation_raters_complete ON reputation_raters(is_complete);

-- 토큰 검증
CREATE UNIQUE INDEX idx_report_access_tokens_token ON report_access_tokens(token);
CREATE INDEX idx_report_access_tokens_expires ON report_access_tokens(expires_at);
```
