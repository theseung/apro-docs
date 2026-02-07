# 디렉토리 구조

> 루트: `/apro/src/`

```
src/
├── index.jsx                     # 앱 진입점 (React DOM root, Router 생성)
├── root.jsx                      # Provider 트리 (Auth, Theme, i18n 등)
├── paths.js                      # 라우트 경로 상수
│
├── routes/                       # 라우트 정의
│   ├── index.jsx                 # 최상위 라우트 (홈, auth, dashboard, 404)
│   ├── auth/index.jsx            # 인증 라우트 (sign-in, forgot-password, sign-out)
│   └── dashboard.jsx             # 대시보드 하위 전체 라우트 트리
│
├── pages/                        # 페이지 컴포넌트 (lazy loaded)
│   ├── auth/                     # 인증 페이지
│   │   ├── sign-in.jsx
│   │   ├── forgot-password.jsx
│   │   └── sign-out.jsx
│   ├── dashboard/
│   │   ├── overview.jsx          # 대시보드 메인
│   │   └── settings/             # 설정 페이지
│   │       ├── account.jsx
│   │       ├── company.jsx
│   │       ├── company-create.jsx
│   │       ├── company-detail.jsx
│   │       ├── notice.jsx
│   │       └── notice-form.jsx
│   ├── personality/              # 인성검사 페이지 (13개)
│   │   ├── index.jsx             # 작업 목록
│   │   ├── create.jsx            # 작업 등록
│   │   ├── details.jsx           # 작업 상세
│   │   ├── sheet.jsx             # 검사지 목록
│   │   ├── sheet-create.jsx      # 검사지 등록
│   │   ├── sheet_detail.jsx      # 검사지 상세
│   │   ├── code.jsx              # 코드 목록
│   │   ├── code_create.jsx       # 코드 등록
│   │   ├── code_detail.jsx       # 코드 상세
│   │   ├── question.jsx          # 문항 목록
│   │   ├── question_create.jsx   # 문항 등록
│   │   ├── question_detail.jsx   # 문항 상세
│   │   └── data.jsx              # 데이터 관리
│   ├── multifaceted/             # 다면진단 페이지 (12개, data 없음)
│   │   └── (personality와 동일 구조)
│   ├── reputation/               # 평판진단 페이지 (13개)
│   │   └── (personality와 동일 구조)
│   └── not-found.jsx             # 404 페이지
│
├── components/                   # 컴포넌트
│   ├── auth/                     # 인증 관련
│   │   ├── auth-guard.jsx        # 인증 가드 (미인증 → 리다이렉트)
│   │   ├── split-layout.jsx      # 인증 페이지 레이아웃
│   │   └── custom/
│   │       └── auth-context.jsx  # AuthProvider & useAuth
│   │
│   ├── core/                     # 공통 UI 컴포넌트
│   │   ├── data-table.jsx        # 데이터 테이블
│   │   ├── filter-button.jsx     # 필터 버튼
│   │   ├── multi-select.jsx      # 다중 선택
│   │   ├── link.jsx              # 라우터 링크 래퍼
│   │   ├── logo.jsx              # 로고
│   │   ├── toaster.jsx           # 토스트 알림
│   │   ├── theme-provider.jsx    # MUI 테마
│   │   ├── i18n-provider.jsx     # 다국어
│   │   ├── localization-provider.jsx
│   │   ├── dropdown/             # 드롭다운 컴포넌트 세트
│   │   ├── settings/             # 테마 설정 UI
│   │   └── text-editor/          # 리치 텍스트 에디터
│   │
│   ├── dashboard/                # 대시보드 전용
│   │   ├── layout/               # 레이아웃 시스템
│   │   │   ├── layout.jsx        # vertical/horizontal 분기
│   │   │   ├── vertical/         # 사이드바 레이아웃
│   │   │   ├── horizontal/       # 상단바 레이아웃
│   │   │   ├── nav-icons.jsx     # 아이콘 매핑
│   │   │   ├── mobile-nav.jsx    # 모바일 네비
│   │   │   └── user-popover/     # 사용자 메뉴
│   │   ├── overview/             # 대시보드 메인 테이블
│   │   ├── settings/             # 설정 페이지 레이아웃
│   │   ├── test/                 # 인성검사 테이블 컴포넌트
│   │   ├── multifaceted/         # 다면진단 테이블 컴포넌트
│   │   └── reputation/           # 평판진단 테이블 컴포넌트
│   │
│   ├── personality/              # 인성검사 비즈니스 컴포넌트
│   │   ├── applicant.jsx         # 응시자 관리
│   │   ├── applicant-status.jsx  # 상태 표시
│   │   ├── applicant-result-table.jsx
│   │   ├── applicant-download.jsx
│   │   ├── register.jsx          # 등록 폼
│   │   ├── result.jsx            # 결과 표시
│   │   ├── sheet-composition.jsx # 검사지 구성
│   │   ├── item-composition.jsx  # 항목 구성
│   │   ├── problem-preview-modal.jsx
│   │   └── settlement-formula.jsx
│   │
│   ├── multifaceted/             # 다면진단 비즈니스 컴포넌트
│   │   └── (personality + rater.jsx)
│   ├── reputation/               # 평판진단 비즈니스 컴포넌트
│   │   └── (personality + rater.jsx)
│   │
│   ├── settings/                 # 관리자 모달
│   │   ├── admin-add-modal.jsx
│   │   └── admin-edit-modal.jsx
│   └── widgets/
│       └── previewer.jsx
│
├── hooks/                        # 커스텀 React Hooks
│   ├── use-all-tests.js          # 전체 검사 데이터 fetch
│   ├── use-dashboard.js          # 대시보드 데이터
│   ├── use-dialog.js             # 다이얼로그 상태
│   ├── use-media-query.js        # 반응형 브레이크포인트
│   ├── use-pathname.js           # 현재 경로명
│   ├── use-popover.js            # 팝오버 상태
│   └── use-selection.js          # 테이블 행 선택
│
├── lib/                          # 유틸리티 & API
│   ├── api-client.js             # HTTP 클라이언트 (fetch 래퍼, 토큰 관리)
│   ├── custom-auth/
│   │   └── client.js             # 인증 클라이언트 (로그인/로그아웃)
│   ├── dayjs.js                  # Day.js 설정
│   ├── default-logger.js         # 기본 로거
│   ├── logger.js                 # 로거 인터페이스
│   ├── get-app-url.js            # 앱 URL 유틸
│   ├── is-nav-item-active.js     # 네비게이션 활성 상태 판별
│   └── settings.js               # 설정 유틸 (localStorage 기반)
│
├── config/                       # 설정
│   ├── app.js                    # 앱 이름("APRO"), 테마, 색상
│   ├── dashboard.js              # 네비게이션 메뉴 (권한별 필터링)
│   └── index.js                  # config 배럴 export
│
├── locales/                      # 다국어 리소스
│   ├── en/index.js               # 영어
│   ├── de/index.js               # 독일어
│   └── es/index.js               # 스페인어
│
├── styles/                       # 스타일
│   ├── theme/                    # MUI 테마 커스터마이징
│   └── global.css                # 글로벌 CSS
│
└── _unused/                      # 미사용 컴포넌트 보관함
```
