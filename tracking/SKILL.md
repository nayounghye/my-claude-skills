---
description: 새 프로젝트에 추적 코드(Meta Pixel, Google Ads/GA4, TikTok, Clarity, 당근, 네이버 등)를 자동 세팅합니다. "추적 코드 설치", "픽셀 세팅", "tracking setup" 등을 말할 때 사용.
argument-hint: [providers...]
disable-model-invocation: true
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /tracking — 추적 코드 자동 세팅

새 프로젝트에 광고/분석 추적 코드를 설치하는 스킬.
마케터로부터 받은 추적 코드 스니펫을 프레임워크에 맞게 `<head>`에 삽입한다.

## 핵심 원칙 (절대 위반 금지)

**모든 추적 코드는 반드시 `<head>` 인라인 스크립트에 하드코딩한다.**

- 금지: `NEXT_PUBLIC_*` 환경변수로 Tracking ID 관리
- 금지: `process.env.*`로 Tracking ID 참조
- 필수: layout 파일의 `<head>`에 코드를 직접 삽입

이유: DOM 로드 타이밍과 환경변수 hydration 타이밍이 달라서, 환경변수로 넣으면 추적 SDK가 초기화 시점에 ID를 인식하지 못함. 실제로 겪은 문제.

**스니펫은 직접 작성하지 않는다.** 각 플랫폼(Meta, Google, TikTok 등)의 공식 코드는 수시로 변경되므로, 반드시 마케터 또는 플랫폼 대시보드에서 받은 최신 코드를 사용한다.

---

## 실행 흐름

### 1단계: 프레임워크 감지

프로젝트 루트를 확인해서 프레임워크를 자동 감지한다.

| 감지 기준 | 프레임워크 | `<head>` 주입 방식 |
|-----------|-----------|-------------------|
| `next.config.*` 존재 | Next.js | `app/layout.tsx` → `dangerouslySetInnerHTML` |
| `nuxt.config.*` 존재 | Nuxt | `nuxt.config.ts` → `app.head.script` |
| `index.html` 존재 | 바닐라/Vite | `index.html` → `<script>` 직접 |

감지 실패 시 사용자에게 질문.

### 2단계: 설치할 추적 코드 선택

사용자에게 질문:

> 어떤 추적 코드를 설치할까요? (번호 또는 이름으로 선택, 쉼표 구분)
>
> 1. Meta Pixel (Facebook)
> 2. Google Ads + GA4 (gtag.js)
> 3. TikTok Pixel
> 4. Microsoft Clarity
> 5. 당근 (Karrot) Pixel
> 6. Naver 전환 추적
> 7. 기타 (직접 입력)

인자가 이미 전달됐으면 (`/tracking meta google clarity`) 질문 생략하고 바로 진행.

### 3단계: 추적 코드 수집

선택한 각 항목에 대해 사용자에게 코드를 요청:

> "{플랫폼명}" 추적 코드를 붙여넣어 주세요.
> (마케터분 또는 플랫폼 대시보드에서 받은 `<script>` 코드 전체)

코드를 받으면 프레임워크에 맞게 변환:
- **Next.js**: `<script>` → `<script dangerouslySetInnerHTML={{ __html: \`...\` }} />`
- **Nuxt**: `nuxt.config.ts`의 `app.head.script`에 추가
- **바닐라/Vite**: `index.html`의 `<head>`에 그대로 삽입

### 4단계: 파일 생성

#### 4-1. `<head>` 스크립트 주입

사용자에게 받은 코드를 프레임워크에 맞는 방식으로 layout 파일의 `<head>` 영역에 삽입.

#### 4-2. 이벤트 래퍼 파일 (`lib/analytics/`)

선택한 각 플랫폼에 대해 래퍼 파일을 생성한다.

각 래퍼는:
- `"use client"` 선언 (Next.js인 경우)
- `typeof window === "undefined"` 가드
- `window.fbq` / `window.gtag` 등 글로벌 존재 확인
- 프로젝트의 AnalyticsEvent 타입에 대한 switch 매핑

통합 허브 `lib/analytics/index.ts`도 생성:
- `trackEvent(event)` — 설치된 플랫폼 전체에 병렬 디스패치
- `AnalyticsEvent` discriminated union 타입 (기본 이벤트 + 프로젝트별 확장용 주석)

#### 4-3. CSP 도메인 추가

프로젝트에 middleware 또는 CSP 설정이 있으면, 선택한 플랫폼의 도메인을 화이트리스트에 추가:

| 플랫폼 | script-src | connect-src | img-src |
|--------|-----------|------------|---------|
| Meta | connect.facebook.net | *.facebook.com | *.facebook.com, *.fbcdn.net |
| Google | googletagmanager.com, *.google-analytics.com | *.google-analytics.com, analytics.google.com, googleads.g.doubleclick.net | *.google-analytics.com, www.googleadservices.com |
| TikTok | analytics.tiktok.com | *.tiktokv.com, analytics.tiktok.com | analytics.tiktok.com |
| Clarity | *.clarity.ms | *.clarity.ms, c.bing.com | *.clarity.ms |
| 당근 | karrot-pixel.business.daangn.com | *.daangn.com | - |

### 5단계: 완료 보고

설치 완료 후 아래 형태로 보고:

```
추적 코드 설치 완료

설치 항목:
  ✓ Meta Pixel
  ✓ Google Ads + GA4
  ✓ Clarity

생성/수정된 파일:
  - app/layout.tsx (<head> 스크립트 삽입)
  - lib/analytics/index.ts
  - lib/analytics/facebook.ts
  - lib/analytics/google-ads.ts
  - middleware.ts (CSP 도메인 추가)

다음 할 일:
  □ AnalyticsEvent 타입에 프로젝트별 이벤트 추가
  □ 전환 이벤트(결제 성공 등)에 trackEvent() 호출 추가
```
