# 네비게이션 메뉴 구조

> 설정 파일: `src/config/dashboard.js`

네비게이션 메뉴는 사용자의 `company_id`에 따라 동적으로 필터링됩니다.

---

## 메뉴 트리

```
대시보드                              (모든 사용자)

주요 서비스
  인성검사
    ├── 검사 작업 관리                 (모든 사용자)
    ├── 검사지 관리                    (company_id === 1)
    ├── 검사 문항 관리                  (company_id === 1)
    ├── 코드 관리                      (company_id === 1)
    └── 데이터 관리                    (company_id === 1)
  다면진단
    ├── 진단 작업 관리                  (모든 사용자)
    ├── 진단지 관리                    (company_id === 1)
    ├── 진단 문항 관리                  (company_id === 1)
    └── 코드 관리                      (company_id === 1)
  평판진단
    ├── 진단 작업 관리                  (모든 사용자)
    ├── 진단지 관리                    (company_id === 1)
    ├── 진단 문항 관리                  (company_id === 1)
    ├── 코드 관리                      (company_id === 1)
    └── 데이터 관리                    (company_id === 1)

마이페이지
  ├── 관리자 정보                      (모든 사용자)
  ├── 업체 관리                        (company_id === 1 또는 2)
  ├── 공지사항                         (company_id === 1 또는 2)
  ├── 상담센터                         (외부 링크, 모든 사용자)
  └── 로그아웃                         (모든 사용자)
```

---

## 권한별 접근 범위

| 권한 | company_id | 메뉴 접근 |
|------|-----------|----------|
| 슈퍼 관리자 | `1` | 전체 메뉴 |
| 에이전시 | `2` | 작업 관리 + 업체/공지 관리 |
| 일반 기업 | 기타 | 작업 관리 + 관리자 정보 + 상담센터 |

---

## 메뉴 항목 구조

`config/dashboard.js`의 `getFilteredNavItems(user)` 함수가 메뉴를 생성합니다.

```javascript
// 메뉴 항목 형식
{
  key: "personality",        // 고유 키
  title: "인성검사",          // 표시 텍스트
  icon: "listcheck",         // nav-icons.jsx에서 매핑되는 아이콘 키
  href: "/dashboard/...",    // 링크 (leaf 항목)
  items: [...],              // 하위 메뉴 (중첩 가능)
  external: true,            // 외부 링크 여부
  button: true,              // 버튼 스타일 여부 (로그아웃 등)
}
```

---

## 메뉴 → 라우트 매핑

### 인성검사

| 메뉴 항목 | 라우트 경로 |
|----------|-----------|
| 검사 작업 관리 | `/dashboard/personality/task` |
| 검사지 관리 | `/dashboard/personality/sheet` |
| 검사 문항 관리 | `/dashboard/personality/question` |
| 코드 관리 | `/dashboard/personality/code` |
| 데이터 관리 | `/dashboard/personality/data` |

### 다면진단

| 메뉴 항목 | 라우트 경로 |
|----------|-----------|
| 진단 작업 관리 | `/dashboard/multifaceted/task` |
| 진단지 관리 | `/dashboard/multifaceted/sheet` |
| 진단 문항 관리 | `/dashboard/multifaceted/question` |
| 코드 관리 | `/dashboard/multifaceted/code` |

### 평판진단

| 메뉴 항목 | 라우트 경로 |
|----------|-----------|
| 진단 작업 관리 | `/dashboard/reputation/task` |
| 진단지 관리 | `/dashboard/reputation/sheet` |
| 진단 문항 관리 | `/dashboard/reputation/question` |
| 코드 관리 | `/dashboard/reputation/code` |
| 데이터 관리 | `/dashboard/reputation/data` |

### 마이페이지

| 메뉴 항목 | 라우트 경로 |
|----------|-----------|
| 관리자 정보 | `/dashboard/settings/account` |
| 업체 관리 | `/dashboard/settings/company` |
| 공지사항 | `/dashboard/settings/notice` |
| 상담센터 | `https://pf.kakao.com/_nZqxln` (외부) |
| 로그아웃 | `/auth/sign-out` |

---

## 아이콘 매핑 (`nav-icons.jsx`)

`config/dashboard.js`에서 사용하는 아이콘 키와 실제 Phosphor Icon의 매핑은 `components/dashboard/layout/nav-icons.jsx`에서 관리됩니다.

```javascript
// 예시
{
  "PresentationChart": <PresentationChart />,
  "listcheck":         <ListChecks />,
  "userGrear":         <UserGear />,
  "UserList":          <UserList />,
  "users":             <Users />,
  "company":           <Buildings />,
  "notification":      <Bell />,
  "question":          <Question />,
  "signOut":           <SignOut />,
}
```

---

## 레이아웃 모드

네비게이션은 두 가지 레이아웃 모드를 지원합니다:

| 모드 | 설명 | 렌더링 컴포넌트 |
|------|------|---------------|
| **vertical** (기본) | 좌측 사이드바 + 상단 바 | `vertical-layout.jsx` + `side-nav.jsx` |
| **horizontal** | 상단 바에 메뉴 통합 | `horizontal-layout.jsx` + `main-nav.jsx` |

레이아웃 모드는 설정 드로어에서 변경 가능하며, `SettingsContext`를 통해 관리됩니다.
