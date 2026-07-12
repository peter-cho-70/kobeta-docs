# 데이터 저장 구조 (메모·사진·카테고리)

> 원본: `app/README.md` (앱 저장소). 2026-06-22 업데이트 — Vercel 배포를 위해 로컬 SQLite 파일(`dev.db`) + 로컬 파일시스템(`public/uploads`) 방식에서, **Neon Postgres(서버리스 DB) + Vercel Blob(파일 스토리지)** 으로 데이터 계층을 옮겼습니다. 기존에 로컬에 쌓여있던 모든 데이터(카테고리, 메모/일지, 업로드된 사진)는 마이그레이션 스크립트로 새 저장소에 그대로 옮겨졌습니다.

이 프로젝트에서 사용자가 화면에서 입력하는 모든 내용(메모, 카테고리, 상단 메뉴, 색상 설정 등)은 두 군데에 나뉘어 저장됩니다: **텍스트/구조화된 데이터는 Neon Postgres**, **업로드한 이미지·파일은 Vercel Blob**. 두 저장소 모두 Vercel 프로젝트(`ubsoldboy-2050s-projects/app`)에 마켓플레이스 통합으로 연결되어 있고, 실제 데이터는 Vercel/Neon 쪽 서버에 있습니다 — 로컬 컴퓨터 디스크에는 더 이상 실데이터가 남지 않습니다.

## 1. 데이터베이스 (텍스트 데이터) — Neon Postgres

- **종류**: [Neon](https://neon.tech) Serverless Postgres, Vercel Marketplace 통합(`vercel integration add neon`)으로 프로비저닝됨.
- **연결**: Vercel이 프로젝트에 자동으로 주입하는 환경변수로 접속.
  - `DATABASE_URL` — 풀링(pgbouncer)된 연결. 앱 런타임(`src/lib/prisma.ts`)이 이걸 씀.
  - `DATABASE_URL_UNPOOLED` — 다이렉트 연결. 스키마 마이그레이션(`prisma.config.ts`)이 이걸 씀.
  - 로컬 개발 시에는 `app/.env`에 같은 값이 복사되어 있고, `vercel env pull .env.local`로 다시 받아올 수 있음.
- **ORM**: [Prisma](https://www.prisma.io/) 7 — Prisma 7부터는 커넥션 정보를 `schema.prisma`에 직접 적지 않고, `@prisma/adapter-neon`(`src/lib/prisma.ts`)으로 만든 어댑터를 `PrismaClient`에 넘겨서 연결. 마이그레이션 이력은 `app/prisma/migrations/`에 남음(과거 SQLite용 이력은 `app/.sqlite-migrations-backup/`에 보관, 더 이상 적용되지 않음).
- **데이터를 직접 들여다보고 싶을 때**: `npx prisma studio` (브라우저 GUI) 또는 Vercel 대시보드 → Storage → 해당 Neon 프로젝트에서 SQL 콘솔로 직접 조회.

### 주요 테이블

| 기능 (화면) | 테이블(모델) | 주요 컬럼 | 설명 |
|---|---|---|---|
| 메모/일지 (`/network`, `/stock`, 홈 화면 "오늘의 일지" 등) | `StudyNote` | `id`, `slug`, `title`, `content`, `url`, `imageUrl`, `order`, `createdAt` | `slug`로 어느 메모판인지 구분. 홈의 "오늘의 일지"는 `slug = "journal"`, `/network` 페이지는 `slug = "network"` 식. `imageUrl`에는 **이미지 파일 자체가 아니라 Vercel Blob에 올라간 이미지의 공개 URL** 만 저장. |
| 교육분야 카테고리 (`/admin/categories`) | `Category` | `id`, `name`, `color`, `parentId`, `isActive` | `color`는 트렌드 대시보드 그래프 색상으로 쓰는 헥스 코드(`#xxxxxx`) 문자열. `parentId`로 상위/하위 트리 구조. |
| 상단 메뉴 카테고리 (`/settings`) | `NavCategory` | `id`, `name`, `color`, `order`, `direct`, `categoryId` | 홈 화면 카드와 상단 메뉴에 보이는 항목들. `color`는 홈 카드 배경/제목 색상. `direct=true`면 클릭 시 바로 첫 항목으로 이동, `false`면 드롭다운으로 펼쳐짐. |
| 상단 메뉴 항목 | `NavItem` | `id`, `categoryId`, `label`, `href`, `order` | `NavCategory` 하나에 여러 `NavItem`이 속함(1:N). `href`가 페이지 경로면 그 경로가 곧 `StudyNote.slug`가 됨. |

## 2. 파일/이미지 저장 (업로드된 사진) — Vercel Blob

- **종류**: [Vercel Blob](https://vercel.com/docs/vercel-blob) — `education-uploads` 스토어(public access).
- **연결**: `BLOB_READ_WRITE_TOKEN` 환경변수(Vercel이 자동 주입) — `app/src/lib/fileStorage.ts`의 `saveUploadedFile()`이 이 토큰으로 `@vercel/blob`의 `put()`을 호출.
- **경로 규칙**: `materials/<파일명>`, `videos/<파일명>`, `notes/<파일명>` (용도별). 업로드된 파일은 공개 URL을 받고, **DB에는 이 URL 문자열만** 저장(파일 바이트 자체는 DB에 없음).
- **사진 붙여넣기(Ctrl+V) 흐름**:
  1. 메모 작성/수정 화면(`MemoBoard` 컴포넌트)의 "내용" 입력칸에서 이미지를 붙여넣으면, 브라우저 클립보드에서 이미지 파일을 즉시 꺼냄.
  2. `POST /api/study-notes/upload-image`로 그 이미지 파일을 멀티파트 폼으로 전송.
  3. 서버가 `saveUploadedFile(file, "notes")`로 Vercel Blob에 업로드하고, 생성된 공개 URL을 응답으로 돌려줌.
  4. 화면에 작은 미리보기로 그 URL의 이미지를 보여주고, 메모를 "추가"/"저장"할 때 그 `imageUrl` 값을 메모 본문과 함께 `POST/PATCH /api/study-notes`로 전송해 `StudyNote.imageUrl` 컬럼에 저장.
  5. 이후 메모를 불러올 때는 `StudyNote.imageUrl`에 저장된 Blob URL을 `<img src="...">`로 그대로 렌더링.

## 3. 요약: 메모 하나를 등록하면 실제로 어떤 일이 일어나는가

1. 사용자가 홈 화면 "오늘의 일지"(또는 `/network` 등 메모 페이지)에서 제목/내용을 쓰고 사진을 붙여넣은 뒤 "추가"를 누름.
2. (사진이 있었다면) 그 사진은 이미 Vercel Blob에 업로드되어 있고, 그 공개 URL 문자열만 들고 있음.
3. 브라우저가 `{ slug, title, content, url, imageUrl }`을 JSON으로 `POST /api/study-notes`에 보냄.
4. 서버(API Route, `app/src/app/api/study-notes/route.ts`)가 Prisma를 통해 Neon Postgres의 `StudyNote` 테이블에 새 행(row)을 추가.
5. 화면은 다시 `GET /api/study-notes?slug=...`로 목록을 새로 받아와 보여줌.

## 4. 로컬 개발 환경 — 로컬과 배포가 데이터를 공유함 (주의)

- 로컬에서 `npm run dev`로 띄워도 **같은 Neon Postgres / 같은 Vercel Blob 스토어**를 그대로 씀 — 로컬에서 메모를 추가하면 배포된 사이트에도 똑같이 보임.
- 환경변수가 바뀌었다면 `vercel env pull .env.local`로 최신 값을 다시 받아오면 됨.
- 과거 로컬 SQLite(`dev.db`)와 `public/uploads/`에 있던 데이터는 일회성 마이그레이션 스크립트로 전부 Postgres/Blob에 옮겨졌고, 이후로는 더 이상 쓰지 않음(둘 다 `.gitignore` 처리됨).
- 실험적인 로컬 작업이 실 데이터에 영향을 주지 않게 하려면 Neon의 브랜치 기능으로 별도 개발용 DB 브랜치를 분리하는 것을 검토할 수 있습니다 (`doc/PROGRESS.md`의 "알려진 이슈" 참고).
