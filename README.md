# APRO - 인성검사 관리 시스템

## 소개

APRO는 기업용 인성검사, 평판진단, 다면평가를 통합 관리하는 웹 기반 평가 시스템입니다.

### 주요 기능

- **인성검사 (Personality Test)**: MBTI, Big5 등 다양한 인성검사 온/오프라인 시행
- **평판진단 (Reputation/360-degree Feedback)**: 동료 평가 기반 평판 분석
- **다면평가 (Multifaceted Evaluation)**: 자기평가 + 타인평가 통합 분석
- **멀티테넌시**: 기업별 독립된 데이터 관리
- **대행사 구조**: 기업이 다른 기업을 대행하여 검사 관리

### 시스템 구성

| 구성요소 | 설명 | 기술 스택 |
|---------|------|----------|
| htdocs | 백엔드 API 서버 | Laravel 12, PHP 8.2+ |
| apro | 관리자 대시보드 | React 18, Material-UI, CRA |
| apro_user | 응시자 프론트엔드 | React 19, TypeScript, Vite |

### 운영 URL

- **관리자**: `https://admin.aprofiler.co.kr`
- **응시자**: `https://{회사코드}.aprofiler.co.kr`
- **API**: `https://api.aprofiler.co.kr`

## 빠른 시작

```bash
# 백엔드 설정
cd htdocs
composer install
npm ci
cp .env.example .env
php artisan key:generate
php artisan migrate --seed

# 개발 서버 시작
composer run dev
```

자세한 설치 가이드는 [시작하기](getting-started/installation.md)를 참조하세요.

## 문서 구조

1. **[시작하기](getting-started/)** - 설치 및 환경 설정
2. **[아키텍처](architecture/)** - 시스템 구조 이해
3. **[백엔드](backend/)** - Laravel API 개발 가이드
4. **[프론트엔드](frontend/)** - React 개발 가이드
5. **[데이터베이스](database/)** - 스키마 및 모델 관계
6. **[API 문서](api/)** - REST API 레퍼런스
