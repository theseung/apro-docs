# 환경 설정

## 백엔드 환경 변수 (htdocs/.env)

### 필수 설정

```env
APP_NAME=APRO
APP_ENV=local
APP_KEY=base64:xxxxx
APP_DEBUG=true
APP_URL=http://localhost

# 데이터베이스 (MySQL)
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=apro
DB_USERNAME=apro
DB_PASSWORD=your-password

# 세션
SESSION_DRIVER=database
SESSION_LIFETIME=120

# 캐시
CACHE_STORE=database
```

### 메일 설정

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@aprofiler.co.kr
MAIL_FROM_NAME="APRO System"
```

### 외부 공개 API 키

```env
# 외부 시스템 연동용 API 키
PUBLIC_API_KEY=your-secure-api-key-here
```

### CORS 설정

`config/cors.php`에서 관리:

```php
'allowed_origins' => ['*'],  // 프로덕션에서는 특정 도메인만 허용
'allowed_methods' => ['*'],
'exposed_headers' => ['Content-Disposition'],
```

## 프론트엔드 환경 변수

### apro/.env

```env
REACT_APP_API_URL=http://localhost/api
```

### apro_user (자동 감지)

`api-client.js`에서 자동으로 API URL 결정:
- localhost:3000 → `http://localhost/api`
- 기타 → `https://api.aprofiler.co.kr/api`

## Sanctum 토큰 설정

`config/sanctum.php`:

```php
'expiration' => null,  // 토큰 만료 없음 (또는 분 단위 설정)

'token_prefix' => env('SANCTUM_TOKEN_PREFIX', ''),
```

## 로깅 설정

`config/logging.php`:

```php
'default' => env('LOG_CHANNEL', 'stack'),

'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['single'],
    ],
    'single' => [
        'driver' => 'single',
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],
],
```
