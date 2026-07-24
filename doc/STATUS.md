# 개발 현황 (STATUS)

> 기준일: 2026-07-12 — 이 문서는 항상 "지금 시점" 기준으로 덮어써서 갱신합니다(과거 이력은 append하지 않습니다). 날짜별 히스토리는 [DEVLOG.md](DEVLOG.md) 참고.

## 프로젝트 개요

KBTE(한국방송기술교육원, 이후 **KOBETA**로 명칭 통일)의 교육 운영 업무를 지원하는 사내 웹 도구입니다. 기획 단계에서 두 개의 PRD가 나왔고, 실제 구현은 그중 하나에 집중되어 있습니다.

- **교육 커리큘럼 설계 플랫폼** ([`docs/spec/PRD_curriculum.md`](spec/PRD_curriculum.md)) — 실제로 구현이 진행된 축. 교육분야/강사/운영담당자 마스터데이터, 과정 카드 Pool, 드래그앤드롭 커리큘럼 빌더가 핵심.
- **쇼츠 제작 플랫폼** ([`docs/spec/PRD.md`](spec/PRD.md)) — 데이터 모델(`ShortsProject`, `ShortsTemplate`, `ShortsClip` 등)과 화면 뼈대(`/shorts/projects`, `/shorts/templates`)만 만들어졌고, PRD의 핵심인 AI 분석·서버사이드 렌더링 파이프라인은 아직 없음.

기술 스택: Next.js(App Router) + Prisma 7 + Neon Postgres(서버리스) + Vercel Blob(파일 저장) + Vercel 배포. 자세한 데이터 저장 구조는 `app/README.md` 참고.

## ✅ 완료 (Phase 1 범위)

- 마스터데이터 관리: 교육분야 카테고리(`/admin/categories`), 강사(`/admin/instructors`), 운영담당자(`/admin/coordinators`)
- 교육과정 Pool(`/pool`) + 드래그앤드롭 커리큘럼 빌더(`/builder`) — 커스텀 슬롯 생성, 시간 편집, 강사 다중 배정, 충돌 감지 일부, 인쇄(A4) 출력, 즐겨찾기
- 트렌드 수집/분석 대시보드(`/booklet/trends`) — 유튜브 채널 분석, 미디어투데이 크롤링, Gemini AI 연동
- 메모/일지(`MemoBoard`) — 클립보드 이미지 붙여넣기
- 오늘의 할 일 — 루틴/오늘 항목 구분, 5초 실행취소, 홈 요약 카드, PWA 설치 지원
- 데이터 계층을 Neon Postgres + Vercel Blob으로 전환 완료 (Vercel 배포 대응)
- 문서 체계 정비 — 로컬 `docs/`(spec/data/STATUS/DEVLOG) + 공개 문서 사이트([kobeta-docs](https://peter-cho-70.github.io/kobeta-docs/)) 구축

## 🚧 진행 중 / 미착수

- **AI 자동 커리큘럼 초안 생성**(PRD_curriculum 4.4) — 미착수
- **강사료 등급/예산 자동산정, KPI 관리, 사업(Program) 계층, 사업계획서 자동 생성**(PRD_curriculum Phase 2~4) — 미착수. 관련 Prisma 모델(`FeeGradeTable`, `BudgetLine`, `Program`, `KPIIndicator` 등)이 현재 스키마에 없음
- **쇼츠 플랫폼**(PRD.md) — 데이터 모델·화면 뼈대만 존재, AI 하이라이트 추천/스마트 리프레임/STT 자막/ffmpeg 렌더링 파이프라인 미구현

## 🐞 알려진 이슈

- **로컬 개발 = 운영 데이터 공유**: 로컬 `npm run dev`도 배포 환경과 동일한 Neon Postgres/Vercel Blob을 사용합니다(`app/README.md` 명시). 로컬 실험이 실 운영 데이터에 바로 반영되므로, 테스트용 Neon 브랜치 분리가 필요합니다.
- **자동화 테스트 부재**: `package.json`에 테스트 스크립트/디렉토리가 없음. 지금까지는 Playwright MCP로 수동 브라우저 확인만 해온 것으로 보임.
- **개인정보 처리 원칙 점검 필요**: PRD_curriculum.md 12장이 "기관현황/인력현황에 주민등록번호 등 식별번호를 저장하지 않는다"는 원칙을 명시. 강사(`Instructor`) 등 개인정보를 다루는 기능을 늘릴 때 이 원칙 준수 여부를 주기적으로 점검해야 함.

## 🗺️ 다음 우선순위

1. **AI 자동 커리큘럼 초안 생성** — 트렌드 분석에 이미 있는 Gemini 연동 경험 재사용 가능
2. **강사 일정 안내 이메일 자동화 + 예산/강사료 자동산정** — 강사료 등급표(`FeeGradeTable`) 마스터부터 필요
3. **교육 안내문(공문) PDF 자동 생성** — `docs/data/`의 2022년 보고서·안내서 포맷이 템플릿 참고 자료
4. **사업(Program) 계층 + KPI 관리 + 사업계획서 자동 생성** — 범위가 크므로 별도 설계 필요
5. **쇼츠 플랫폼 재개 여부 결정** — 계속 개발할지, 커리큘럼 플랫폼에 집중할지 우선순위 결정
6. **테스트 체계 도입** — API 라우트(카테고리/강사/빌더 저장) 스모크 테스트부터
7. **로컬/운영 데이터 분리** — Neon 브랜치 기능 활용
