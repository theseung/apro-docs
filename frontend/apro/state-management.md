# 상태 관리

APRO는 별도의 전역 상태 라이브러리(Redux 등)를 사용하지 않으며, **React Context API + 커스텀 Hooks**로 상태를 관리합니다.

---

## Context 목록

| Context | 위치 | 역할 |
|---------|------|------|
| `AuthContext` | `components/auth/custom/auth-context.jsx` | 인증 상태 (user, isAuthenticated) |
| `SettingsContext` | `components/core/settings/settings-context.jsx` | 테마 설정 (색상, 레이아웃, 방향) |
| i18n Context | `components/core/i18n-provider.jsx` | 다국어 (react-i18next) |

---

## 커스텀 Hooks (`src/hooks/`)

| Hook | 파일 | 용도 |
|------|------|------|
| `useAllTests` | `use-all-tests.js` | 전체 검사 데이터 fetch (대시보드 메인용) |
| `useDashboard` | `use-dashboard.js` | 대시보드 관련 데이터 |
| `useDialog` | `use-dialog.js` | 다이얼로그 open/close 상태 |
| `useMediaQuery` | `use-media-query.js` | 반응형 브레이크포인트 감지 |
| `usePathname` | `use-pathname.js` | 현재 라우트 pathname |
| `usePopover` | `use-popover.js` | 팝오버 anchor/open 상태 |
| `useSelection` | `use-selection.js` | 테이블 행 체크박스 선택 상태 |

### 사용 예시

```jsx
// 다이얼로그
const dialog = useDialog();
<Button onClick={dialog.handleOpen}>열기</Button>
<Dialog open={dialog.open} onClose={dialog.handleClose}>...</Dialog>

// 팝오버
const popover = usePopover();
<Button ref={popover.anchorRef} onClick={popover.handleOpen}>메뉴</Button>
<Popover anchorEl={popover.anchorRef.current} open={popover.open}>...</Popover>

// 테이블 선택
const selection = useSelection(rows.map(r => r.id));
<Checkbox
  checked={selection.selected.has(row.id)}
  onChange={() => selection.handleSelect(row.id)}
/>
```

---

## 폼 상태 관리

폼은 **React Hook Form + Zod**로 관리합니다.

```jsx
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().min(1, '필수 입력').email('이메일 형식 오류'),
  password: z.string().min(1, '필수 입력'),
});

function LoginForm() {
  const { control, handleSubmit, formState: { errors } } = useForm({
    defaultValues: { email: '', password: '' },
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data) => { /* API 호출 */ };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="email"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            error={!!errors.email}
            helperText={errors.email?.message}
          />
        )}
      />
    </form>
  );
}
```

---

## 로컬 스토리지

| 키 | 용도 | 관리 파일 |
|-----|------|----------|
| `access_token` | API 인증 토큰 | `lib/api-client.js` |
| `refresh_token` | 토큰 갱신용 | `lib/api-client.js` |
| `settings` | 테마/레이아웃 설정 | `lib/settings.js` |

---

## 데이터 페칭 패턴

별도의 데이터 페칭 라이브러리(React Query 등)를 사용하지 않으며, `useEffect` + `useState`로 직접 관리합니다.

```jsx
function PersonalityList() {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchData = async () => {
      const response = await apiGet('/personality');
      if (response.success) {
        setData(response.data);
      }
      setLoading(false);
    };
    fetchData();
  }, []);

  if (loading) return <CircularProgress />;
  return <TestListTable rows={data} />;
}
```
