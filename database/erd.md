# ERD (Entity Relationship Diagram)

## 전체 관계도

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              COMPANIES (중심)                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ 1:N (company_id)
        ┌─────────────────────────────┼─────────────────────────────┐
        │                             │                             │
        ▼                             ▼                             ▼
┌───────────────┐          ┌─────────────────┐          ┌─────────────────┐
│    users      │          │ personality_    │          │ reputation_     │
│               │          │    sheets       │          │    sheets       │
└───────────────┘          └────────┬────────┘          └────────┬────────┘
                                    │                            │
                                    │ 1:N                        │ 1:N
                                    ▼                            ▼
                           ┌─────────────────┐          ┌─────────────────┐
                           │  personalities  │          │ reputation_     │
                           │     (UUID)      │          │    tasks        │
                           └────────┬────────┘          │    (UUID)       │
                                    │                   └────────┬────────┘
                                    │ 1:N                        │
                                    ▼                    ┌───────┴───────┐
                           ┌─────────────────┐           │               │
                           │  personality_   │           ▼               ▼
                           │  applicants     │   ┌─────────────┐ ┌─────────────┐
                           │    (UUID)       │   │ reputation_ │ │ reputation_ │
                           └─────────────────┘   │ applicants  │ │   raters    │
                                                 │   (UUID)    │ │   (UUID)    │
                                                 └──────┬──────┘ └─────────────┘
                                                        │              ▲
                                                        │ 1:N          │
                                                        └──────────────┘
```

## 인성검사 서브시스템

```
┌───────────────────────────────────────────────────────────────────┐
│                    PERSONALITY TEST SYSTEM                        │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────────┐                                          │
│  │personality_test_types│◄──────┐                                 │
│  │  (id, name, etc.)   │       │                                  │
│  └─────────────────────┘       │ test_type                        │
│                                │                                  │
│  ┌─────────────────────┐       │                                  │
│  │ personality_codes   │───────┘                                  │
│  │  (id, code, name)   │◄──────┐                                  │
│  └─────────────────────┘       │ code                             │
│                                │                                  │
│  ┌─────────────────────┐       │                                  │
│  │personality_questions│───────┘                                  │
│  │  (id, content, etc.)│                                          │
│  └─────────────────────┘                                          │
│           │                                                       │
│           │ JSON 참조 (problem 필드)                               │
│           ▼                                                       │
│  ┌─────────────────────┐                                          │
│  │ personality_sheets  │ ◄─────── company_id ───────┐             │
│  │  (id, code:JSON)    │                            │             │
│  └──────────┬──────────┘                            │             │
│             │                                       │             │
│             │ personality_sheet_id                  │             │
│             ▼                                       │             │
│  ┌─────────────────────┐                   ┌────────┴────────┐    │
│  │   personalities     │                   │    companies    │    │
│  │  (idx:UUID, name)   │◄── company_id ────│                 │    │
│  └──────────┬──────────┘                   └─────────────────┘    │
│             │                                                     │
│             │ personality_idx                                     │
│             ▼                                                     │
│  ┌─────────────────────┐                                          │
│  │personality_applicants│                                         │
│  │  (idx:UUID, answer)  │                                         │
│  │  JSON 필드:           │                                         │
│  │  - answer            │                                         │
│  │  - scoring_result    │                                         │
│  │  - report_value      │                                         │
│  └─────────────────────┘                                          │
│                                                                   │
│  ┌─────────────────────────┐                                      │
│  │personality_data_        │                                      │
│  │statistics               │                                      │
│  │  (id, code, average)    │                                      │
│  └─────────────────────────┘                                      │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

## 평판진단/다면평가 서브시스템

```
┌───────────────────────────────────────────────────────────────────┐
│                 REPUTATION / MULTIFACETED SYSTEM                  │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────┐    ┌─────────────────┐                       │
│  │ reputation_     │    │ multifaceted_   │                       │
│  │ test_types      │    │ test_types      │                       │
│  └────────┬────────┘    └────────┬────────┘                       │
│           │                      │                                │
│           ▼                      ▼                                │
│  ┌─────────────────┐    ┌─────────────────┐                       │
│  │ reputation_codes│    │ multifaceted_   │                       │
│  └────────┬────────┘    │ codes           │                       │
│           │             └────────┬────────┘                       │
│           ▼                      ▼                                │
│  ┌─────────────────┐    ┌─────────────────┐                       │
│  │ reputation_     │    │ multifaceted_   │                       │
│  │ questions       │    │ questions       │                       │
│  └────────┬────────┘    └────────┬────────┘                       │
│           │                      │                                │
│           ▼                      ▼                                │
│  ┌─────────────────┐    ┌─────────────────┐                       │
│  │ reputation_     │    │ multifaceted_   │                       │
│  │ sheets          │    │ sheets          │                       │
│  └────────┬────────┘    └────────┬────────┘                       │
│           │ sheet_id             │ sheet_id                       │
│           ▼                      ▼                                │
│  ┌─────────────────┐    ┌─────────────────┐                       │
│  │ reputation_tasks│    │ multifaceted_   │                       │
│  │   (idx:UUID)    │    │ tasks (idx:UUID)│                       │
│  └────────┬────────┘    └────────┬────────┘                       │
│           │                      │                                │
│     ┌─────┴─────┐          ┌─────┴─────┐                          │
│     │           │          │           │                          │
│     ▼           ▼          ▼           ▼                          │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐                       │
│ │ repu-  │ │ repu-  │ │ multi- │ │ multi- │                       │
│ │ tation_│ │ tation_│ │ faceted│ │ faceted│                       │
│ │ appli- │ │ raters │ │ _appli-│ │ _raters│                       │
│ │ cants  │ │        │ │ cants  │ │        │                       │
│ │ (UUID) │ │ (UUID) │ │ (UUID) │ │ (UUID) │                       │
│ └───┬────┘ └────────┘ └───┬────┘ └────────┘                       │
│     │          ▲          │          ▲                            │
│     │          │          │          │                            │
│     └──────────┘          └──────────┘                            │
│     applicant_idx         applicant_idx                           │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

## 대행사 구조 (Self-Reference)

```
┌─────────────────────────────────────────────────────────────────┐
│                     AGENCY HIERARCHY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐                                            │
│  │  Company A      │  agency_id = NULL (최상위)                  │
│  │  (id=1)         │                                            │
│  └────────┬────────┘                                            │
│           │                                                     │
│           │ agency_id = 1                                       │
│           ├────────────────┐                                    │
│           │                │                                    │
│           ▼                ▼                                    │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │  Company B  │  │  Company C  │                               │
│  │  (id=2)     │  │  (id=3)     │                               │
│  └─────────────┘  └──────┬──────┘                               │
│                          │                                      │
│                          │ agency_id = 3                        │
│                          ▼                                      │
│                  ┌─────────────┐                                │
│                  │  Company D  │                                │
│                  │  (id=4)     │                                │
│                  └─────────────┘                                │
│                                                                 │
│  접근 권한:                                                      │
│  - A 사용자: A, B, C 접근 가능 (D는 직접 대행 아님)               │
│  - B 사용자: B만 접근 가능                                       │
│  - C 사용자: C, D 접근 가능                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 관계 요약

| 부모 테이블 | 자식 테이블 | 관계 | 외래키 |
|------------|------------|------|--------|
| companies | companies | 1:N | agency_id |
| companies | users | 1:N | company_id |
| companies | personality_sheets | 1:N | company_id |
| companies | personalities | 1:N | company_id |
| personality_sheets | personalities | 1:N | personality_sheet_id |
| personalities | personality_applicants | 1:N | personality_idx (UUID) |
| personality_test_types | personality_codes | 1:N | test_type |
| personality_codes | personality_questions | 1:N | code |
| reputation_tasks | reputation_applicants | 1:N | task_idx (UUID) |
| reputation_tasks | reputation_raters | 1:N | task_idx (UUID) |
| reputation_applicants | reputation_raters | 1:N | applicant_idx (UUID) |

## UUID vs Integer ID

| UUID 사용 (외부 노출) | Integer ID 사용 (내부) |
|---------------------|----------------------|
| personalities.idx | companies.id |
| personality_applicants.idx | users.id |
| reputation_tasks.idx | personality_sheets.id |
| reputation_applicants.idx | personality_codes.id |
| reputation_raters.idx | personality_questions.id |
| multifaceted_tasks.idx | |
| multifaceted_applicants.idx | |
| multifaceted_raters.idx | |
