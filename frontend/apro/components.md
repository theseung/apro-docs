# 컴포넌트

> 위치: `src/components/`

컴포넌트는 용도에 따라 분류되어 있으며, 페이지에서 조합하여 사용합니다.

---

## 컴포넌트 분류 개요

```
components/
├── auth/               # 인증 관련 (가드, 레이아웃, Context)
├── core/               # 공통 UI (테이블, 드롭다운, 에디터, 테마 등)
├── dashboard/          # 대시보드 레이아웃 & 테이블
│   ├── layout/         # 레이아웃 시스템 (vertical/horizontal)
│   ├── overview/       # 대시보드 메인
│   ├── settings/       # 설정 레이아웃
│   ├── test/           # 인성검사 테이블
│   ├── multifaceted/   # 다면진단 테이블
│   └── reputation/     # 평판진단 테이블
├── personality/        # 인성검사 비즈니스 컴포넌트
├── multifaceted/       # 다면진단 비즈니스 컴포넌트
├── reputation/         # 평판진단 비즈니스 컴포넌트
├── settings/           # 관리자 모달
└── widgets/            # 위젯
```

---

## 인증 컴포넌트 (`components/auth/`)

| 파일 | 설명 |
|------|------|
| `auth-guard.jsx` | 인증 가드. 미인증 시 `/auth/sign-in`으로 리다이렉트 |
| `split-layout.jsx` | 인증 페이지 좌우 분할 레이아웃 |
| `custom/auth-context.jsx` | `AuthProvider` & `useAuth()` 훅. 인증 상태(user, isAuthenticated, isLoading) 관리 |

---

## 코어 컴포넌트 (`components/core/`)

재사용 가능한 공통 UI 컴포넌트입니다.

### 기본 UI

| 파일 | 설명 |
|------|------|
| `data-table.jsx` | 공통 데이터 테이블 (정렬, 선택, 페이지네이션) |
| `filter-button.jsx` | 필터 버튼 + 팝오버 |
| `multi-select.jsx` | 다중 선택 드롭다운 |
| `option.jsx` | Select Option 컴포넌트 |
| `link.jsx` | React Router 링크 래퍼 |
| `logo.jsx` | 애플리케이션 로고 |
| `breadcrumbs-separator.jsx` | 브레드크럼 구분자 |
| `presence.jsx` | 온라인 상태 표시 |
| `property-list.jsx` | 속성 목록 컨테이너 (key-value 쌍 표시) |
| `property-item.jsx` | 속성 항목 |
| `toaster.jsx` | 토스트 알림 (Sonner 기반) |
| `no-ssr.jsx` | SSR 방지 래퍼 |
| `scroll-restoration.jsx` | 스크롤 위치 복원 |

### 드롭다운 (`components/core/dropdown/`)

| 파일 | 설명 |
|------|------|
| `dropdown.jsx` | 드롭다운 메인 컨테이너 |
| `dropdown-context.jsx` | 드롭다운 상태 Context |
| `dropdown-trigger.jsx` | 드롭다운 트리거 (클릭 영역) |
| `dropdown-popover.jsx` | 드롭다운 팝오버 (펼침 영역) |

### 테마 설정 (`components/core/settings/`)

우측 하단 설정 버튼을 통해 테마를 커스터마이징합니다.

| 파일 | 설명 |
|------|------|
| `settings-context.jsx` | 설정 상태 Context (색상, 레이아웃, 방향 등) |
| `settings-button.jsx` | 설정 열기 버튼 (우측 하단 FAB) |
| `settings-drawer.jsx` | 설정 드로어 패널 |
| `options-color-scheme.jsx` | 라이트/다크 모드 선택 |
| `options-direction.jsx` | 텍스트 방향 (LTR/RTL) 선택 |
| `options-layout.jsx` | 레이아웃 (vertical/horizontal) 선택 |
| `options-nav-color.jsx` | 네비게이션 색상 선택 |
| `options-primary-color.jsx` | 메인 브랜드 색상 선택 |

### 리치 텍스트 에디터 (`components/core/text-editor/`)

| 파일 | 설명 |
|------|------|
| `text-editor.jsx` | TipTap 기반 리치 텍스트 에디터 |
| `text-editor-toolbar.jsx` | 에디터 툴바 (볼드, 이탤릭, 리스트 등) |

### Provider 컴포넌트

| 파일 | 설명 |
|------|------|
| `theme-provider.jsx` | MUI 테마 Provider |
| `localization-provider.jsx` | MUI 로케일(날짜 등) Provider |
| `i18n-provider.jsx` | 다국어 Provider (react-i18next) |
| `rtl.jsx` | RTL 지원 래퍼 |
| `analytics.jsx` | 분석 도구 연동 |

---

## 레이아웃 컴포넌트 (`components/dashboard/layout/`)

### 레이아웃 시스템

| 파일 | 설명 |
|------|------|
| `layout.jsx` | 레이아웃 분기 (vertical 또는 horizontal, 설정에 따라 전환) |

### Vertical 레이아웃 (기본)

좌측 사이드바 + 상단 바 구조입니다.

| 파일 | 설명 |
|------|------|
| `vertical/vertical-layout.jsx` | 수직 레이아웃 컨테이너 |
| `vertical/side-nav.jsx` | 좌측 사이드바 네비게이션 |
| `vertical/main-nav.jsx` | 상단 네비게이션 바 |
| `vertical/styles.js` | 스타일 상수 (너비, 높이 등) |

### Horizontal 레이아웃

상단 바에 네비게이션이 통합된 구조입니다.

| 파일 | 설명 |
|------|------|
| `horizontal/horizontal-layout.jsx` | 수평 레이아웃 컨테이너 |
| `horizontal/main-nav.jsx` | 수평 메인 네비게이션 |
| `horizontal/styles.js` | 스타일 상수 |

### 공통 레이아웃 컴포넌트

| 파일 | 설명 |
|------|------|
| `nav-icons.jsx` | 네비게이션 아이콘 매핑 (key → Phosphor Icon) |
| `mobile-nav.jsx` | 모바일 반응형 네비게이션 (드로어) |
| `search-dialog.jsx` | 검색 다이얼로그 |
| `notifications-popover.jsx` | 알림 팝오버 |
| `language-popover.jsx` | 언어 선택 팝오버 |
| `workspaces-switch.jsx` | 워크스페이스 전환 |
| `workspaces-popover.jsx` | 워크스페이스 목록 팝오버 |
| `contacts-popover.jsx` | 연락처 팝오버 |

### 사용자 메뉴 (`components/dashboard/layout/user-popover/`)

| 파일 | 설명 |
|------|------|
| `user-popover.jsx` | 사용자 프로필 팝오버 메뉴 |
| `custom-sign-out.jsx` | 로그아웃 버튼 핸들러 |

---

## 테이블 컴포넌트 (`components/dashboard/`)

각 진단 유형별로 데이터 테이블 컴포넌트가 분리되어 있습니다.

### 대시보드 메인

| 파일 | 설명 |
|------|------|
| `overview/all-tests-table.jsx` | 전체 검사 통합 테이블 |

### 인성검사 테이블 (`components/dashboard/test/`)

| 파일 | 설명 |
|------|------|
| `TestList-table.jsx` | 검사 작업 목록 테이블 |
| `Sheet-table.jsx` | 검사지 목록 테이블 |
| `code-table.jsx` | 코드 목록 테이블 |
| `question-table.jsx` | 문항 목록 테이블 |
| `applicant-table.jsx` | 응시자 목록 테이블 |

### 다면진단 테이블 (`components/dashboard/multifaceted/`)

| 파일 | 설명 |
|------|------|
| `TestList-table.jsx` | 진단 작업 목록 테이블 |
| `Sheet-table.jsx` | 진단지 목록 테이블 |
| `code-table.jsx` | 코드 목록 테이블 |
| `question-table.jsx` | 문항 목록 테이블 |
| `applicant-table.jsx` | 응시자 목록 테이블 |
| `rater-table.jsx` | 평가자 목록 테이블 |

### 평판진단 테이블 (`components/dashboard/reputation/`)

다면진단과 동일한 구조 (`rater-table.jsx` 포함).

### 설정

| 파일 | 설명 |
|------|------|
| `settings/layout.jsx` | 설정 페이지 좌측 탭 레이아웃 |

---

## 비즈니스 컴포넌트

세 가지 진단 유형(personality, multifaceted, reputation)이 동일한 패턴의 컴포넌트를 가집니다.

### 공통 구조 (`components/[personality|multifaceted|reputation]/`)

| 파일 | 설명 |
|------|------|
| `applicant.jsx` | 응시자 관리 (등록, 삭제, 엑셀 업로드) |
| `applicant-status.jsx` | 응시자 상태 표시 (미시작/진행중/완료) |
| `applicant-status-table.jsx` | 응시자 상태 테이블 |
| `applicant-result-table.jsx` | 응시자 결과 테이블 |
| `applicant-download.jsx` | 응시자 데이터/결과 다운로드 |
| `register.jsx` | 등록 폼 (응시자 개별/일괄 등록) |
| `result.jsx` | 결과 표시/분석 |
| `sheet-composition.jsx` | 검사지/진단지 구성 표시 |
| `item-composition.jsx` | 항목(문항) 구성 표시 |
| `problem-preview-modal.jsx` | 문제 미리보기 모달 |
| `settlement-formula.jsx` | 점수 산출 공식 설정 |

### 다면진단/평판진단 추가 컴포넌트

| 파일 | 설명 |
|------|------|
| `rater.jsx` | 평가자 관리 (다면진단, 평판진단에만 존재) |

---

## 설정 컴포넌트 (`components/settings/`)

| 파일 | 설명 |
|------|------|
| `admin-add-modal.jsx` | 관리자 추가 모달 (업체 상세에서 사용) |
| `admin-edit-modal.jsx` | 관리자 수정 모달 |

---

## 위젯 (`components/widgets/`)

| 파일 | 설명 |
|------|------|
| `previewer.jsx` | 컴포넌트 미리보기 래퍼 |

---

## 컴포넌트 계층 구조

### 전체 앱 Provider 트리

```
<HelmetProvider>
  <AuthProvider>
    <Analytics>
      <LocalizationProvider>
        <SettingsProvider>
          <I18nProvider>
            <Rtl>
              <ThemeProvider>
                <RouterProvider />
                <Toaster />
              </ThemeProvider>
            </Rtl>
          </I18nProvider>
        </SettingsProvider>
      </LocalizationProvider>
    </Analytics>
  </AuthProvider>
</HelmetProvider>
```

### 대시보드 레이아웃 트리 (Vertical)

```
<AuthGuard>
  <VerticalLayout>
    ├── <SideNav>           ← 좌측 사이드바
    │     └── NavItems      ← 메뉴 항목 (재귀 렌더링)
    └── <Box component="main">
          ├── <MainNav>     ← 상단 바 (검색, 알림, 유저 메뉴)
          └── <Outlet>      ← 페이지 컨텐츠
```
