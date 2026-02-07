# 개발 서버 실행

## 백엔드 (htdocs)

### 통합 개발 서버 (권장)

```bash
cd htdocs
composer run dev
```

이 명령은 동시에 실행:
- `php artisan serve` - Laravel 서버 (8000 포트)
- `php artisan queue:listen` - 큐 워커
- `php artisan pail` - 실시간 로그
- `npm run dev` - Vite 개발 서버

### 개별 실행

```bash
# Laravel 서버만
php artisan serve

# Vite 개발 서버만
npm run dev

# 큐 워커
php artisan queue:listen --tries=1
```

### SSR 모드

```bash
composer run dev:ssr
```

## 관리자 프론트엔드 (apro)

```bash
cd apro
npm start
```

기본 포트: `3000`

## 응시자 프론트엔드 (apro_user)

```bash
cd apro_user
npm run dev
```

기본 포트: `5173` (Vite)

## 테스트 실행

### PHP 테스트

```bash
cd htdocs

# 전체 테스트
composer run test

# 특정 테스트 파일
./vendor/bin/phpunit tests/Feature/Auth/AuthenticationTest.php

# 특정 테스트 메서드
./vendor/bin/phpunit --filter testLoginSuccess
```

### 프론트엔드 테스트

```bash
cd apro
npm test
```

## 코드 품질

### PHP 포맷팅 (Laravel Pint)

```bash
cd htdocs
./vendor/bin/pint
```

### JavaScript/TypeScript 포맷팅

```bash
cd htdocs
npm run format        # Prettier 적용
npm run format:check  # 검사만
npm run lint          # ESLint
npm run types         # TypeScript 타입 체크
```

## 빌드

### 프론트엔드 프로덕션 빌드

```bash
# htdocs (Inertia.js)
cd htdocs
npm run build
npm run build:ssr  # SSR 포함

# apro (관리자)
cd apro
npm run build

# apro_user (응시자)
cd apro_user
npm run build
npm run build:clean  # 클린 빌드
```

## 유용한 Artisan 명령어

```bash
# 라우트 목록
php artisan route:list

# 모델 정보
php artisan model:show User

# 마이그레이션 상태
php artisan migrate:status

# 캐시 초기화
php artisan optimize:clear

# Tinker (REPL)
php artisan tinker
```
