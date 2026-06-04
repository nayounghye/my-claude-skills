---
name: code-review
description: "종합 코드 리뷰 (QA + 보안 + 최적화 + 코드품질). 파일 경로와 카테고리를 인자로 받아 심도 있는 리뷰를 수행합니다."
---

# 종합 코드 리뷰 스킬

사용자가 요청한 파일에 대해 다음 4가지 관점에서 종합적인 코드 리뷰를 수행합니다:

1. **버그/QA 체크** - 잠재적 버그, 타입 안정성, 에러 핸들링
2. **보안 체크** - XSS, SQL Injection, 권한 검증, 민감 정보 노출
3. **최적화** - 성능 병목, N+1 쿼리, 리렌더링, 번들 사이즈
4. **코드 품질** - DRY, SOLID, 네이밍, 일관성, 테스트 가능성

## 사용 방법

```
/code-review app/api/events/[id]/register/route.ts
/code-review components/features/events/EventCard.tsx --category=performance
```

## 실행 프롬프트

파일 경로: **{{file_path}}**
카테고리: **{{category}}** (기본값: all)

다음 체크리스트를 기반으로 코드를 분석하고, 발견된 문제를 심각도별로 분류하여 보고하세요:

---

## 📋 체크리스트

### 1️⃣ 버그/QA 체크

#### 타입 안정성
- [ ] `any` 타입 남용 여부
- [ ] 타입 단언(as) 남용 또는 잘못된 사용
- [ ] 옵셔널 체이닝 누락 (obj.prop vs obj?.prop)
- [ ] 타입 가드 누락 (Array.isArray, typeof 등)

#### Null/Undefined 안전성
- [ ] null/undefined 체크 누락
- [ ] `!!value` 대신 명시적 체크 권장
- [ ] Optional 파라미터의 기본값 누락

#### 에러 핸들링
- [ ] try-catch 블록 누락 (async 함수, 외부 API 호출)
- [ ] Promise rejection 미처리
- [ ] 에러 로깅 누락
- [ ] 사용자 친화적 에러 메시지 부재

#### 엣지 케이스
- [ ] 빈 배열/객체 처리
- [ ] 0, '', false 등 falsy 값 처리
- [ ] 경계값 테스트 (최대/최소값)
- [ ] 동시성 문제 (race condition)

#### API/데이터 검증
- [ ] API 응답 형식 검증 누락
- [ ] Zod/Yup 등 스키마 검증 누락
- [ ] 입력값 sanitization 누락

#### 로직 오류
- [ ] 조건문 논리 오류 (&&, ||, ! 사용 실수)
- [ ] 무한 루프 가능성
- [ ] 메모리 누수 (이벤트 리스너, 타이머 해제 누락)

---

### 2️⃣ 보안 체크

#### XSS (Cross-Site Scripting)
- [ ] dangerouslySetInnerHTML 사용 시 sanitization 누락
- [ ] 사용자 입력을 직접 HTML에 삽입
- [ ] URL 파라미터를 검증 없이 사용

#### SQL Injection
- [ ] Raw SQL 쿼리에 문자열 연결 사용
- [ ] ORM의 where 조건에 사용자 입력 직접 삽입

#### 인증/권한 검증
- [ ] API 라우트에서 세션 확인 누락
- [ ] 본인 확인 로직 누락 (userId 비교)
- [ ] 권한 레벨 체크 누락 (admin, user)

#### 민감 정보 노출
- [ ] 환경변수 대신 하드코딩된 비밀키
- [ ] 에러 메시지에 내부 정보 노출 (스택 트레이스, DB 쿼리)
- [ ] 로그에 비밀번호, 토큰 등 출력

#### 환경변수 관리
- [ ] process.env 값 검증 누락
- [ ] 프론트엔드에서 서버 전용 env 사용

#### Rate Limiting
- [ ] 로그인, 회원가입 등 민감한 엔드포인트에 rate limit 미적용
- [ ] SMS/이메일 발송 무제한 허용

#### Input Validation
- [ ] 파일 업로드 시 확장자/크기 검증 누락
- [ ] 이메일, 전화번호 등 형식 검증 누락

---

### 3️⃣ 최적화

#### React 리렌더링
- [ ] 불필요한 useState 사용 (useRef로 대체 가능)
- [ ] useCallback/useMemo 누락 (props로 전달되는 함수/객체)
- [ ] key prop 누락 또는 index 사용
- [ ] 거대한 컴포넌트 (200줄 이상) → 분할 권장

#### N+1 쿼리
- [ ] 반복문 안에서 DB 조회
- [ ] Airtable batch 조회 가능한데 개별 조회

#### API 호출 최적화
- [ ] 병렬 처리 가능한데 순차 호출
- [ ] 불필요한 중복 API 호출
- [ ] 캐싱 가능한데 매번 fetch
- [ ] **대규모 트래픽 문제**: 1000명이 접속하면 1000번 API 호출 (CSR 문제)
  - SSR/SSG/ISR로 서버에서 미리 렌더링 가능한 데이터인가?
  - SWR/React Query 등 캐싱 라이브러리 사용 검토
  - HTTP Cache-Control 헤더 설정 누락
- [ ] **실시간성 검토**: 모든 데이터를 실시간으로 가져올 필요가 있는가?
  - 정적 데이터(이벤트 목록, 공지사항): SSG/ISR 사용
  - 사용자별 데이터(마이페이지): SSR 또는 클라이언트 캐싱
  - 진짜 실시간 필요한 데이터만(결제 상태, 채팅): 클라이언트 폴링/웹소켓

#### 번들 사이즈
- [ ] 전체 라이브러리 import (lodash 전체 vs lodash/get)
- [ ] 사용하지 않는 의존성
- [ ] dynamic import로 code splitting 가능

#### 이미지 최적화
- [ ] next/image 대신 <img> 사용
- [ ] 적절한 width/height prop 미지정
- [ ] WebP/AVIF 등 최신 포맷 미사용

#### DB 쿼리 최적화
- [ ] 인덱스 없는 필드로 조회
- [ ] SELECT * 사용 (필요한 필드만 조회)
- [ ] 페이지네이션 누락 (전체 데이터 로드)

#### 병렬 처리
- [ ] Promise.all 사용 가능한데 순차 await
- [ ] 독립적인 작업을 동기적으로 실행

---

### 4️⃣ 코드 품질

#### DRY (Don't Repeat Yourself)
- [ ] 중복 코드 (3회 이상 반복 → 함수/컴포넌트 추출)
- [ ] 유사한 함수들 (공통 로직 추출 가능)
- [ ] **동일 동작, 다른 함수명**: 같은 기능을 하는데 함수명만 달라서 중복 존재
  - 예: `getUserData()`, `fetchUserInfo()`, `loadUserDetails()` 모두 같은 작업
  - 하나로 통일하고 나머지 제거 필요

#### SOLID 원칙
- [ ] 단일 책임 원칙 위반 (하나의 함수가 너무 많은 일)
- [ ] 의존성 역전 위반 (구체적인 구현에 의존)

#### 네이밍
- [ ] 의미 불명확한 변수명 (a, tmp, data)
- [ ] 함수명이 동작을 설명하지 않음
- [ ] Boolean 변수가 is/has/should로 시작하지 않음
- [ ] **camelCase 명명규칙 위반**: 프로젝트 표준은 camelCase
  - 변수/함수: `userName`, `fetchData()` (camelCase)
  - 컴포넌트/클래스: `UserProfile`, `EventCard` (PascalCase)
  - 상수: `MAX_RETRY_COUNT` (SCREAMING_SNAKE_CASE)
  - 파일명: 컴포넌트는 PascalCase, 나머지는 camelCase 또는 kebab-case 일관성 유지

#### 불필요한 코드
- [ ] 주석 처리된 코드 (Git에 기록 있으므로 삭제)
- [ ] 사용하지 않는 import
- [ ] TODO 주석만 있고 구현 없음

#### 일관성
- [ ] 인용부호 혼용 (' vs ")
- [ ] 들여쓰기 불일치
- [ ] 파일명 규칙 불일치 (camelCase vs kebab-case)

#### 테스트 가능성
- [ ] 테스트하기 어려운 구조 (전역 상태 의존, 하드코딩된 값)
- [ ] 순수 함수로 분리 가능한데 side effect 포함

#### 사용자 경험
- [ ] 에러 메시지가 개발자 용어 (사용자 친화적으로 수정)
- [ ] 로딩 상태 표시 누락
- [ ] 성공/실패 피드백 누락

#### 접근성
- [ ] img alt 속성 누락
- [ ] button type 미지정 (기본값 submit)
- [ ] label for 속성 누락

---

## 📊 출력 형식

리뷰 결과를 다음 형식으로 보고하세요:

```markdown
## 코드 리뷰 요약
**파일**: [파일경로](파일경로)
**평가**: ⚠️ 개선 필요 | ✅ 양호 | 🔴 심각한 문제
**발견**: Critical: X, Warning: Y, Suggestion: Z

### 주요 발견 (Top 3)
1. 🔴 [카테고리] 문제 요약 (line X)
2. 🟡 [카테고리] 문제 요약 (line Y)
3. 🟢 [카테고리] 문제 요약 (line Z)

---

## 상세 리뷰

### 🔴 Critical (반드시 수정 필요)

#### [파일명:줄번호](파일명#줄번호) - 문제 제목
**카테고리**: 보안 / QA / 최적화 / 코드품질
**문제**:
구체적인 문제 설명

**현재 코드**:
```typescript
// 문제가 있는 코드
```

**권장 수정**:
```typescript
// 수정된 코드
```

**이유**: 왜 이렇게 수정해야 하는지 설명

---

### 🟡 Warning (가능하면 수정 권장)

(위와 동일한 형식)

---

### 🟢 Suggestion (개선 제안)

(위와 동일한 형식)

---

## ✅ 좋은 점

- 코드에서 잘 작성된 부분 언급
- 모범 사례 사용 예시

---

## 📝 종합 의견

전체적인 코드 품질 평가 및 우선순위 제안
```

## 심각도 분류 기준

- **🔴 Critical**: 보안 취약점, 런타임 에러 가능성, 데이터 손실 위험
- **🟡 Warning**: 성능 저하, 유지보수성 문제, 잠재적 버그
- **🟢 Suggestion**: 코드 품질 개선, 가독성, 일관성

---

## 참고사항

- 프로젝트 아키텍처: Next.js 14 App Router, Airtable DB, NextAuth v4
- 백엔드: `lib/backend/`, 프론트엔드: `lib/frontend/`, `components/`
- API 응답 형식: `{ success: boolean, data?: any, error?: string, code?: string }`
- 타입 위치: `types/index.ts`

파일을 읽고 위 체크리스트를 기반으로 심도 있는 리뷰를 수행하세요.
