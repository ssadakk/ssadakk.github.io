# Google AdSense 설정 가이드

## 1. Google AdSense 계정 준비

1. [Google AdSense](https://www.google.com/adsense/) 에서 계정 신청
2. 사이트 심사 대기 (보통 1-14일 소요)
3. 승인 후 광고 코드 받기

## 2. 환경변수 설정

`.env` 파일을 생성하고 AdSense Publisher ID를 추가:

```bash
# .env 파일 생성
cp .env.example .env

# Publisher ID 추가 (ca-pub-로 시작하는 ID)
PUBLIC_GOOGLE_ADSENSE_ID=ca-pub-XXXXXXXXXX
```

## 3. 광고 슬롯 ID 설정

`src/layouts/PostDetails.astro` 파일에서 광고 슬롯 ID를 실제 값으로 변경:

```astro
<!-- 포스트 상단 광고 -->
<GoogleAdsense 
  slot="실제_상단_광고_슬롯_ID" 
  format="auto"
/>

<!-- 포스트 하단 광고 -->
<GoogleAdsense 
  slot="실제_하단_광고_슬롯_ID" 
  format="auto"
/>
```

## 4. 광고 위치 커스터마이징

### 메인 페이지에 광고 추가
`src/pages/index.astro`에 광고 컴포넌트 import 후 원하는 위치에 추가:

```astro
---
import GoogleAdsense from "@/components/GoogleAdsense.astro";
---

<!-- 원하는 위치에 광고 추가 -->
<GoogleAdsense slot="메인페이지_광고_슬롯_ID" format="auto" />
```

### 광고 포맷 옵션
- `auto`: 자동 크기 조정 (권장)
- `fluid`: 유동적 크기
- `rectangle`: 직사각형
- `vertical`: 세로형
- `horizontal`: 가로형

## 5. 광고 정책 준수 사항

⚠️ **중요**: Google AdSense 정책을 반드시 준수해야 합니다:

- 한 페이지당 광고 개수 제한 준수
- 콘텐츠와 광고 구분 명확히 하기
- 클릭 유도 문구 사용 금지
- 자체 클릭 금지
- 유효하지 않은 트래픽 생성 금지

## 6. 테스트 및 확인

1. 로컬에서 테스트:
```bash
npm run dev
```

2. 빌드 후 확인:
```bash
npm run build
npm run preview
```

3. 배포 후 AdSense 대시보드에서 광고 노출 확인

## 7. 문제 해결

### 광고가 표시되지 않는 경우:
- AdSense 계정이 승인되었는지 확인
- 환경변수가 올바르게 설정되었는지 확인
- 광고 슬롯 ID가 올바른지 확인
- 사이트가 AdSense 정책을 준수하는지 확인
- 광고 차단 프로그램 비활성화 후 테스트

### 수익이 발생하지 않는 경우:
- 트래픽이 충분한지 확인
- 광고 위치가 적절한지 검토
- 콘텐츠 품질 개선
- SEO 최적화로 방문자 증가