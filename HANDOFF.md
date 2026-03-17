# 작업 인수인계

---
---

# [프로젝트 4] 치크루팅 상품/판매 대시보드

**최초 작성**: 2026년 3월 9일
**최근 업데이트**: 2026년 3월 17일 (세션 5)
**프로젝트**: 운영 현황을 한눈에 파악하는 실시간 대시보드

---

## 목표
- Google Sheets(주문/채용관) + Strapi(공고) + Supabase(스냅샷/경쟁사) 데이터를 통합한 운영 대시보드
- 매시간 GitHub Actions → Cloudflare Workers 자동 배포 (정적 HTML)
- 팀 공유 URL로 항상 최신 데이터 확인

---

## 현재 진행 상황

### ✅ 완료
- **대시보드 7개 탭 구현** (순서 변경됨): 요약 / 공고 현황 / 월별 매출 추이 / 주문 내역 / 해피콜 대상 / VIP 채용관 관리 / 프리미엄 채용관 관리
- **탭 순서 재배치 (세션 2)**: 자주 보는 현황/추이 탭을 앞, 상세 관리 탭을 뒤로
  - `showTab` 함수 내 `tabs` 배열도 HTML 순서와 동기화 필수 (불일치 시 탭 클릭 오류 발생)
- **공고 현황 탭**:
  - KPI 5개 (퍼널 구조): 정규직 활성공고 / 이번달 지원자 / 공고당 평균 지원자 / 치과119 활성 / 덴탈잡 대비%
  - 정규직 추이 차트 (치크루팅 vs 덴탈잡): **주별이 디폴트** + 월별 토글 (20주)
  - **덴탈잡 대비 비율 추이 바 차트 (세션 2)**: 주별 모드에서 각 바 위에 % 수치 직접 표시 (chartjs-plugin-datalabels), 색상으로 수준 구분 (≥20% 초록, ≥15% 파랑, ≥10% 노랑, <10% 빨강)
  - 퍼널 차트: 조회수(막대) + 지원자수(선) 이중 Y축
  - **치과119 아르바이트 차트: 주별/월별 토글 추가 (세션 2)** — 주별 디폴트, 20주 기준
  - **경쟁사 입력 폼 확장 (세션 2)**: 덴탈잡/사람인 + 잡코리아/인디드/알바몬/직접입력
  - **엑셀 다운로드 개선 (세션 2)**: 3종 시트 — 전체 / 날짜별 비교(피벗) / 플랫폼별 개별 시트
  - 입력 내역 테이블: 최신순 정렬
- **월별 매출 추이 탭**: 월별 상세 테이블 최신순 정렬 (세션 2)
- **Supabase 테이블 구축 및 데이터 임포트**:
  - `chicruiting_snapshots` (date, posting_type, active_count): 정규직 277건 + 아르바이트 84건
  - `competitor_tracking` (date, competitor, post_count, entered_by): 덴탈잡 120건 + 사람인 19건
- **데일리 스냅샷 자동 기록**: `update_dashboard.js` 빌드 시 오늘 스냅샷 없으면 정규직/아르바이트 자동 insert
- **update_dashboard.js**: job-posting-contracts 최근 12개월 수집 추가 (`ALL_CONTRACT_POSTINGS`)
- **폴더 구조 정리 (세션 3)**: 대시보드 파일 전체를 `대시보드/` 독립 폴더로 분리
  - `대시보드/web/` — HTML 파일 (dashboard_template.html, 상품_등록_현황_대시보드.html, crm.html 등)
  - `대시보드/scripts/` — 빌드/임포트 스크립트
  - `대시보드/package.json`, `대시보드/wrangler.toml`
- **GitHub 독립 레포 연결 (세션 3)**: `teamdeneer/chicruiting-dashboard` (main 브랜치), 최신 커밋 `6592be5`
- **자동 배포 (세션 3)**: GitHub Actions → Cloudflare Workers (`npx wrangler deploy`)
  - KST 08:00~18:30 30분마다 + KST 19:00~22:00 1시간마다 (세션 4에서 변경)
- **차트 버그 수정 (세션 3)**:
  - `_snapByWeek`: 주 마지막값 → 찍힌 날 평균으로 변경 + 오늘 날짜 포함
  - 공고 관심도 퍼널 지원자: 공고 등록월 기준 → 지원서 생성월 기준으로 변경 (`appMonths` 배열 추가)
- **디자인 브랜드 적용 (세션 3)**: 네이비 다크 테마 → 치크루팅 디자인 시스템
  - 헤더: `#3872F0` 브랜드 블루, 폰트: Pretendard, 배경: `#F5F6FA`, 테두리: `#E1E4EC`
- **UI 개선 (세션 4)**:
  - 요약 탭 VIP `/4`, 프리미엄 `/16` → 카드 우측 하단에 작게/얇게/회색 표시 (position:absolute)
  - 요약 탭 "프리미엄 공고 총 지원자" KPI → 프리미엄 채용관 관리 탭으로 이동
- **스냅샷 쿼리 개선 (세션 4)**:
  - 기존: `filters[isTemp]=false&filters[status]=ACTIVE` → 실제보다 훨씬 적은 수치 기록
  - 변경: `filters[status][$in][0]=ACTIVE&[$in][1]=EDITED&[$in][2]=PENDING` (isTemp 필터 제거)
  - 2026-03-09~13 수치 Supabase에서 1026으로 수동 수정 완료
- **CRM 연동 (세션 4)**:
  - `crm.html` 별도 파일로 존재 → 같은 Cloudflare Workers에 `/crm.html` 경로로 배포 추가
  - `syncCrmContacts`: Strapi `/api/companies` 전체 병원 회원 → `crm_contacts` 자동 동기화
  - null 값 제거로 수동 입력 데이터(태그/메모) 덮어쓰기 방지
- **빌드 스케줄 변경 (세션 4)**:
  - 기존: 매시간 정각
  - 변경: KST 08:00~18:30 30분마다 + KST 19:00~22:00 1시간마다 (하루 26회)
- **Basic Auth 비밀번호 보호 (세션 4)**:
  - `worker.js` 추가: DASHBOARD_PASSWORD 환경변수로 Basic Auth 검증
  - `wrangler.toml`: `main = "worker.js"` + `binding = "ASSETS"` 추가
  - 비밀번호: Cloudflare Workers Secret에 `DASHBOARD_PASSWORD`로 등록 완료
  - 아이디는 아무거나, 비밀번호만 맞으면 접속 가능
- **Node.js 24 업그레이드**: GitHub Actions deprecation 경고 해소 (FORCE_JAVASCRIPT_ACTIONS_TO_NODE24=true 설정, 경고는 GitHub 안내 메시지라 계속 보임 — 무시해도 됨)

### ✅ 완료 (세션 5: 2026-03-17)

#### 보안 강화
- **Strapi 토큰 하드코딩 제거**: `scripts/fetch_all_premium.js`에 박혀 있던 토큰을 `process.env.STRAPI_TOKEN`으로 변경
- **기존 Strapi 토큰 폐기 + 신규 토큰 발급**: Read-only (find + findOne) 권한으로 재발급, GitHub Secret `STRAPI_TOKEN` 업데이트 완료
- **Supabase 키 GitHub Secrets 이전**: 기존 코드에 하드코딩되어 있던 `SUPABASE_URL`, `SUPABASE_KEY`를 모두 GitHub Secrets로 이전
- **GitHub 레포 Public 전환 가능 상태 확인**: Secrets 이전 + Basic Auth로 코드/데이터 모두 보호됨
- **Basic Auth 팝업 수정**: `wrangler.toml`에 `run_worker_first = true` 추가 → Cloudflare가 정적 파일 직접 서빙하지 않고 worker를 먼저 거치도록 수정 (이전엔 Basic Auth 팝업이 뜨지 않았던 원인)

#### Integration Playground 이전 (사내 인프라)
- **배경**: 개발 리더가 모든 내부 도구를 회사 인프라(`playground.integrationcorp.co.kr`)에 배포하도록 공지
- **SSH 키 생성 및 Bitbucket 등록**: `~/.ssh/id_ed25519_bitbucket` 생성, Bitbucket 계정에 공개키 등록
- **integration-playground 클론**: `git@bitbucket.org:medistream-team/integration-playground.git`
  - React 19 + TypeScript + Vite + shadcn/ui + TanStack Query 풀스택 앱
  - 기존에 CX 대시보드(수멤버스/아큐렉스 상담 관리)가 iframe 방식으로 이미 포함되어 있음
- **치크루팅 대시보드 페이지 추가**:
  - `frontend/src/features/chicruiting-dashboard/pages/ChicrutingDashboardPage.tsx` 생성 (iframe 방식)
  - `frontend/src/app/router.tsx`에 `/chicruiting-dashboard` 라우트 추가
  - `frontend/src/components/layout/sidebar-data.ts`에 "치크루팅 대시보드" 메뉴 추가 (Stethoscope 아이콘)
  - `frontend/public/chicruiting-dashboard.html` 플레이스홀더 생성 (파이프라인 연결 전 임시)
  - Bitbucket push 성공 → Pipeline #96 자동 빌드/배포 성공
- **GitHub Actions → Bitbucket 자동 푸시 파이프라인 연결**:
  - `build.yml`에 "Integration Playground에 대시보드 HTML 푸시" 스텝 추가
  - GitHub Secret에 `BITBUCKET_SSH_KEY` 추가 (private key: `~/.ssh/id_ed25519_bitbucket`)
  - 빌드 후 `web/상품_등록_현황_대시보드.html` → Bitbucket `frontend/public/chicruiting-dashboard.html`로 자동 커밋/푸시
  - Pipeline #99 성공 확인 — 이후 매 빌드마다 playground에 최신 HTML 반영됨

### 🚧 미완료
- 사람인 데이터는 DB에 있으나 차트에 미표시 (필요 시 추가 가능)
- 주별 차트에서 덴탈잡 데이터 희박 (주 1회 수동 입력 기반) → 시간 지나면 자연 해결
- **`playground.integrationcorp.co.kr` 접속 확인 필요**: 사이드바 "치크루팅 대시보드" 메뉴 노출 여부 — 2026-03-17 재택으로 VPN 접속 불가, 출근 후 확인 예정
- **Strapi company 0건 이슈**: 빌드 로그에 `Strapi 병원(company): 0건`으로 출력됨. 토큰에 find/findOne 권한 설정했음에도 0건. 원인 재확인 필요
- **`crm_contacts` Supabase 테이블 미생성**: 현재 `crm_contacts 테이블 미생성 — 동기화 건너뜀` 출력 중. 테이블 생성 후 CRM 연동 재시도 필요
  - 테이블 컬럼: `hospital_name` (PK), `company_id`, `contact_name`, `phone`, `region`, `region_detail`

---

## 성공한 접근법
- **치과119 = 치크루팅 아르바이트**: competitor가 아닌 자사 서비스 → `posting_type='contract'`로 분류
- **중복 방지**: Supabase insert 시 `resolution=ignore-duplicates` 헤더 사용
- **주별 집계**: 월요일 기준 `_snapByWeek` / `_compByWeek` 함수로 주간 마지막 스냅샷 추출
- **datalabels 전역 비활성**: `Chart.defaults.plugins.datalabels = { display: false }` 후 비율 차트에서만 `plugins: [ChartDataLabels]` + `display: true`로 개별 활성 (다른 차트에 영향 없음)
- **COMP_LABELS 매핑 객체**: 경쟁사 코드→한글 변환을 한 곳에서 관리

## 실패한 접근법 (반복 금지)
- **`type` 컬럼명**: PostgreSQL 예약어 → `posting_type`으로 변경 (DROP TABLE 후 재생성)
- **배치 insert 중복 충돌**: `resolution=ignore-duplicates`가 배치에서 비정상 동작 → 1건씩 loop 처리
- **탭 순서 변경 시 tabs 배열 미동기화**: HTML 탭 순서만 변경하고 `showTab` 내 `tabs` 배열을 안 바꾸면 탭 클릭 시 엉뚱한 탭에 active 클래스 부여됨

---

## 다음 할 일

### 🟠 사람인 추이 차트 추가 (선택적)
- `competitor_tracking`에 saramin 데이터 존재, `renderJobTrendChart`에 데이터셋 추가하면 됨

### 🟡 경쟁사 데이터 정기 입력 루틴
- 덴탈잡: 매일 수동 입력 / 사람인: 매주 수동 입력
- 대시보드 "공고 현황" 탭 > 경쟁사 공고 수 입력 폼 사용 (잡코리아/인디드/알바몬/직접입력도 가능)

### 🔴 playground 접속 확인 (출근 후)
- `playground.integrationcorp.co.kr` 접속 → 로그인(deneer.co.kr Google 계정) → 사이드바 "치크루팅 대시보드" 메뉴 확인
- 안 보이면 개발 리더에게 접근 권한 문의

### 🔴 crm_contacts 테이블 Supabase 생성
- Supabase `gomtqjpoqdjnbolhzdze.supabase.co` → SQL Editor에서 아래 실행:
```sql
CREATE TABLE crm_contacts (
  hospital_name TEXT PRIMARY KEY,
  company_id TEXT,
  contact_name TEXT,
  phone TEXT,
  region TEXT,
  region_detail TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```
- 생성 후 GitHub Actions 수동 실행 → `고객 N건 동기화 완료` 로그 확인

### 🔴 Strapi company 0건 원인 파악
- 신규 Read-only 토큰의 company find/findOne 권한 설정 후에도 0건 출력 중
- Strapi 관리자에서 토큰 권한 재확인 필요 (Settings → API Tokens → 해당 토큰 → Company: find 체크 여부)

---

## 주요 파일/경로

| 파일 | 경로 | 용도 |
|------|------|------|
| dashboard_template.html | `클로드코드/대시보드/web/dashboard_template.html` | 대시보드 HTML 템플릿 |
| update_dashboard.js | `클로드코드/대시보드/scripts/update_dashboard.js` | 빌드 스크립트 (`node scripts/update_dashboard.js`) |
| 상품_등록_현황_대시보드.html | `클로드코드/대시보드/web/상품_등록_현황_대시보드.html` | 빌드 결과물 (로컬 미리보기) |
| import_competitor_data.js | `클로드코드/대시보드/scripts/import_competitor_data.js` | 덴탈잡/사람인 CSV → Supabase |
| import_chicruiting_snapshots.js | `클로드코드/대시보드/scripts/import_chicruiting_snapshots.js` | 정규직 과거 스냅샷 CSV → Supabase |
| build.yml | `클로드코드/대시보드/.github/workflows/build.yml` | GitHub Actions 빌드/배포 설정 |
| wrangler.toml | `클로드코드/대시보드/wrangler.toml` | Cloudflare Workers 배포 설정 |

**GitHub**: `teamdeneer/chicruiting-dashboard` (main 브랜치), 최신 커밋 `7e164d6`
**배포 URL**: `chicruiting-dashboard.team-999.workers.dev` (Basic Auth 비밀번호 보호)
**Playground URL**: `playground.integrationcorp.co.kr/chicruiting-dashboard` (사내 인프라, deneer.co.kr Google 계정 로그인 필요)
**Bitbucket**: `bitbucket.org/medistream-team/integration-playground` (main 브랜치)
**SSH 키**: `~/.ssh/id_ed25519_bitbucket` (Bitbucket 공개키 등록 완료, GitHub Secret `BITBUCKET_SSH_KEY`에 private key 등록 완료)
**Supabase**: `gomtqjpoqdjnbolhzdze.supabase.co`
- `chicruiting_snapshots`: unique(date, posting_type), posting_type = 'regular' | 'contract'
- `competitor_tracking`: competitor = 'dentalJob' | 'saramin' | 'jobkorea' | 'indeed' | 'albamon' | 기타(직접입력)

**외부 라이브러리**:
- Chart.js 4.4.1
- chartjs-plugin-datalabels 2.2.0 (비율 차트 수치 표시용)
- SheetJS (XLSX) 0.18.5

---

## 새 세션 시작 방법

```
이 파일 읽고 이어서 작업해줘:
C:\Users\Yujeong Ha\Desktop\클로드코드\HANDOFF.md

치크루팅 상품/판매 대시보드 작업 이어서 할게.
```

---
---

# [프로젝트 3] 치크루팅 상품 추천 랜딩페이지

**최초 작성**: 2026년 3월 9일
**최근 업데이트**: 2026년 3월 9일
**프로젝트**: 상품이 늘어나며 선택 부담 증가 → 맞춤 추천 랜딩페이지로 해결

---

## 목표
- 상품 수 증가로 고객 선택 어려움 해소 → 추천 랜딩페이지 제공
- 들어오자마자 생각 없이 한 번만 탭하면 상품이 바로 보이도록
- 추천 페이지에서 바로 구매 연결

---

## 팀 결정 사항 (2026-03-09)

1. **기존 상품 가격 변경 금지** — 현재 잘 팔리고 있어 크리티컬 리스크 있음, 신규 상품만 추가
2. **신규 상품 추가**: 지방권 대응 소규모 상품
   - `면접제안권 10건 (90일)` — 25,000원 (유효기간 90일, 건당 2,500원)
3. **추천 랜딩 UX 원칙**: 고민/생각 비용 최소화, 한 바닥에 다 보여줌

---

## 현재 진행 상황

### ✅ 완료
- 추천 랜딩 HTML 제작: `치크루팅_추천랜딩.html`
- UX 구조: 상황 탭 4개 상단 배치 → 탭 클릭 → 즉시 상품 카드 표시
  - 탭: 😥 지원자 없음 / 🤔 질 낮음 / 📍 지방 / ⚡ 급함
  - 예산 필터(전체/10만원이하/10~20만원/20만원이상)는 선택사항
  - 기본값: "전체" — 모든 상품 보임
- 상품 데이터 4가지 상황 × 3가지 예산 구성 완료
- "신규" 뱃지로 면접제안권 10건 표기

### 🚧 미완료 (다음 할 일)
- 실제 Strapi 상품 ID/URL 연결 (현재 "구매하러 가기" 버튼 미연결)
- 신규 상품(면접제안권 10건) Strapi 실제 등록 필요
- 랜딩 페이지 실제 배포 (현재 로컬 HTML만 존재)

---

## UX 설계 핵심 원칙

- **단계 제거**: 기존 Step1→2→3 흐름 → 탭 1번으로 즉시 결과
- **전체 노출**: 기본값 "전체 예산"으로 들어오자마자 상품 보임
- **추천 뱃지**: 가장 적합한 상품 1개에 초록 "추천" 뱃지
- **secondary 카드**: 대안 상품도 함께 표시 (border만 gray)

---

## 실패한 접근법

- **Step1→2→3 퀴즈형**: 상황 선택 → 예산 선택 → 결과 순서
  - 문제: 2번 생각해야 상품 보임, 인지 비용 높음 → 탭+즉시표시로 교체

---

## 주요 파일/경로

| 파일 | 경로 | 용도 |
|------|------|------|
| 치크루팅_추천랜딩.html | `클로드코드/치크루팅_추천랜딩.html` | 상품 추천 랜딩 페이지 (배포 필요) |
| 상품_전략_최종_확정안.md | `클로드코드/상품_전략_최종_확정안.md` | 전체 상품 라인업 기준 |

---

## 다음 할 일

### 🔴 1. 신규 상품 Strapi 등록
- 상품명: `면접제안권 10건 (90일)`
- 가격: 25,000원
- 유효기간: 90일
- 담당: 개발팀 요청 필요 (Strapi 직접 수정 금지)

### 🔴 2. 구매 버튼 URL 연결
- Strapi 각 상품 페이지 URL 확보
- `치크루팅_추천랜딩.html` 내 `buy-btn` 클릭 시 해당 URL로 이동 처리

### 🟠 3. 랜딩 페이지 배포
- 정적 HTML → Netlify/Vercel 또는 치크루팅 서비스 내 연동
- 배포 후 URL → 카카오톡 링크, 채널톡 버튼 등에 삽입

### 🟡 4. 상품 추천 데이터 검증
- 현재 추천 상품 데이터는 기획 기준으로 작성
- 실제 Strapi 상품명/가격과 일치 여부 확인 필요

---

## 새 세션 시작 방법

```
이 파일 읽고 이어서 작업해줘:
C:\Users\Yujeong Ha\Desktop\클로드코드\HANDOFF.md

치크루팅_추천랜딩.html 구매 버튼 URL 연결부터 할게.
```

---
---

# [프로젝트 2] 치크루팅 FAQ 도움센터 + 채널톡 봇 설정

**최초 작성**: 2026년 2월 27일
**최근 업데이트**: 2026년 3월 10일 (세션 3)
**프로젝트**: 치크루팅 FAQ 검색 홈페이지 + 채널톡 상담 자동화 연동

---

## 목표
- 치크루팅 서비스에 연동할 **검색 가능한 FAQ 도움센터** 웹페이지 제작
- 유저가 채널톡 봇에서 **1-2번 클릭**으로 바로 답변을 찾을 수 있도록 페이지별 봇 설정
- FAQ 도움센터와 채널톡 봇을 **딥링크(URL 파라미터)**로 연동하여 UX 통합

---

## 현재 진행 상황

### ✅ 완료된 것들

#### 1. FAQ 정적 페이지 (`faq.html`)
- 단일 HTML 파일 (CSS+JS 올인원, Pretendard 폰트만 외부 CDN)
- 치크루팅 디자인 시스템 컬러 적용 (메인: `#3872F0`)
- 모바일 반응형 + iPhone 노치 대응 (safe-area-inset)

#### 2. 구현된 기능
- **스마트 검색**: 실시간 자동완성 + Enter로 전체 검색
  - 동의어 사전 (`SYNONYM_MAP`) 50+ 키워드 지원
  - 가중치 검색 (phrase 50pt > word 12pt > synonym 6pt)
  - XSS 방지 sanitize 적용
- **회원 유형 토글**: 전체 / 병원회원 / 개인회원
  - 검색박스 안에 토글+검색창을 하나의 카드로 묶은 UX
  - 유형 선택 시 서브타이틀, 검색 placeholder, 인기 질문 자동 변경
  - localStorage로 유형 선택 기억
- **추천 질문 버블**: isPopular=TRUE 항목 상위 5개 표시
  - 회원 유형에 따라 자동 필터링
- **답변 모달 팝업**: 검색 결과 클릭 시 모달로 답변 표시
  - 카테고리 뱃지 + 회원 유형 뱃지
  - 모바일에서 bottom-sheet 패턴
- **검색 결과 카드**: 좌측 파란색 accent bar, 답변 미리보기
  - 매칭 유형 / 다른 유형 자동 분류 + 구분선
- **카테고리 아코디언**: 6개 카테고리
- **검색 하이라이트**: 단일 정규식 패턴 (중첩 mark 방지)
- **줄바꿈 처리**: CSV 멀티라인 필드 → `<br>` 렌더링
- **URL 파라미터 딥링크**:
  - `?q=키워드` → 자동 검색
  - `?category=카테고리명` → 해당 카테고리 섹션 스크롤
  - `?type=employer|jobseeker` → 유저 타입 자동 설정
- **검색 초기화**: X 버튼 + 하단 초기화 버튼 + 버블 복원
- **Escape 키**: 답변 모달 / 전화 팝업 닫기

#### 3. Google Sheets 연동 (완료)
- Google Sheets "웹에 게시" → CSV URL로 실시간 연동
- `parseCSV()`: 멀티라인 따옴표 필드 정상 파싱 (character-by-character)
- 5초 AbortController 타임아웃
- file:// 프로토콜에서는 Sheets 로드 스킵 → SAMPLE_FAQ 사용
- 오프라인/실패 시 기본 FAQ 표시 + 오프라인 배너

#### 4. 디자인 최적화 (세션 3)
- **히어로 섹션**: "도움이 필요하신가요?" + 서브타이틀 + 검색박스
  - 검색박스 = 연한 하늘색(`#EEF4FF`) 카드에 토글+검색창 통합
  - shadow로 떠있는 카드 느낌 (`box-shadow: 0 2px 12px rgba(56,114,240,0.08)`)
- **Header**: backdrop-filter blur (반투명 글래스 효과)
- **크루니 마스코트**: 하단 CTA 영역에 140px 배치
  - 검색 결과 없을 때 120px 크루니 표시
- **FAQ 답변 영역**: 배경색 `#FAFBFD` + 텍스트 `#3D3F45`로 가독성 향상
- **카테고리 아이콘**: 40px, border-radius 10px
- **검색 결과 카드**: 좌측 accent bar (`border-left: 3px solid #3872F0`)
  - 다른 유형 결과: 배경색 구분 (opacity 대신)
- **버블 :active 상태**: 모바일 터치 피드백
- **Focus ring**: 검색박스 배경에서도 잘 보이도록 보정
- **OG meta 태그**: 소셜 공유 미리보기

#### 5. 채널톡 봇 설정 가이드 (`채널톡_봇설정_가이드.md`)
- **상담 데이터 분석 결과** (미해결 1,715건, 메시지 10,015개)
- **7개 페이지별 봇 버튼 구성** 작성 완료
- FAQ 딥링크 URL 형식 정리 (배포 후 실제 URL로 교체 필요)

#### 6. FAQ 데이터
- 50개 FAQ (`치크루팅_FAQ.csv` + Google Sheets)
- 카테고리 6개: account / recruitment / jobseeker / premium / payment / service
- Google Sheets에서 직접 수정 가능 (스프레드시트 사용법은 시트 내 별도 탭에 정리됨)

---

## 스프레드시트 관리 가이드

### 컬럼 구성
| 컬럼 | 필수 | 설명 | 예시 |
|---|---|---|---|
| question | O | 질문 | 비밀번호를 변경하고 싶어요 |
| answer | O | 답변 (Ctrl+Enter로 줄바꿈) | 마이페이지 > 설정에서... |
| category | O | 카테고리 코드 | account |
| userType | O | 대상: `employer` / `jobseeker` / `all` | employer |
| tags | - | 검색 키워드 (쉼표 구분) | 비번, 패스워드, PW |
| isPopular | - | TRUE=자주묻는질문 표시 (상위 5개) | TRUE |
| order | - | 정렬 순서 (작을수록 위) | 1 |

### category 코드
- `account` → 👤 계정 · 로그인 · 프로필
- `recruitment` → 📋 채용공고 · 지원자 관리
- `jobseeker` → 🙋 이력서 · 지원 · 면접 제안
- `premium` → ⭐ 인재 탐색 · 면접제안권
- `payment` → 💳 결제 · 환불
- `service` → 🛠 앱 오류 · 기타

### 답변 서식
- 줄바꿈: **Ctrl + Enter** (Mac: Cmd + Enter)
- 굵게: `<b>텍스트</b>` 또는 `<strong>텍스트</strong>`
- 기울임: `<em>텍스트</em>`
- 수정 후 반영까지 최대 5분 (Google 캐시)

---

## 성공한 접근법

- **단일 HTML 파일**: 배포 간단, 외부 의존성 없음
- **URL 파라미터 딥링크**: 채널톡 봇 버튼에서 FAQ 페이지로 바로 연결
- **Google Sheets CSV 연동**: API 키 없이 "웹에 게시" 기능으로 연동
- **가중치 검색 + 동의어 사전**: 단순 fuzzy 대비 정확도 대폭 향상
- **회원 유형 토글과 검색창 통합**: 유형 선택이 검색을 위한 것임을 직관적으로 전달
- **SAMPLE_FAQ 즉시 표시 + Sheets 백그라운드 로드**: 로딩 체감 속도 개선
- **크루니 마스코트 하단 CTA 배치**: 히어로는 검색 중심, 하단에서 캐릭터 친근함 제공

## 실패한 접근법 (반복 금지)

- **oopy.io/chicruiting.com 크롤링**: Notion 기반 동적 사이트라 WebFetch 불가
- **Google Sheets MCP 자동 업로드**: MCP 로드 실패 → CSV 수동 가져오기로 대체
- **Google Sheets gviz/tq URL**: 401 에러 → `pub?output=csv` 형식으로 변경
- **csv.split('\n')**: 멀티라인 CSV 필드 깨짐 → character-by-character 파서로 교체
- **escapeHtml() 후 highlightText()**: `<br>` 태그가 escape됨 → 정규식 negative lookahead로 해결
- **forEach 다중 replace**: 중첩 `<mark>` 태그 발생 → 단일 정규식 `(word1|word2)` 패턴으로 해결
- **크루니 히어로 타이틀 옆 배치**: 48px은 너무 작고 애매 → 하단 CTA에 140px로 이동

---

## 다음 할 일

### 🔴 1. faq.html 배포
- Netlify/Vercel 등에 정적 파일로 배포
- 필요 파일: `faq.html` + `logo.png` + `favicon.ico` + `도움센터_크루니.png` + `도움센터_크루니_결과없음.png`
- 배포 후 실제 URL 확보 → 채널톡 봇 딥링크 URL 교체

### 🟠 2. 채널톡 워크플로우 실제 적용
- `채널톡_봇설정_가이드.md` 내용을 채널톡 관리자에서 실제 세팅
- 각 페이지별 봇 트리거 URL 조건 + 버튼 구성 등록
- FAQ 딥링크 URL을 실제 배포 URL로 교체

### 🟡 3. FAQ 내용 보완
- 현재 50개 → 최신화 필요
- Google Sheets에서 직접 수정/추가 가능
- 카테고리별 인기 질문(isPopular) 재검토

---

## 주요 파일/경로

| 파일 | 경로 | 용도 |
|------|------|------|
| faq.html | `클로드코드/faq.html` | FAQ 도움센터 메인 페이지 |
| 도움센터_크루니.png | `클로드코드/도움센터_크루니.png` | 크루니 마스코트 (하단 CTA용) |
| 도움센터_크루니_결과없음.png | `클로드코드/도움센터_크루니_결과없음.png` | 크루니 (검색 결과 없음용) |
| 채널톡_봇설정_가이드.md | `클로드코드/채널톡_봇설정_가이드.md` | 채널톡 봇 페이지별 설정 가이드 |
| 치크루팅_FAQ.csv | `클로드코드/치크루팅_FAQ.csv` | Google Sheets 초기 데이터 |
| logo.png | `클로드코드/logo.png` | 치크루팅 메인 로고 |
| favicon.ico | `클로드코드/favicon.ico` | 브라우저 탭 아이콘 |

---

## 새 세션 시작 방법

```
이 파일을 읽고 작업을 이어서 진행해줘:
C:\Users\Yujeong Ha\Desktop\클로드코드\HANDOFF.md

치크루팅 FAQ 도움센터 작업 이어서 할게.
```

---
---

# [프로젝트 1] 치크루팅 상품 전략

**최초 작성**: 2026년 2월 13일
**최근 업데이트**: 2026년 3월 11일 (세션 5)
**프로젝트**: 치크루팅 채용 상품 패키지 재설계 + 골드 채용관 런칭

---

## 목표

**치크루팅 채용 상품의 전면 리뉴얼**:
1. 기존 단품 중심 → 예산별 패키지 중심으로 전환
2. 3단계 노출 구조 신설 (VIP → 골드 → 프리미엄)
3. 922건 주문 데이터 기반 의사결정
4. 디코이 효과 활용 (단품 가격 유지 → 패키지 구매 유도)

---

## 현재 진행 상황

### ✅ 완료

#### 1. 데이터 분석 (922건 주문, 386개 병원)
- 재구매율 37.3%, 평균 지출 15-30만원
- VIP/프리미엄 효과 입증 (+3.6명 ~ +7.4명)
- 교차구매율 5% → 면접 고객 ≠ 채용관 고객 → 강제 번들 폐지 근거
- 설문 101명: 10만원 미만 선호 47%, 10-30만원 40%

#### 2. 3단계 노출 구조 확정
```
VIP채용관 (4구좌) - 최상단 독점
     ↓
골드채용관 (10구좌) 🆕 - 메인 첫 화면
     ↓
프리미엄채용관 (16구좌) - 대중적
```

#### 3. 패키지 라인업 최종 확정 (2026-03-11)
| # | 패키지 | 구성 | 단품합 | 패키지가 | 할인율 | 포지셔닝 |
|---|--------|------|--------|----------|--------|----------|
| 1 | **가성비** | P3일 + 면접20 | 100,000원 | **87,000원** | 13% | 찍먹 / 소규모 채용 |
| 2 | **실속형** ⭐ | P7일 + 면접50 | 182,000원 | **154,000원** | 15% | 일반 채용 (핵심 추천) |
| 3 | **올인원** 🆕 | 골드7일 + 면접30 + 이미지 제작 | 287,000원 | **244,000원** | 15% | 종합 패키지 (개원 추천) |
| 4 | **듀얼 노출** 🆕 | 골드7일 + P7일 | 321,000원 | **279,000원** | 13% | 노출 극대화 |

**할인율 구조**: 13% → 15% → 15% → 13% (가운데 핵심 상품 할인 극대화)

**고객 여정 퍼널**:
```
[신규가입] 면접 5건 무료 → [맛보기] 면접10건(25K) → [첫구매] 면접30건(49K)
→ [패키지] 가성비(87K) → 실속형(154K) → 올인원(244K) → 듀얼노출(279K)
→ [프리미엄 단품] 골드·VIP 단품 → [엔터프라이즈] 치즈톡 프리미엄
```

#### 4. 면접제안권 단품 가격 체계
| 상품 | 건수 | 가격 | 건당 가격 |
|------|------|------|-----------|
| 면접제안권 10건 | 10건 | 25,000원 | 2,500원 |
| **면접제안권 20건** 🆕 | 20건 | 41,000원 | 2,050원 |
| 면접제안권 30건 | 30건 | 49,000원 | 1,633원 |
| 면접제안권 50건 | 50건 | 70,000원 | 1,400원 |
| 면접제안권 100건 | 100건 | 120,000원 | 1,200원 |

#### 5. 골드 채용관 등록 정보 정리 완료
- **판매 방식**: 마켓 내부 상품 (랜딩 페이지 X)
- 골드 채용관 7일: 정가 380,000원 → **판매가 209,000원** (45% 할인)
- 골드 채용관 14일: 정가 650,000원 → **판매가 357,500원** (45% 할인)
- 구좌: 초기 5~6개 (최대 10개)
- 노출 위치: 치크루팅 메인 홈 첫 화면

#### 6. 골드 채용관 상세페이지 구성안 작성 완료
- 마켓 홈 썸네일 카드 카피 작성
- 상세페이지 구성: 히어로 → 3단계 비교표 → 소구 포인트 3개 → 이용 방법 → FAQ
- VIP/프리미엄 대비 포지셔닝: "VIP의 효과, 프리미엄의 가격"

### 🚧 미완료

#### 🔴 즉시 필요
1. **골드 채용관 Strapi 등록** — 등록 정보 정리 완료, 실제 등록 대기
2. **골드 채용관 상세페이지 실제 제작** — 구성안 작성 완료, 구현 대기
3. **면접제안권 20건 Strapi 등록** — 신규 상품

#### 🟠 차주 진행 예정
4. **나머지 패키지 상품 추가** — 가성비/실속형/올인원/듀얼노출 4개 패키지 등록
5. **추천 상품 가이드 제작** — 별도 랜딩페이지, 상품 선택 도우미 역할

#### 🟡 후순위
6. **엔터프라이즈용 메뉴** — 대형 병원 타겟 맞춤 상담
7. **마케팅 자료 준비** — 비교표, 고객 추천 플로우, 이메일 캠페인 카피

---

## 성공한 접근법 (유지)

### 1. 역산 설계 방식
- 설문 데이터 → 목표 가격 → 패키지 구성 → 단품 조정
- 단품 가격 조정은 10-15% 범위 내

### 2. 디코이 효과 활용
- 단품 가격 유지하여 패키지 구매 유도 (단품합 대비 할인)

### 3. 3단계 구조로 가격대 공백 해소
- 기존: 15만원 → 37만원 (22만원 갭)
- 신규: 87K → 154K → 244K → 279K (자연스러운 계단)

### 4. 교차구매율 데이터 기반 의사결정
- 교차구매율 5% → 면접/채용관 강제 번들 폐지 근거

### 5. 149K/154K 자연 분기
- 올인원(149K 수준)과 실속형(154K)의 5K 차이가 자연 분기 장치로 작동
- 이미지 필요→올인원, 면접 물량 필요→실속형 (UI 분리 불필요)

---

## 실패한 접근법 (반복 금지)

### 1. 단품 가격 기준으로 패키지 설계
- 올바른 순서: 목표 가격 → 역산 → 구성 → 단품 조정

### 2. 개원응원 B안 (189K) 제안 — 절대 반복 금지
- 189K-154K=35K인데 이미지 제작이 29K → 실속형보다 6K 비싸면서 면접 20건 적음
- 사용자 강하게 지적: "우리 서비스 유저들이 병신인줄 아세요?"

### 3. 개원응원 A안 (119K) — 카니발리제이션
- 119K가 되면 실속형(154K) 매출 잠식 + 가성비(87K) 존재 의미 소멸
- 기존 병원도 이미지 필요 없지만 싼 가격 때문에 개원응원 구매하게 됨

### 4. 실속형보다 높은 할인율의 상위 상품
- 듀얼노출을 16%로 제안했으나 "실속형보다 할인율이 더 높은게 웃기다" 지적 → 13%로 조정

---

## 주요 파일/경로

### 필수 참고 파일
1. **상품_전략_최종_확정안.md** — 전체 패키지 라인업, 가격, 구성, 전략 (최종본)
   - `C:\Users\Yujeong Ha\Desktop\클로드코드\상품_전략_최종_확정안.md`
2. **종합_검증_및_개선안.md** — 3개 분석팀 교차검증 결과
   - `C:\Users\Yujeong Ha\Desktop\클로드코드\종합_검증_및_개선안.md`
3. **치크루팅_상품전략_리뉴얼안.md** — 전략 배경/맥락 문서
   - `C:\Users\Yujeong Ha\Desktop\클로드코드\치크루팅_상품전략_리뉴얼안.md`

### 분석 데이터
4. **분석결과.md** — 구매 데이터 분석 (상품별 효과)
5. **설문조사_분석.md** — 101명 설문 분석
6. **면접제안권_이벤트_분석_보고서.md** — 이벤트 효과 분석

### 참고용 이전 버전 (읽지 마세요)
- 상품_가격_전략_최종안.md (초기 버전 2/12, 이후 대체됨)

---

## 새 세션 시작 방법

```
이 파일을 읽고 작업을 이어서 진행해줘:
C:\Users\Yujeong Ha\Desktop\클로드코드\HANDOFF.md

그리고 이 파일도 참고해:
C:\Users\Yujeong Ha\Desktop\클로드코드\상품_전략_최종_확정안.md

골드 채용관 Strapi 등록부터 시작할게.
```

---

## 핵심 포인트 (꼭 기억)

1. **패키지 4개**: 가성비(87K) / 실속형(154K) / 올인원(244K) / 듀얼노출(279K)
2. **할인율 구조**: 13% → 15% → 15% → 13% (가운데 핵심 상품에 할인 집중)
3. **3단계 구조**: VIP(4구좌) → 골드(10구좌) → 프리미엄(16구좌)
4. **골드 채용관**: 마켓 내부 판매, 7일 209K / 14일 357.5K
5. **기존 상품 가격 변경 금지** — 신규 상품만 추가
6. **교차구매율 5%** — 면접/채용관 강제 번들 불필요
7. **주문 데이터**: 922건 (2026.03.11 기준)

---

**작업 인계 완료**
**다음 작업자는 "골드 채용관 Strapi 등록"부터 시작하면 됩니다!**
