# 인증 시스템

> 관련 파일: `src/components/auth/`, `src/lib/custom-auth/client.js`, `src/lib/api-client.js`

---

## 인증 흐름

```
앱 시작
  │
  ▼
AuthProvider 초기화
  │
  ├─ localStorage에 토큰 존재?
  │    ├─ Yes → GET /user/me 호출
  │    │         ├─ 성공 → isAuthenticated: true, user 설정
  │    │         └─ 실패 → 토큰 삭제, isAuthenticated: false
  │    └─ No  → isAuthenticated: false
  │
  ▼
/ (홈) 접속
  │
  ├─ 인증됨   → /dashboard로 리다이렉트
  └─ 미인증   → /auth/sign-in으로 리다이렉트
```

---

## 핵심 파일

### Auth Context (`components/auth/custom/auth-context.jsx`)

인증 상태를 React Context로 관리합니다.

**제공하는 상태:**

| 속성 | 타입 | 설명 |
|------|------|------|
| `isAuthenticated` | `boolean` | 인증 여부 |
| `isLoading` | `boolean` | 인증 확인 중 여부 |
| `user` | `object \| null` | 로그인된 사용자 정보 |
| `setUser` | `function` | 사용자 정보 업데이트 |

**사용법:**

```jsx
import { useAuth } from '@/components/auth/custom/auth-context';

function MyComponent() {
  const { user, isAuthenticated } = useAuth();

  if (!isAuthenticated) return null;
  return <span>{user.name}</span>;
}
```

### Auth Client (`lib/custom-auth/client.js`)

실제 인증 API를 호출하는 클라이언트입니다.

| 메서드 | 설명 |
|--------|------|
| `signInWithPassword({ email, password })` | POST `/auth/login` 호출. 성공 시 토큰을 localStorage에 저장 |
| `getUser()` | GET `/user/me` 호출. 현재 인증된 사용자 정보 반환 |
| `signOut()` | 토큰 삭제 후 `auth:signout` 커스텀 이벤트 디스패치 |

### Auth Guard (`components/auth/auth-guard.jsx`)

보호된 라우트를 감싸는 래퍼 컴포넌트입니다.

- 인증되지 않은 사용자 → `/auth/sign-in`으로 리다이렉트
- 인증 확인 중 → 빈 화면 (로딩)
- 인증 완료 → children 렌더링

```
/dashboard/* 라우트 구조:
<AuthGuard>
  <DashboardLayout>
    <Outlet />    ← 페이지 컴포넌트
  </DashboardLayout>
</AuthGuard>
```

---

## 토큰 관리 (`lib/api-client.js`)

| 함수 | 설명 |
|------|------|
| `setAccessToken(token)` | access_token을 localStorage에 저장 |
| `setRefreshToken(token)` | refresh_token을 localStorage에 저장 |
| `setTokens(access, refresh)` | 두 토큰을 한 번에 저장 |
| `getAccessToken()` | access_token 조회 |
| `getRefreshToken()` | refresh_token 조회 |
| `clearTokens()` | 모든 토큰 삭제 |

**API 요청 시 자동으로 `Authorization: Bearer {token}` 헤더가 추가됩니다.**

---

## 로그인 순서

1. 사용자가 `/auth/sign-in`에서 이메일/비밀번호 입력
2. `authClient.signInWithPassword()` 호출
3. 서버가 `access_token` + `refresh_token` 반환
4. 토큰을 localStorage에 저장
5. `AuthContext`의 `setUser()`로 사용자 정보 설정
6. `/dashboard`로 리다이렉트

## 로그아웃 순서

1. 사이드바 "로그아웃" 클릭 또는 `/auth/sign-out` 접속
2. `authClient.signOut()` 호출
3. localStorage에서 토큰 삭제
4. `window.dispatchEvent(new Event('auth:signout'))` 발생
5. `AuthProvider`가 이벤트를 감지하여 상태 초기화
6. `/` (홈)으로 리다이렉트 → 미인증이므로 `/auth/sign-in`으로 이동
