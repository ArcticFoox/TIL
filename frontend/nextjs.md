# Next.js 특징 & 장점
## 파일 기반 라우팅
React Router를 직접 설정할 필요가 없음
```bash
app/
 ├── page.js        → /
 ├── about/
 │    └── page.js   → /about
 └── blog/[id]/
      └── page.js   → /blog/123
```
- Lazy loading 자동 적용
- Layout 분리 쉬움
- 동적 라우팅 지원
  
## 다양한 렌더링(SSR, SSG, ISR, CSR) 지원
  
## Server Component 기반 구조
- 서버에서 호출하는 fetch는 불필요한 JS 번들 낭비 없음
- 클라이언트로 API 키 노출 위험 없음
- Suspense 기반 Streaming 지원
  
## 정적 파일 빌드/번들링 자동화
- code splitting 자동 적용
- tree shaking 자동 적용
- lazy loading 자동 적용
- dead code 제거
- CSS/JS 최적화

