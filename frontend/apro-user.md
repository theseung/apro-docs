# APRO_USER (응시자 프론트엔드)

## 개요

응시자(지원자)용 React 웹 애플리케이션입니다.

- **위치**: `/apro_user`
- **기술 스택**: React 19, TypeScript, Vite, Material-UI 7
- **빌드 도구**: Vite

## 디렉토리 구조

```
apro_user/
├── src/
│   ├── pages/                # 페이지 컴포넌트
│   │   ├── login.jsx         # 로그인
│   │   ├── landing.jsx       # 인성검사 랜딩
│   │   ├── test.jsx          # 인성검사 응시
│   │   ├── end.jsx           # 완료 페이지
│   │   ├── privacy.tsx       # 개인정보처리방침
│   │   │
│   │   ├── multifacetedLandingPage.jsx      # 다면평가 랜딩
│   │   ├── multifacetedStatusPage.jsx       # 평가자 현황
│   │   ├── multifacetedRequestPage.jsx      # 평가자 요청
│   │   ├── multifacetedTestPage.jsx         # 다면평가 테스트
│   │   ├── multifacetedDescriptivePage.jsx  # 서술형 평가
│   │   ├── multifacetedRaterLandingPage.jsx # 평가자 랜딩
│   │   ├── multifacetedRaterChkInfoPage.jsx # 평가자 정보확인
│   │   │
│   │   └── preview/          # 미리보기 페이지들
│   │
│   ├── components/           # 컴포넌트
│   │   ├── AppTextField.tsx  # 커스텀 텍스트 필드
│   │   └── MailPreviewDialog.jsx # 메일 미리보기
│   │
│   ├── hooks/                # 커스텀 훅
│   │   └── useMultifacetedRater.js # 평가자 관리
│   │
│   ├── assets/               # 정적 에셋
│   ├── App.tsx               # 메인 앱 (22KB)
│   ├── api-client.js         # API 통신
│   ├── main.tsx              # 진입점
│   └── index.css             # 글로벌 스타일
│
├── public/                   # 정적 파일
├── dist/                     # 빌드 결과
├── package.json
├── vite.config.ts
├── tsconfig.json
└── tsconfig.app.json
```

## 개발 명령어

```bash
cd apro_user

# 개발 서버
npm run dev

# 프로덕션 빌드
npm run build

# 클린 빌드
npm run build:clean

# 린트
npm run lint

# 미리보기
npm run preview
```

## API 클라이언트 (`src/api-client.js`)

### 서버 URL 결정

```javascript
function getServerUrl() {
  if (window.location.hostname === 'localhost' &&
      window.location.port === '3000') {
    return 'http://localhost/api';
  }
  return 'https://api.aprofiler.co.kr/api';
}
```

### 회사 코드 헤더

```javascript
function getCompanyCode() {
  if (window.location.hostname === 'localhost') {
    return 'hrdna2';  // 개발용 기본값
  }
  // 서브도메인에서 추출: company.aprofiler.co.kr -> company
  return window.location.hostname.split('.')[0];
}

// API 호출 시 자동 추가
headers['company'] = getCompanyCode();
```

### 에러 처리

```javascript
// 특수 에러 처리
if (errorMessage === '테스트 기간이 종료되었습니다') {
  window.location.href = '/end';
  return;
}

if (errorMessage === '인증 토큰이 없습니다') {
  // 검은 오버레이 표시
  showOverlay('잘못된 접근입니다');
  return;
}

if (errorMessage.includes('토큰이 유효하지 않거나 만료')) {
  clearTokens();
  window.location.href = '/login';
  return;
}
```

## 메인 앱 구조 (`src/App.tsx`)

### 인증 상태 관리

```typescript
type AuthStatus = 'loading' | 'authenticated' | 'unauthenticated';

function App() {
  const [authStatus, setAuthStatus] = useState<AuthStatus>('loading');
  const [applicantInfo, setApplicantInfo] = useState(null);

  useEffect(() => {
    const token = getAccessToken();
    if (!token) {
      setAuthStatus('unauthenticated');
      return;
    }

    // 토큰 유효성 검증
    apiGet('/applicant/info')
      .then(response => {
        setApplicantInfo(response.data);
        setAuthStatus('authenticated');
      })
      .catch(() => {
        clearTokens();
        setAuthStatus('unauthenticated');
      });
  }, []);

  if (authStatus === 'loading') return <LoadingSpinner />;
  if (authStatus === 'unauthenticated') return <LoginPage />;

  return <AppContent info={applicantInfo} />;
}
```

### 라우팅

```tsx
<Router>
  <Routes>
    <Route path="/privacy" element={<PrivacyPage />} />
    <Route path="/login" element={<LoginPage />} />
    <Route path="/" element={<AppContent />} />

    {/* 인성검사 */}
    <Route path="/personality/landing" element={<AppContent page="personality-landing" />} />
    <Route path="/personality/test" element={<AppContent page="personality-test" />} />
    <Route path="/personality/end" element={<AppContent page="personality-end" />} />

    {/* 다면평가 */}
    <Route path="/multifaceted/landing" element={<AppContent page="multifaceted-landing" />} />
    <Route path="/multifaceted/request" element={<AppContent page="multifaceted-request" />} />
    <Route path="/multifaceted/status" element={<AppContent page="multifaceted-status" />} />
    <Route path="/multifaceted/test" element={<AppContent page="multifaceted-test" />} />

    {/* 미리보기 */}
    <Route path="/preview/:pageNumber" element={<AppContent page="preview" />} />
  </Routes>
</Router>
```

## 주요 페이지

### 로그인 페이지 (`pages/login.jsx`)

```jsx
function LoginPage() {
  const [id, setId] = useState('');
  const [pw, setPw] = useState('');
  const [companyInfo, setCompanyInfo] = useState(null);

  useEffect(() => {
    // 기업 정보 조회 (로고, 이름)
    apiGet('/company-info').then(res => {
      setCompanyInfo(res.data);
    });
  }, []);

  const handleLogin = async () => {
    const response = await apiPost('/applicant/login', { id, pw });
    if (response.success) {
      setAccessToken(response.data.access_token);
      window.location.href = '/';
    }
  };

  return (
    <Box>
      <img src={companyInfo?.logo} alt="Company Logo" />
      <TextField label="아이디" value={id} onChange={e => setId(e.target.value)} />
      <TextField label="비밀번호" type="password" value={pw} onChange={e => setPw(e.target.value)} />
      <Button onClick={handleLogin}>로그인</Button>
    </Box>
  );
}
```

### 인성검사 랜딩 (`pages/landing.jsx`)

- 검사 안내 메시지 표시
- 응시 상태 (미진행/진행중/완료)
- 남은 시간 표시
- "시작" 버튼

### 인성검사 테스트 (`pages/test.jsx`)

```jsx
function TestPage({ info }) {
  const [questions, setQuestions] = useState([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [answers, setAnswers] = useState({});

  useEffect(() => {
    // 10개씩 문항 로드
    loadQuestions(currentIndex, 10);
  }, [currentIndex]);

  const loadQuestions = async (start, count) => {
    const response = await apiGet(`/applicant/questions-bulk?start=${start}&count=${count}`);
    setQuestions(response.data);
  };

  const handleAnswer = async (questionId, value) => {
    setAnswers(prev => ({ ...prev, [questionId]: value }));

    // 자동 저장
    await apiPost('/applicant/answer', {
      question_id: questionId,
      value
    });
  };

  const handleSubmit = async () => {
    await apiPost('/applicant/end');
    window.location.href = '/end';
  };

  return (
    <Box>
      <ProgressBar current={Object.keys(answers).length} total={info.total_questions} />
      <Timer endTime={info.end_date} />

      {questions.map(q => (
        <QuestionCard
          key={q.id}
          question={q}
          value={answers[q.id]}
          onChange={value => handleAnswer(q.id, value)}
        />
      ))}

      <Button onClick={handleSubmit}>제출</Button>
    </Box>
  );
}
```

## 평가자 관리 훅 (`hooks/useMultifacetedRater.js`)

```javascript
export function useMultifacetedRater(applicantInfo, type = 'multifaceted') {
  const [raters, setRaters] = useState([]);
  const [loading, setLoading] = useState(false);

  // 평가자 목록 조회
  const fetchRaters = async () => {
    const response = await apiPost(`/applicant/${type}-rater/list`);
    setRaters(response.data);
  };

  // 평가자 추가
  const addRater = async (raterData) => {
    await apiPost(`/applicant/${type}-rater`, raterData);
    await fetchRaters();
  };

  // 평가자 수정
  const updateRater = async (raterIdx, raterData) => {
    await apiPut(`/applicant/${type}-rater/${raterIdx}`, raterData);
    await fetchRaters();
  };

  // 평가자 삭제
  const deleteRater = async (raterIdx) => {
    await apiDelete(`/applicant/${type}-rater/${raterIdx}`);
    await fetchRaters();
  };

  // 이메일 발송
  const sendEmail = async (raterIdx) => {
    try {
      await apiPost(`/applicant/${type}-rater/${raterIdx}/send-email`);
      return { success: true };
    } catch (error) {
      if (error.status === 429) {
        return { success: false, message: '5분 이내에 재발송할 수 없습니다.' };
      }
      throw error;
    }
  };

  // 미완료 평가자 일괄 발송
  const sendEmailToIncomplete = async () => {
    const incomplete = raters.filter(r => !r.is_complete);
    const results = await Promise.all(
      incomplete.map(r => sendEmail(r.idx))
    );
    return results;
  };

  return {
    raters,
    loading,
    fetchRaters,
    addRater,
    updateRater,
    deleteRater,
    sendEmail,
    sendEmailToIncomplete,
  };
}
```

## 미리보기 모드

개발/테스트용 미리보기 기능:

```jsx
// URL: /preview/1, /preview/2, ...
function PreviewPage({ pageNumber }) {
  const mockInfo = {
    name: '테스트 응시자',
    applicant_number: 'TEST-001',
    // ...
  };

  switch (pageNumber) {
    case '1':
      return <MultifacetedLandingPage info={mockInfo} />;
    case '2':
      return <MultifacetedStatusPage info={mockInfo} />;
    // ...
  }
}
```

## 스타일링

### Material-UI 테마

```tsx
import { createTheme, ThemeProvider } from '@mui/material';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
  },
});

<ThemeProvider theme={theme}>
  <App />
</ThemeProvider>
```

### 글로벌 스타일 (`src/index.css`)

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Noto Sans KR', sans-serif;
}
```
