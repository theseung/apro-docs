# APRO (관리자 대시보드)

## 개요

관리자용 React 웹 애플리케이션입니다.

- **위치**: `/apro`
- **기술 스택**: React 18, Material-UI 7, CRA (craco)
- **빌드 도구**: Create React App + CRACO

## 디렉토리 구조

```
apro/
├── src/
│   ├── components/           # 컴포넌트
│   │   ├── auth/             # 인증 관련
│   │   │   └── custom/       # 커스텀 인증 컨텍스트
│   │   ├── core/             # 핵심 컴포넌트
│   │   │   ├── theme-provider/
│   │   │   ├── i18n-provider/
│   │   │   ├── settings/
│   │   │   └── toaster/
│   │   ├── dashboard/        # 대시보드 전용
│   │   ├── personality/      # 인성검사 컴포넌트
│   │   ├── multifaceted/     # 다면평가 컴포넌트
│   │   └── reputation/       # 평판진단 컴포넌트
│   │
│   ├── pages/                # 페이지 컴포넌트
│   │   ├── auth/             # 로그인/회원가입
│   │   └── dashboard/        # 대시보드 페이지
│   │       ├── personality/  # 인성검사 관리
│   │       ├── multifaceted/ # 다면평가 관리
│   │       ├── reputation/   # 평판진단 관리
│   │       └── settings/     # 설정
│   │
│   ├── hooks/                # 커스텀 훅
│   │   ├── use-media-query.js
│   │   ├── use-popover.js
│   │   ├── use-dialog.js
│   │   └── use-selection.js
│   │
│   ├── lib/                  # 유틸리티
│   │   ├── api-client.js     # API 통신
│   │   ├── dayjs.js          # 날짜 처리
│   │   ├── logger.js         # 로깅
│   │   └── settings.js       # 설정 관리
│   │
│   ├── config/               # 설정
│   │   ├── app.js            # 앱 설정
│   │   └── dashboard.js      # 네비게이션 메뉴
│   │
│   ├── routes/               # 라우팅
│   │   ├── index.jsx         # 메인 라우터
│   │   └── dashboard.jsx     # 대시보드 라우트
│   │
│   ├── locales/              # 다국어 리소스
│   ├── styles/               # 스타일
│   ├── paths.js              # 경로 상수
│   ├── root.jsx              # 루트 컴포넌트
│   └── index.jsx             # 진입점
│
├── public/                   # 정적 파일
├── build/                    # 빌드 결과
├── package.json
└── craco.config.js           # CRA 설정 오버라이드
```

## 개발 명령어

```bash
cd apro

# 개발 서버
npm start

# 프로덕션 빌드
npm run build

# 테스트
npm test
```

## API 클라이언트 (`src/lib/api-client.js`)

### 서버 URL

```javascript
export function getServerUrl() {
  if (window.location.hostname === 'localhost') {
    return 'http://localhost/api';
  }
  return 'https://api.aprofiler.co.kr/api';
}
```

### 기본 사용법

```javascript
import { apiGet, apiPost, apiPut, apiDelete } from '../lib/api-client';

// GET 요청
const response = await apiGet('/personality');

// POST 요청
const result = await apiPost('/personality', {
  name: '테스트',
  company_id: 1
});

// PUT 요청
await apiPut(`/personality/task/${idx}`, data);

// DELETE 요청
await apiDelete(`/personality/task/${idx}`);
```

### 토큰 관리

```javascript
// 토큰 저장
setAccessToken(token);
setRefreshToken(refreshToken);

// 토큰 조회
const token = getAccessToken();

// 토큰 삭제
clearTokens();
```

### 파일 다운로드

```javascript
import { downloadFile } from '../lib/api-client';

await downloadFile('/personality/task/123/download-excel', 'report.xlsx');
```

## 인증 컨텍스트 (`src/components/auth/custom/auth-context.jsx`)

### Provider 구조

```jsx
<CustomAuthProvider>
  <App />
</CustomAuthProvider>
```

### 사용법

```jsx
import { useAuth } from '../components/auth/custom/auth-context';

function MyComponent() {
  const { user, isAuthenticated, signIn, signOut } = useAuth();

  const handleLogin = async () => {
    await signIn({ email, password });
  };

  return (
    <div>
      {isAuthenticated ? (
        <span>Welcome, {user.name}</span>
      ) : (
        <button onClick={handleLogin}>Login</button>
      )}
    </div>
  );
}
```

## 라우팅 (`src/routes/`)

### 메인 라우터 (`index.jsx`)

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <HomeRedirect />,  // 인증 상태에 따라 리다이렉트
  },
  {
    path: '/auth/*',
    element: <AuthRoutes />,
  },
  {
    path: '/dashboard/*',
    element: <DashboardRoutes />,
  },
]);
```

### 경로 상수 (`src/paths.js`)

```javascript
export const paths = {
  home: '/',
  auth: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    forgotPassword: '/auth/forgot-password',
  },
  dashboard: {
    overview: '/dashboard',
    personality: {
      index: '/dashboard/personality',
      create: '/dashboard/personality/create',
      details: (idx) => `/dashboard/personality/${idx}`,
    },
    // ...
  },
};
```

## 네비게이션 설정 (`src/config/dashboard.js`)

```javascript
export function getFilteredNavItems(user) {
  const baseItems = [
    {
      key: 'overview',
      title: '대시보드',
      href: paths.dashboard.overview,
      icon: 'house',
    },
    {
      key: 'personality',
      title: '인성검사',
      href: paths.dashboard.personality.index,
      icon: 'clipboard',
    },
    // ...
  ];

  // 특정 회사만 볼 수 있는 메뉴
  if (user.company_id === 1) {
    baseItems.push({
      key: 'admin',
      title: '관리자 설정',
      href: paths.dashboard.admin,
      icon: 'gear',
    });
  }

  return baseItems;
}
```

## 주요 페이지

### 인성검사 목록 (`pages/dashboard/personality/index.jsx`)

- 검사 목록 테이블
- 상태별 필터링
- 검색 기능
- 페이지네이션

### 인성검사 상세 (`pages/dashboard/personality/details.jsx`)

- 검사 정보 수정
- 응시자 목록 관리
- 대량 등록 (Excel)
- 결과 다운로드

### 검사지 관리 (`pages/dashboard/personality/sheet/`)

- 검사지 CRUD
- 코드/문항 선택
- 문항 순서 설정

## UI 컴포넌트

### Material-UI 기반

```jsx
import {
  Button,
  TextField,
  Table,
  Dialog,
  Card,
} from '@mui/material';
```

### 커스텀 훅

```jsx
// 팝오버
const { anchorEl, open, handleOpen, handleClose } = usePopover();

// 다이얼로그
const { open, handleOpen, handleClose } = useDialog();

// 다중 선택
const { selected, handleSelect, handleSelectAll, handleDeselectAll } = useSelection(items);
```

## 테마 설정

```jsx
// src/root.jsx
<ThemeProvider
  colorPreset={settings.colorPreset}
  direction={settings.direction}
>
  <CssBaseline />
  <App />
</ThemeProvider>
```

## 다국어 지원

```jsx
import { useTranslation } from 'react-i18next';

function MyComponent() {
  const { t, i18n } = useTranslation();

  return (
    <div>
      <h1>{t('dashboard.title')}</h1>
      <button onClick={() => i18n.changeLanguage('en')}>
        English
      </button>
    </div>
  );
}
```
