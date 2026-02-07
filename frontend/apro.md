# APRO (관리자 대시보드)

## 개요

관리자용 React 웹 애플리케이션입니다. 인성검사, 다면진단, 평판진단 서비스를 통합 관리합니다.

- **위치**: `/apro`
- **기술 스택**: React 18, Material-UI 7, CRA (craco)
- **라우팅**: React Router v7 (lazy loading)
- **상태 관리**: Context API + 커스텀 Hooks
- **폼**: React Hook Form + Zod

## 개발 명령어

```bash
cd apro
npm install    # 의존성 설치
npm start      # 개발 서버 (http://localhost:3000)
npm run build  # 프로덕션 빌드
npm test       # 테스트 실행
```

## API 서버

| 환경 | Base URL |
|------|----------|
| 개발 (localhost) | `http://localhost/api` |
| 운영 | `https://api.aprofiler.co.kr/api` |

## 권한 체계

| 권한 | company_id | 접근 범위 |
|------|-----------|----------|
| 슈퍼 관리자 | `1` | 전체 기능 |
| 에이전시 | `2` | 작업 관리 + 업체/공지 관리 |
| 일반 기업 | 기타 | 작업 관리만 |

## 상세 문서

| 문서 | 설명 |
|------|------|
| [디렉토리 구조](apro/directory-structure.md) | 전체 소스 파일 트리 |
| [라우팅](apro/routing.md) | 전체 라우트 맵, 경로 상수, 가드 |
| [페이지 컴포넌트](apro/pages.md) | 49개 페이지 목록 및 역할 |
| [컴포넌트](apro/components.md) | 코어/레이아웃/비즈니스 컴포넌트 |
| [네비게이션](apro/navigation.md) | 메뉴 구조, 권한별 필터링 |
| [API 클라이언트](apro/api-client.md) | HTTP 클라이언트, 토큰 관리 |
| [인증 시스템](apro/authentication.md) | 로그인/로그아웃 흐름, Auth Guard |
| [상태 관리](apro/state-management.md) | Context, Hooks, 폼, 데이터 페칭 |
