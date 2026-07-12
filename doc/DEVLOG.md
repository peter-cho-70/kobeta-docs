# 개발 기록 (DEVLOG)

> 날짜별로 새 섹션을 계속 append합니다(과거 항목은 고치지 않음 — 현재 상태 요약은 [STATUS.md](STATUS.md) 참고). git 이력(`app/`, 커밋 8개)과 기획 문서를 근거로 2026-07-12에 최초 작성했습니다.

## 2026-06-19 — 프로젝트 시작 (Create Next App 스캐폴드)

- **추가/변경**: `npx create-next-app`으로 초기 스캐폴드 생성 (`7af1737`).

## 2026-06-19 ~ 2026-06-22 — 전체 기능 최초 구현

- **추가/변경**: 스캐폴드 이후 며칠간 로컬에서 개발한 내용을 한 번에 커밋 (`a584bf7`, 157개 파일, +18,989줄). 세부 커밋 없이 통짜로 들어간 첫 대형 커밋이라, 이 구간 내부의 세부 이력은 git에는 남아있지 않음(단, `.playwright-mcp/`에 있던 브라우저 테스트 로그로 06-19~06-22 사이 반복적으로 화면을 띄워보며 개발한 흔적은 있었음 — 이후 정리 과정에서 삭제).
  - 마스터데이터 관리: 교육분야 카테고리(`/admin/categories`), 강사(`/admin/instructors`), 운영담당자(`/admin/coordinators`)
  - 교육과정 Pool(`/pool`) + 드래그앤드롭 커리큘럼 빌더(`/builder`, `/builder/[id]`, `/builder/templates`)
  - 트렌드 수집/분석 대시보드(`/booklet/trends`, `/booklet/trends/youtube`) — 유튜브 채널 분석, 미디어투데이 크롤링, Gemini 기반 AI 분석(`src/lib/gemini.ts`)
  - 쇼츠 제작 플랫폼 뼈대(`/shorts/projects`, `/shorts/templates`) — PRD.md의 데이터 모델만 구현, STT/렌더링 등 핵심 로직은 없음
  - 설정 화면(`/settings`): 상단 메뉴 카테고리·색상 관리
  - 메모/일지 기능(`MemoBoard`): 사진 붙여넣기(Ctrl+V) 지원
- **2026-06-22 11:53** — 그 시점까지 로컬에 쌓인 실 데이터(SQLite `dev.db`, 업로드 사진)를 임시로 git에 백업 (`759e3d6`).
- **2026-06-22 12:45** — Vercel 배포를 위해 데이터 계층 전환 (`6a6430a`): 로컬 SQLite/로컬 파일시스템은 서버리스 환경에서 휘발되므로, **Neon Postgres**(Prisma 7 + `@prisma/adapter-neon`)와 **Vercel Blob**으로 이전. 기존 데이터는 일회성 스크립트로 마이그레이션. SQLite 이력은 `app/.sqlite-migrations-backup/`에 참고용으로만 보관.

## 2026-06-30 — 커리큘럼 빌더 집중 개선 (하루 3커밋)

- **추가/변경 (00:10, `64242f0`)**: 빌더 기능 대폭 확장 — 커스텀 슬롯 생성, 강사 다중 태그 입력, 슬롯 전체 필드 수정, Day 삭제, 참고자료 팝업, 저장 시 로컬 즉시 반영(중복 GET 제거), "KBTE → KOBETA" 전체 명칭 변경, DB 스키마 필드 추가(`customTitle`, `level`, `customInstructors`, `schedule`).
- **버그 수정**: 강사 다중 입력 시 한글 IME(조합 입력) 상태에서 Enter 키가 잘못 동작하던 문제 수정.
- **추가/변경 (00:57, `66abb0b`)**: 시간 슬롯 편집 UX 개선 — `TimeSelect` 컴포넌트 추가, 고정 세션 슬롯 수정/삭제 버튼과 메모 라벨 추가.
- **추가/변경 (08:03, `de0ca9a`)**: 인쇄 기능(`window.print()` + `@media print` A4 레이아웃), Pool 즐겨찾기(localStorage), Pool 사이드바 인라인 카테고리 추가, "교육완료 처리"(과정카드 → 교육자료 일괄 등록), Neon DB 안정성 개선(`connect_timeout`/`pool_timeout` 파라미터, `fetchConnectionCache` 활성화로 콜드스타트 개선).
- **버그 수정**: Gemini 모델명 오타(`gemini-3.1-flash-lite`, 존재하지 않는 모델명) → `gemini-2.0-flash-lite`로 수정. 수정 전까지는 트렌드 분석 AI 호출이 실패했을 가능성이 높음.

## 2026-07-04 — 오늘 할 일(Todo) 기능 추가

- **추가/변경 (22:31, `9fcc400`)**: 일지/쇼츠 옆에 "오늘 중심" Todo 기능 추가. 루틴/오늘만 항목 구분, 스와이프·버튼 체크(5초 실행취소), 날짜 이동 시 루틴 자동 복제, 홈 화면 요약 카드, 일지 본문의 "내일 할 일" 문구 자동 등록, 하단 모바일 탭바, PWA 설치 지원(`manifest.json`, 아이콘).

## 2026-07-12 — 문서 정리 및 공개 문서 사이트 구축

- **추가/변경**: 최상위에 흩어져 있던 기획 문서 5개를 `docs/spec/`(제품 스펙)과 `docs/data/`(콘텐츠 자료) 두 폴더로 분류. 순수 캐시/디버그 산출물인 `.DS_Store`, `.playwright-mcp/`(브라우저 자동화 로그·스크린샷 400여 개) 삭제, 재발 방지용 `.gitignore` 추가. 진행 기록 문서(`docs/PROGRESS.md`) 최초 작성.
- **추가/변경**: 공개 문서 사이트 [my-education-docs](https://peter-cho-70.github.io/my-education-docs/) 신설 — GitHub Pages로 발행하고 [peter-cho-70.github.io](https://peter-cho-70.github.io/) 허브에 등록. `stockdashboard-docs` 등 기존 문서 사이트와 동일한 index.html + viewer.html 패턴 사용.
- **추가/변경**: 이 문서 정리 방식을 모든 프로젝트에 적용되는 표준으로 정식화 — `~/.claude/CLAUDE.md`에 "프로젝트 문서화 규칙" 추가, `~/.claude/templates/project-docs-site/`에 재사용 템플릿 저장, 허브에 [문서화 표준](https://peter-cho-70.github.io/doc/viewer.html?file=DOCS-STANDARD.md) 문서 게시.
- **추가/변경**: 위 표준에 맞춰 `docs/PROGRESS.md`를 `docs/STATUS.md`(현재 상태, 덮어쓰기)와 `docs/DEVLOG.md`(이 문서, 날짜별 append)로 분리. `docs/product/` → `docs/spec/`, `docs/curriculum-data/` → `docs/data/`로 폴더명 정리(공개 문서 사이트와 이름 통일).
