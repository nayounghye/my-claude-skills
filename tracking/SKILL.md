---
description: 새 프로젝트에 추적 코드(Meta Pixel, Google Ads/GA4, TikTok, Clarity, 당근, 네이버 등)를 자동 세팅합니다. "추적 코드 설치", "픽셀 세팅", "tracking setup" 등을 말할 때 사용.
argument-hint: [providers...]
disable-model-invocation: true
user-invocable: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /tracking — 추적 코드 자동 세팅

새 프로젝트에 광고/분석 추적 코드를 설치하는 스킬.

## 핵심 원칙 (절대 위반 금지)

**모든 추적 코드 ID(Meta Pixel, Google Ads gtag, GA4, Clarity, TikTok, 네이버, 당근 등)는 반드시 `<head>` 인라인 스크립트에 하드코딩한다.**

- 금지: `NEXT_PUBLIC_*` 환경변수 사용
- 금지: `process.env.*`로 Tracking ID 참조
- 필수: layout 파일의 `<head>`에 ID를 직접 문자열로 삽입

이유: DOM 로드 타이밍과 환경변수 hydration 타이밍이 달라서, 환경변수로 넣으면 추적 SDK가 초기화 시점에 ID를 인식하지 못함. 실제로 겪은 문제.

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

인자가 이미 전달됐으면 (`/tracking meta google clarity`) 질문 생략하고 바로 진행.

### 3단계: Tracking ID 수집

선택한 각 항목에 대해 ID를 질문:

- Meta Pixel → "Meta Pixel ID를 알려주세요 (예: 1234567890)"
- Google Ads → "Google Ads ID를 알려주세요 (예: AW-18196402617)" + "Conversion Label? (예: abcDEF123)"
- GA4 → Google Ads ID와 동일 (gtag 공유) 또는 별도 GA4 ID (예: G-XXXXXXXXXX)
- TikTok → "TikTok Pixel ID를 알려주세요"
- Clarity → "Clarity Project ID를 알려주세요 (예: x0k85x9hcm)"
- 당근 → "당근 Pixel ID를 알려주세요"
- Naver → "네이버 전환추적 ID를 알려주세요"

### 4단계: 파일 생성

아래 순서로 파일을 생성/수정한다.

#### 4-1. `<head>` 스크립트 주입 (프레임워크별)

선택한 추적 코드의 초기화 스크립트를 `<head>`에 하드코딩 삽입.
**반드시 ID를 문자열 리터럴로 직접 넣는다. 환경변수 참조 절대 금지.**

**Meta Pixel 스니펫:**
```html
<script dangerouslySetInnerHTML={{ __html: `
  !function(f,b,e,v,n,t,s){if(f.fbq)return;n=f.fbq=function(){n.callMethod?
  n.callMethod.apply(n,arguments):n.queue.push(arguments)};if(!f._fbq)f._fbq=n;
  n.push=n;n.loaded=!0;n.version='2.0';n.queue=[];t=b.createElement(e);t.async=!0;
  t.src=v;s=b.getElementsByTagName(e)[0];s.parentNode.insertBefore(t,s)}
  (window,document,'script','https://connect.facebook.net/en_US/fbevents.js');
  fbq('init','여기에_PIXEL_ID');
  fbq('track','PageView');
`}} />
```

**Google Ads / GA4 스니펫:**
```html
<script async src="https://www.googletagmanager.com/gtag/js?id=여기에_ADS_ID" />
<script dangerouslySetInnerHTML={{ __html: `
  window.dataLayer=window.dataLayer||[];
  function gtag(){dataLayer.push(arguments);}
  gtag('js',new Date());
  gtag('config','여기에_ADS_ID');
`}} />
```

**Clarity 스니펫:**
```html
<script dangerouslySetInnerHTML={{ __html: `
  (function(c,l,a,r,i,t,y){c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
  t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
  y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y)})
  (window,document,"clarity","script","여기에_CLARITY_ID");
`}} />
```

**TikTok Pixel 스니펫:**
```html
<script dangerouslySetInnerHTML={{ __html: `
  !function(w,d,t){w.TiktokAnalyticsObject=t;var ttq=w[t]=w[t]||[];
  ttq.methods=["page","track","identify","instances","debug","on","off","once","ready","alias","group",
  "enableCookie","disableCookie","holdConsent","revokeConsent","grantConsent"];
  ttq.setAndDefer=function(t,e){t[e]=function(){t.push([e].concat(Array.prototype.slice.call(arguments,0)))}};
  for(var i=0;i<ttq.methods.length;i++)ttq.setAndDefer(ttq,ttq.methods[i]);
  ttq.instance=function(t){for(var e=ttq._i[t]||[],n=0;n<ttq.methods.length;n++)ttq.setAndDefer(e,ttq.methods[n]);return e};
  ttq.load=function(e,n){var r="https://analytics.tiktok.com/i18n/pixel/events.js",
  o=n&&n.partner;ttq._i=ttq._i||{};ttq._i[e]=[];ttq._i[e]._u=r;ttq._t=ttq._t||{};
  ttq._t[e]=+new Date;ttq._o=ttq._o||{};ttq._o[e]=n||{};
  var s=d.createElement("script");s.type="text/javascript";s.async=!0;s.src=r+"?sdkid="+e+"&lib="+t;
  var a=d.getElementsByTagName("script")[0];a.parentNode.insertBefore(s,a)};
  ttq.load('여기에_TIKTOK_ID');ttq.page();
  }(window,document,'ttq');
`}} />
```

**당근 Pixel 스니펫:**
```html
<script dangerouslySetInnerHTML={{ __html: `
  (function(){var s=document.createElement('script');s.async=true;
  s.src='https://karrot-pixel.business.daangn.com/pixel/0.2/karrot-pixel.umd.js';
  s.onload=function(){window.karrotPixel&&(karrotPixel.init('여기에_KARROT_ID'),karrotPixel.track('ViewPage'))};
  document.head.appendChild(s)})();
`}} />
```

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
  ✓ Meta Pixel (ID: 1234567890)
  ✓ Google Ads (ID: AW-xxxxx)
  ✓ Clarity (ID: xxxxxx)

생성된 파일:
  - lib/analytics/index.ts
  - lib/analytics/facebook.ts
  - lib/analytics/google-ads.ts
  - ...

다음 할 일:
  □ AnalyticsEvent 타입에 프로젝트별 이벤트 추가
  □ 전환 이벤트(결제 성공 등)에 trackEvent() 호출 추가
```
