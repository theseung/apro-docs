# API 클라이언트

> 위치: `src/lib/api-client.js`

Fetch API를 래핑한 커스텀 HTTP 클라이언트입니다. 토큰 관리, 에러 처리, 파일 다운로드를 포함합니다.

---

## 서버 URL

```javascript
import { getServerUrl } from '@/lib/api-client';

// 자동 분기:
// - localhost 접속 시: http://localhost/api
// - 운영 환경:        https://api.aprofiler.co.kr/api
```

---

## API 함수

### 기본 HTTP 메서드

| 함수 | 설명 | 사용 예시 |
|------|------|----------|
| `apiGet(url)` | GET 요청 | `apiGet('/personality')` |
| `apiPost(url, data)` | POST 요청 | `apiPost('/personality', { name: '검사명' })` |
| `apiPut(url, data)` | PUT 요청 | `apiPut('/personality/task/123', data)` |
| `apiDelete(url)` | DELETE 요청 | `apiDelete('/personality/task/123')` |

### 범용 API 함수

```javascript
import { api } from '@/lib/api-client';

// 쿼리 파라미터 포함
const response = await api('/personality', 'GET', null, { page: 1, per_page: 10 });

// POST with body
const result = await api('/personality', 'POST', { name: '테스트' });
```

### 파일 업로드

```javascript
import { apiUpload } from '@/lib/api-client';

const formData = new FormData();
formData.append('file', file);

const result = await apiUpload('/personality/task/123/upload', 'POST', formData);
```

### 파일 다운로드

```javascript
import { downloadFile, downloadFilePost } from '@/lib/api-client';

// GET 방식 다운로드
await downloadFile('/personality/task/123/download-excel', 'report.xlsx');

// POST 방식 다운로드 (필터 조건 포함)
await downloadFilePost('/personality/task/123/download', { type: 'pdf' }, 'report.pdf');
```

---

## 요청 헤더

모든 API 요청에 자동으로 추가되는 헤더:

| 헤더 | 값 | 설명 |
|------|-----|------|
| `Content-Type` | `application/json` | JSON 요청 (업로드 제외) |
| `Accept` | `application/json` | JSON 응답 기대 |
| `Authorization` | `Bearer {token}` | 인증 토큰 (저장된 경우) |

---

## 응답 형식

서버 응답은 다음 형식을 따릅니다:

```javascript
// 성공
{
  success: true,
  message: "처리되었습니다.",
  data: { ... }
}

// 실패
{
  success: false,
  error_message: "오류 메시지"
}
```

---

## 토큰 관리 함수

| 함수 | 설명 |
|------|------|
| `setAccessToken(token)` | access_token 저장 |
| `setRefreshToken(token)` | refresh_token 저장 |
| `setTokens(access, refresh)` | 두 토큰 동시 저장 |
| `getAccessToken()` | access_token 조회 |
| `getRefreshToken()` | refresh_token 조회 |
| `clearTokens()` | 모든 토큰 삭제 |
| `removeToken()` | 토큰 삭제 (clearTokens 별칭) |
| `getToken()` | access_token 조회 (getAccessToken 별칭) |
