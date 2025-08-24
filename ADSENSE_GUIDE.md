# Google AdSense 설정 완료!

## ✅ 현재 설정 상태

Publisher ID가 환경변수에 설정되었습니다:
- Publisher ID: `ca-pub-3651717576639643`

## 📝 남은 작업

### 1. Google AdSense 대시보드에서 광고 유닛 생성

1. [Google AdSense](https://www.google.com/adsense/) 로그인
2. 왼쪽 메뉴에서 **광고** → **광고 단위** 클릭
3. **새 광고 단위 만들기** 클릭
4. 다음 광고 유닛들을 생성:

#### 포스트 상단 광고
- 이름: `ssadakk_post_top`
- 광고 유형: 디스플레이 광고
- 광고 크기: 반응형 (권장)

#### 포스트 하단 광고
- 이름: `ssadakk_post_bottom`
- 광고 유형: 디스플레이 광고
- 광고 크기: 반응형 (권장)

#### 메인 페이지 광고 (선택사항)
- 이름: `ssadakk_homepage`
- 광고 유형: 디스플레이 광고
- 광고 크기: 반응형 (권장)

### 2. 광고 슬롯 ID 업데이트

각 광고 유닛을 생성하면 `data-ad-slot="숫자"` 형태의 슬롯 ID를 받게 됩니다.

`src/layouts/PostDetails.astro` 파일에서 슬롯 ID를 실제 값으로 변경:

```astro
<!-- 포스트 상단 광고 -->
<GoogleAdsense 
  slot="실제_상단_슬롯_ID"  <!-- 예: "1234567890" -->
  format="auto"
/>

<!-- 포스트 하단 광고 -->
<GoogleAdsense 
  slot="실제_하단_슬롯_ID"  <!-- 예: "0987654321" -->
  format="auto"
/>
```

### 3. 메인 페이지에 광고 추가 (선택사항)

`src/pages/index.astro`에 광고 추가:

```astro
---
import GoogleAdsense from "@/components/GoogleAdsense.astro";
---

<!-- 포스트 목록 사이나 원하는 위치에 추가 -->
<GoogleAdsense slot="메인페이지_슬롯_ID" format="auto" />
```

## 🚀 테스트 방법

1. 로컬 테스트:
```bash
npm run dev
```

2. 빌드 및 프리뷰:
```bash
npm run build
npm run preview
```

## ⚠️ 주의사항

### 광고가 표시되지 않을 수 있는 경우:
1. **새 사이트**: AdSense 승인이 아직 완료되지 않음
2. **트래픽 부족**: 일정 트래픽이 있어야 광고 표시
3. **광고 차단**: 브라우저 광고 차단 확장 프로그램 확인
4. **개발 환경**: localhost에서는 빈 광고 공간만 표시될 수 있음

### AdSense 정책:
- 자체 클릭 금지
- 클릭 유도 문구 사용 금지
- 콘텐츠와 광고 명확히 구분
- 성인 콘텐츠, 폭력적 콘텐츠 금지

## 📊 수익 최적화 팁

1. **광고 위치**: 
   - 첫 화면에 보이는 위치 (Above the fold)
   - 콘텐츠 중간 (In-content)
   - 사이드바 (데스크톱)

2. **콘텐츠 품질**:
   - 고품질 콘텐츠 작성
   - SEO 최적화
   - 정기적인 포스팅

3. **사용자 경험**:
   - 광고 과다 배치 금지
   - 모바일 최적화
   - 빠른 로딩 속도 유지