# 공통 컴포넌트

## Material-UI 기반 컴포넌트

두 프론트엔드 모두 Material-UI(MUI) 7.x를 사용합니다.

### 기본 컴포넌트

```jsx
import {
  // 레이아웃
  Box,
  Container,
  Grid,
  Stack,
  Paper,
  Card,
  CardContent,
  CardHeader,
  CardActions,

  // 입력
  TextField,
  Button,
  IconButton,
  Checkbox,
  Radio,
  RadioGroup,
  Select,
  MenuItem,
  Switch,
  Slider,

  // 표시
  Typography,
  Chip,
  Badge,
  Avatar,
  Tooltip,
  Alert,
  Snackbar,

  // 테이블
  Table,
  TableHead,
  TableBody,
  TableRow,
  TableCell,
  TableContainer,
  TablePagination,

  // 네비게이션
  Tabs,
  Tab,
  Breadcrumbs,
  Link,
  Menu,

  // 피드백
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  CircularProgress,
  LinearProgress,
  Skeleton,

} from '@mui/material';
```

### 아이콘

```jsx
import {
  Add,
  Edit,
  Delete,
  Search,
  Close,
  Check,
  ArrowBack,
  ArrowForward,
  Download,
  Upload,
  Settings,
  Person,
  Email,
} from '@mui/icons-material';
```

---

## APRO 커스텀 컴포넌트

### Toaster (알림)

```jsx
// src/components/core/toaster/index.jsx
import { Toaster } from 'sonner';

<Toaster
  position="top-right"
  toastOptions={{
    style: {
      background: '#333',
      color: '#fff',
    },
  }}
/>

// 사용
import { toast } from 'sonner';

toast.success('저장되었습니다.');
toast.error('오류가 발생했습니다.');
toast.info('정보 메시지');
```

### FileDropzone

```jsx
// src/components/core/file-dropzone/index.jsx
import { useDropzone } from 'react-dropzone';

function FileDropzone({ onDrop, accept }) {
  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept,
  });

  return (
    <Box {...getRootProps()}>
      <input {...getInputProps()} />
      {isDragActive ? (
        <Typography>파일을 여기에 놓으세요</Typography>
      ) : (
        <Typography>파일을 드래그하거나 클릭하여 업로드</Typography>
      )}
    </Box>
  );
}
```

### DataTable

```jsx
// 페이지네이션, 정렬, 선택 기능이 포함된 테이블
function DataTable({
  columns,
  rows,
  page,
  rowsPerPage,
  total,
  onPageChange,
  onRowsPerPageChange,
  onRowClick,
  selected,
  onSelect,
}) {
  return (
    <TableContainer component={Paper}>
      <Table>
        <TableHead>
          <TableRow>
            <TableCell padding="checkbox">
              <Checkbox
                checked={selected.length === rows.length}
                onChange={() => onSelect('all')}
              />
            </TableCell>
            {columns.map(col => (
              <TableCell key={col.field}>{col.headerName}</TableCell>
            ))}
          </TableRow>
        </TableHead>
        <TableBody>
          {rows.map(row => (
            <TableRow
              key={row.id}
              hover
              onClick={() => onRowClick?.(row)}
              selected={selected.includes(row.id)}
            >
              <TableCell padding="checkbox">
                <Checkbox
                  checked={selected.includes(row.id)}
                  onChange={() => onSelect(row.id)}
                />
              </TableCell>
              {columns.map(col => (
                <TableCell key={col.field}>
                  {col.render ? col.render(row) : row[col.field]}
                </TableCell>
              ))}
            </TableRow>
          ))}
        </TableBody>
      </Table>
      <TablePagination
        component="div"
        count={total}
        page={page}
        rowsPerPage={rowsPerPage}
        onPageChange={onPageChange}
        onRowsPerPageChange={onRowsPerPageChange}
      />
    </TableContainer>
  );
}
```

---

## APRO_USER 커스텀 컴포넌트

### AppTextField

```tsx
// src/components/AppTextField.tsx
import { TextField, TextFieldProps } from '@mui/material';

interface AppTextFieldProps extends TextFieldProps {
  errorMessage?: string;
}

export function AppTextField({
  label,
  error,
  errorMessage,
  ...props
}: AppTextFieldProps) {
  return (
    <TextField
      fullWidth
      label={label}
      error={error || !!errorMessage}
      helperText={errorMessage}
      variant="outlined"
      {...props}
    />
  );
}
```

### MailPreviewDialog

```jsx
// src/components/MailPreviewDialog.jsx
function MailPreviewDialog({ open, onClose, rater, onConfirm }) {
  return (
    <Dialog open={open} onClose={onClose} maxWidth="md" fullWidth>
      <DialogTitle>메일 미리보기</DialogTitle>
      <DialogContent>
        <Typography variant="subtitle1">
          받는 사람: {rater.email}
        </Typography>
        <Typography variant="subtitle2">
          제목: 평가 요청
        </Typography>
        <Box sx={{ mt: 2, p: 2, bgcolor: 'grey.100' }}>
          <Typography>
            안녕하세요, {rater.name}님.
            <br /><br />
            {rater.applicant_name}님에 대한 평가를 요청드립니다.
            <br /><br />
            아래 링크를 클릭하여 평가를 진행해 주세요.
          </Typography>
        </Box>
      </DialogContent>
      <DialogActions>
        <Button onClick={onClose}>취소</Button>
        <Button onClick={onConfirm} variant="contained">
          발송
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```

### QuestionCard (인성검사)

```jsx
function QuestionCard({ question, value, onChange }) {
  return (
    <Card sx={{ mb: 2 }}>
      <CardContent>
        <Typography variant="body1" gutterBottom>
          {question.question_number}. {question.content}
        </Typography>

        <RadioGroup
          row
          value={value}
          onChange={e => onChange(e.target.value)}
        >
          {[1, 2, 3, 4, 5].map(val => (
            <FormControlLabel
              key={val}
              value={val}
              control={<Radio />}
              label={val}
            />
          ))}
        </RadioGroup>
      </CardContent>
    </Card>
  );
}
```

### ProgressTimer

```jsx
function ProgressTimer({ startTime, endTime, onTimeout }) {
  const [remaining, setRemaining] = useState(null);

  useEffect(() => {
    const timer = setInterval(() => {
      const now = new Date();
      const end = new Date(endTime);
      const diff = end - now;

      if (diff <= 0) {
        onTimeout?.();
        clearInterval(timer);
        return;
      }

      setRemaining(diff);
    }, 1000);

    return () => clearInterval(timer);
  }, [endTime, onTimeout]);

  const formatTime = (ms) => {
    const hours = Math.floor(ms / 3600000);
    const minutes = Math.floor((ms % 3600000) / 60000);
    const seconds = Math.floor((ms % 60000) / 1000);
    return `${hours}:${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
  };

  return (
    <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
      <AccessTimeIcon />
      <Typography variant="h6">
        {remaining ? formatTime(remaining) : '--:--:--'}
      </Typography>
    </Box>
  );
}
```

---

## 공통 패턴

### 로딩 상태

```jsx
function LoadingState() {
  return (
    <Box sx={{
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      minHeight: 200
    }}>
      <CircularProgress />
    </Box>
  );
}
```

### 에러 상태

```jsx
function ErrorState({ message, onRetry }) {
  return (
    <Box sx={{ textAlign: 'center', py: 4 }}>
      <Typography color="error" gutterBottom>
        {message || '오류가 발생했습니다.'}
      </Typography>
      {onRetry && (
        <Button onClick={onRetry} variant="outlined">
          다시 시도
        </Button>
      )}
    </Box>
  );
}
```

### 빈 상태

```jsx
function EmptyState({ message, action }) {
  return (
    <Box sx={{ textAlign: 'center', py: 4 }}>
      <Typography color="text.secondary" gutterBottom>
        {message || '데이터가 없습니다.'}
      </Typography>
      {action}
    </Box>
  );
}
```

### 확인 다이얼로그

```jsx
function ConfirmDialog({ open, title, message, onConfirm, onCancel }) {
  return (
    <Dialog open={open} onClose={onCancel}>
      <DialogTitle>{title}</DialogTitle>
      <DialogContent>
        <Typography>{message}</Typography>
      </DialogContent>
      <DialogActions>
        <Button onClick={onCancel}>취소</Button>
        <Button onClick={onConfirm} variant="contained" color="primary">
          확인
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```
