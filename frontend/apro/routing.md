# 라우팅

> 라우트 정의 파일: `src/routes/`, 경로 상수: `src/paths.js`

APRO는 React Router v7의 `createBrowserRouter`를 사용하며, 모든 페이지는 **lazy loading**으로 코드 분할됩니다.

---

## 라우트 구성 파일

| 파일 | 역할 |
|------|------|
| `src/routes/index.jsx` | 최상위 라우트 (홈 리다이렉트, auth, dashboard, 404) |
| `src/routes/auth/index.jsx` | 인증 관련 라우트 |
| `src/routes/dashboard.jsx` | 대시보드 하위 전체 라우트 트리 |
| `src/paths.js` | 경로 문자열 상수 (하드코딩 방지) |

---

## 홈 & 에러

| 경로 | 동작 |
|------|------|
| `/` | 인증 상태면 `/dashboard`로, 미인증이면 `/auth/sign-in`으로 리다이렉트 |
| `*` (그 외) | `pages/not-found.jsx` 404 페이지 표시 |

---

## 인증 (Auth) 라우트

| 경로 | 페이지 파일 | 설명 |
|------|------------|------|
| `/auth/sign-in` | `pages/auth/sign-in.jsx` | 로그인 |
| `/auth/forgot-password` | `pages/auth/forgot-password.jsx` | 비밀번호 찾기 |
| `/auth/sign-out` | `pages/auth/sign-out.jsx` | 로그아웃 처리 후 홈으로 리다이렉트 |

---

## 대시보드 라우트

`/dashboard` 하위의 모든 경로는 `AuthGuard`로 보호됩니다. 미인증 시 자동으로 로그인 페이지로 리다이렉트됩니다.

### 메인

| 경로 | 페이지 파일 | 설명 |
|------|------------|------|
| `/dashboard` | `pages/dashboard/overview.jsx` | 대시보드 메인 (전체 검사 현황) |

---

### 인성검사 (Personality)

| 경로 | 페이지 파일 | 설명 |
|------|------------|------|
| `/dashboard/personality/task` | `pages/personality/index.jsx` | 검사 작업 목록 |
| `/dashboard/personality/task/create` | `pages/personality/create.jsx` | 검사 작업 등록 |
| `/dashboard/personality/task/detail/:id` | `pages/personality/details.jsx` | 검사 작업 상세 |
| `/dashboard/personality/sheet` | `pages/personality/sheet.jsx` | 검사지 목록 |
| `/dashboard/personality/sheet/create` | `pages/personality/sheet-create.jsx` | 검사지 등록 |
| `/dashboard/personality/sheet/:id` | `pages/personality/sheet_detail.jsx` | 검사지 상세 |
| `/dashboard/personality/code` | `pages/personality/code.jsx` | 코드 목록 |
| `/dashboard/personality/code/create` | `pages/personality/code_create.jsx` | 코드 등록 |
| `/dashboard/personality/code/:id` | `pages/personality/code_detail.jsx` | 코드 상세 |
| `/dashboard/personality/question` | `pages/personality/question.jsx` | 문항 목록 |
| `/dashboard/personality/question/create` | `pages/personality/question_create.jsx` | 문항 등록 |
| `/dashboard/personality/question/:id` | `pages/personality/question_detail.jsx` | 문항 상세 |
| `/dashboard/personality/data` | `pages/personality/data.jsx` | 데이터 관리 |

---

### 다면진단 (Multifaceted)

| 경로 | 페이지 파일 | 설명 |
|------|------------|------|
| `/dashboard/multifaceted/task` | `pages/multifaceted/index.jsx` | 진단 작업 목록 |
| `/dashboard/multifaceted/task/create` | `pages/multifaceted/create.jsx` | 진단 작업 등록 |
| `/dashboard/multifaceted/task/:id` | `pages/multifaceted/details.jsx` | 진단 작업 상세 |
| `/dashboard/multifaceted/sheet` | `pages/multifaceted/sheet.jsx` | 진단지 목록 |
| `/dashboard/multifaceted/sheet/create` | `pages/multifaceted/sheet-create.jsx` | 진단지 등록 |
| `/dashboard/multifaceted/sheet/:id` | `pages/multifaceted/sheet_detail.jsx` | 진단지 상세 |
| `/dashboard/multifaceted/code` | `pages/multifaceted/code.jsx` | 코드 목록 |
| `/dashboard/multifaceted/code/create` | `pages/multifaceted/code_create.jsx` | 코드 등록 |
| `/dashboard/multifaceted/code/:id` | `pages/multifaceted/code_detail.jsx` | 코드 상세 |
| `/dashboard/multifaceted/question` | `pages/multifaceted/question.jsx` | 문항 목록 |
| `/dashboard/multifaceted/question/create` | `pages/multifaceted/question_create.jsx` | 문항 등록 |
| `/dashboard/multifaceted/question/:id` | `pages/multifaceted/question_detail.jsx` | 문항 상세 |

---

### 평판진단 (Reputation)

| 경로 | 페이지 파일 | 설명 |
|------|------------|------|
| `/dashboard/reputation/task` | `pages/reputation/index.jsx` | 진단 작업 목록 |
| `/dashboard/reputation/task/create` | `pages/reputation/create.jsx` | 진단 작업 등록 |
| `/dashboard/reputation/task/:id` | `pages/reputation/details.jsx` | 진단 작업 상세 |
| `/dashboard/reputation/sheet` | `pages/reputation/sheet.jsx` | 진단지 목록 |
| `/dashboard/reputation/sheet/create` | `pages/reputation/sheet-create.jsx` | 진단지 등록 |
| `/dashboard/reputation/sheet/:id` | `pages/reputation/sheet_detail.jsx` | 진단지 상세 |
| `/dashboard/reputation/code` | `pages/reputation/code.jsx` | 코드 목록 |
| `/dashboard/reputation/code/create` | `pages/reputation/code_create.jsx` | 코드 등록 |
| `/dashboard/reputation/code/:id` | `pages/reputation/code_detail.jsx` | 코드 상세 |
| `/dashboard/reputation/question` | `pages/reputation/question.jsx` | 문항 목록 |
| `/dashboard/reputation/question/create` | `pages/reputation/question_create.jsx` | 문항 등록 |
| `/dashboard/reputation/question/:id` | `pages/reputation/question_detail.jsx` | 문항 상세 |
| `/dashboard/reputation/data` | `pages/reputation/data.jsx` | 데이터 관리 |

---

### 설정 / 마이페이지 (Settings)

설정 하위 경로는 `SettingsLayout`으로 감싸져 좌측 탭 네비게이션이 표시됩니다.

| 경로 | 페이지 파일 | 설명 |
|------|------------|------|
| `/dashboard/settings` | `pages/dashboard/settings/account.jsx` | 계정 설정 (기본) |
| `/dashboard/settings/account` | `pages/dashboard/settings/account.jsx` | 계정 설정 |
| `/dashboard/settings/company` | `pages/dashboard/settings/company.jsx` | 업체 목록 |
| `/dashboard/settings/company/create` | `pages/dashboard/settings/company-create.jsx` | 업체 등록 |
| `/dashboard/settings/company/:id` | `pages/dashboard/settings/company-detail.jsx` | 업체 상세 |
| `/dashboard/settings/notice` | `pages/dashboard/settings/notice.jsx` | 공지사항 목록 |
| `/dashboard/settings/notice/create` | `pages/dashboard/settings/notice-form.jsx` | 공지사항 작성 |
| `/dashboard/settings/notice/:id` | `pages/dashboard/settings/notice-form.jsx` | 공지사항 수정 |

---

## 경로 상수 (`paths.js`)

코드에서 경로를 하드코딩하지 않고 `paths` 객체를 사용합니다.

```javascript
import { paths } from '@/paths';

// 사용 예시
navigate(paths.dashboard.personality.home);          // "/dashboard/personality/task"
navigate(paths.dashboard.personality.detail('abc'));  // "/dashboard/personality/task/detail/abc"
navigate(paths.dashboard.mypage.account);             // "/dashboard/settings/account"
```

### 주요 경로 상수

```
paths.home                                     → "/"
paths.auth.signIn                              → "/auth/sign-in"
paths.auth.forgotPassword                      → "/auth/forgot-password"

paths.dashboard.overview                       → "/dashboard"

paths.dashboard.personality.home               → "/dashboard/personality/task"
paths.dashboard.personality.create             → "/dashboard/personality/task/create"
paths.dashboard.personality.detail(id)         → "/dashboard/personality/task/detail/{id}"
paths.dashboard.personality.sheet              → "/dashboard/personality/sheet"
paths.dashboard.personality.sheet_create       → "/dashboard/personality/sheet/create"
paths.dashboard.personality.sheet_detail(id)   → "/dashboard/personality/sheet/{id}"
paths.dashboard.personality.code               → "/dashboard/personality/code"
paths.dashboard.personality.code_detail(id)    → "/dashboard/personality/code/{id}"
paths.dashboard.personality.question           → "/dashboard/personality/question"
paths.dashboard.personality.question_detail(id)→ "/dashboard/personality/question/{id}"
paths.dashboard.personality.data               → "/dashboard/personality/data"

paths.dashboard.multifaceted.task              → "/dashboard/multifaceted/task"
paths.dashboard.multifaceted.task_create       → "/dashboard/multifaceted/task/create"
paths.dashboard.multifaceted.task_detail(id)   → "/dashboard/multifaceted/task/{id}"
paths.dashboard.multifaceted.sheet             → "/dashboard/multifaceted/sheet"
paths.dashboard.multifaceted.sheet_create      → "/dashboard/multifaceted/sheet/create"
paths.dashboard.multifaceted.sheet_detail(id)  → "/dashboard/multifaceted/sheet/{id}"
paths.dashboard.multifaceted.code              → "/dashboard/multifaceted/code"
paths.dashboard.multifaceted.code_detail(id)   → "/dashboard/multifaceted/code/{id}"
paths.dashboard.multifaceted.question          → "/dashboard/multifaceted/question"
paths.dashboard.multifaceted.question_detail(id)→"/dashboard/multifaceted/question/{id}"

paths.dashboard.reputation.task               → "/dashboard/reputation/task"
paths.dashboard.reputation.task_create        → "/dashboard/reputation/task/create"
paths.dashboard.reputation.task_detail(id)    → "/dashboard/reputation/task/{id}"
paths.dashboard.reputation.sheet              → "/dashboard/reputation/sheet"
paths.dashboard.reputation.sheet_create       → "/dashboard/reputation/sheet/create"
paths.dashboard.reputation.sheet_detail(id)   → "/dashboard/reputation/sheet/{id}"
paths.dashboard.reputation.code               → "/dashboard/reputation/code"
paths.dashboard.reputation.code_detail(id)    → "/dashboard/reputation/code/{id}"
paths.dashboard.reputation.question           → "/dashboard/reputation/question"
paths.dashboard.reputation.question_detail(id)→ "/dashboard/reputation/question/{id}"
paths.dashboard.reputation.data               → "/dashboard/reputation/data"

paths.dashboard.mypage.account                → "/dashboard/settings/account"
paths.dashboard.mypage.company                → "/dashboard/settings/company"
paths.dashboard.mypage.company_detail(id)     → "/dashboard/settings/company/{id}"
paths.dashboard.mypage.notice                 → "/dashboard/settings/notice"
paths.dashboard.settings.noticeCreate         → "/dashboard/settings/notice/create"
paths.dashboard.settings.noticeEdit(id)       → "/dashboard/settings/notice/{id}"
```

---

## 라우트 가드

모든 `/dashboard/*` 라우트는 `AuthGuard` 컴포넌트로 감싸져 있습니다.

```
/dashboard/* → AuthGuard → DashboardLayout → 페이지 컴포넌트
```

- 인증되지 않은 사용자는 자동으로 `/auth/sign-in`으로 리다이렉트
- 인증 확인 중에는 빈 화면(로딩) 표시
- 인증 상태는 `AuthContext`에서 관리
