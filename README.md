# KOBETA 교육 커리큘럼 플랫폼 — 문서 사이트

> 한국방송기술교육원(KBTE → **KOBETA**)의 교육 운영 업무를 지원하는 사내 웹 도구. 교육분야/강사/운영담당자 마스터데이터, 교육과정 Pool, 드래그앤드롭 커리큘럼 빌더를 중심으로 트렌드 분석 대시보드·메모(일지)·오늘의 할 일 기능까지 갖춘 올인원 관리 도구로 확장되고 있습니다.

브라우저에서 보기 좋은 버전: **[doc/index.html](doc/index.html)** — 문서를 클릭하면 원본 마크다운을 제목·표·코드블록이 정리된 화면(`doc/viewer.html`)으로 바로 보여줍니다.

---

## 📚 상세 문서

문서 전체 인덱스는 **[doc/index.html](doc/index.html)** 참고. 주요 문서:

- [개발 현황 (STATUS)](doc/STATUS.md) — 완료/진행중/알려진 이슈/다음 우선순위, 항상 최신 기준으로 갱신
- [개발 기록 (DEVLOG)](doc/DEVLOG.md) — 날짜별 개발 타임라인, 수정/버그 이력
- [교육 커리큘럼 설계 플랫폼 PRD](doc/spec/PRD_curriculum.md) — 실제 구현이 진행된 축. 마스터데이터·Pool·빌더·예산/KPI/사업계획서 관리 설계
- [쇼츠 제작 플랫폼 PRD](doc/spec/PRD.md) — 데이터 모델과 화면 뼈대만 구현된 상태 (AI 분석·렌더링 파이프라인은 미구현)
- [데이터 저장 구조](doc/architecture.md) — Neon Postgres + Vercel Blob 아키텍처
- [교육과정 콘텐츠 자료](doc/data/) — 2022년 결과보고서 분석, 2020~2026 전체 과정 통합 Pool, 지역사 특화 프로그램 기획

---

## ✨ 핵심 기능 (구현됨)

- **마스터데이터 관리**: 교육분야 카테고리, 강사, 운영담당자 (`/admin/*`)
- **교육과정 Pool + 드래그앤드롭 커리큘럼 빌더** (`/pool`, `/builder`): 커스텀 슬롯 생성, 시간 편집, 강사 다중 배정, 충돌 감지 일부, 인쇄(A4) 출력, 즐겨찾기
- **트렌드 수집/분석 대시보드** (`/booklet/trends`): 유튜브 채널 분석, 미디어투데이 크롤링, Gemini AI 연동 분석
- **메모/일지** (`MemoBoard`): 클립보드 이미지 붙여넣기 지원
- **오늘의 할 일**: 루틴/오늘 항목 구분, 5초 실행취소, 홈 요약 카드, PWA 설치 지원

기술 스택: Next.js(App Router) + Prisma 7 + Neon Postgres(서버리스) + Vercel Blob + Vercel 배포.

자세한 현재 상태는 **[doc/STATUS.md](doc/STATUS.md)**, 개발 타임라인·버그 수정 이력은 **[doc/DEVLOG.md](doc/DEVLOG.md)** 참고.

---

## 🔗 관련 저장소

- 앱 소스코드: `peter-cho-70/my_education` (private)
- 문서 허브: [peter-cho-70.github.io](https://peter-cho-70.github.io/)
