# 설치 가이드

## 요구 사항

### 시스템 요구 사항

- PHP 8.2 이상
- Composer 2.x
- Node.js 18.x 이상
- npm 9.x 이상

### PHP 확장

```
openssl, pdo, mbstring, tokenizer, xml, ctype, json, bcmath, gd
```

## 백엔드 설치 (htdocs)

### 1. 의존성 설치

```bash
cd htdocs

# PHP 의존성
composer install

# Node.js 의존성
npm ci
```

### 2. 환경 설정

```bash
# 환경 파일 복사
cp .env.example .env

# 애플리케이션 키 생성
php artisan key:generate
```

### 3. 데이터베이스 설정

```bash
# SQLite 파일 생성 (개발용)
touch database/database.sqlite

# 마이그레이션 실행
php artisan migrate

# 시드 데이터 생성
php artisan db:seed
```

### 4. 캐시 초기화

```bash
php artisan config:clear
php artisan route:clear
php artisan cache:clear
```

## 관리자 프론트엔드 설치 (apro)

```bash
cd apro

# 의존성 설치
npm install

# 환경 파일 설정
cp .env.example .env
```

## 응시자 프론트엔드 설치 (apro_user)

```bash
cd apro_user

# 의존성 설치
npm install
```

## 기본 계정

시드 데이터 생성 후 사용 가능한 계정:

| 이메일 | 비밀번호 | 역할 | 회사 |
|-------|---------|-----|-----|
| admin@example.com | password | 관리자 (level 3) | 테크솔루션 |
| manager@example.com | password | 팀장 (level 2) | 테크솔루션 |
| developer@example.com | password | 개발자 (level 1) | 디지털마케팅 |

## 트러블슈팅

### Composer 메모리 부족

```bash
COMPOSER_MEMORY_LIMIT=-1 composer install
```

### 권한 오류

```bash
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

### SQLite 연결 오류

```bash
# 파일 존재 확인
ls -la database/database.sqlite

# 권한 확인
chmod 664 database/database.sqlite
```
