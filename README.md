<div align="center">

<img src="logo.svg" alt="TokenRace" width="96" />

# TokenRace

**팀의 Claude Code 토큰 사용량을 F1 레이싱처럼 실시간으로 추적**

[![License](https://img.shields.io/badge/License-MIT-EE695C?style=flat-square)](LICENSE)
[![Go](https://img.shields.io/badge/Go-1.25-00ADD8?style=flat-square&logo=go)](https://go.dev)
[![Next.js](https://img.shields.io/badge/Next.js-16-000?style=flat-square&logo=next.js)](https://nextjs.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Status](https://img.shields.io/badge/Status-Live-EE695C?style=flat-square)](https://tokenrace.baeksang.dev)

<br />

[**🌐 라이브 데모**](https://tokenrace.baeksang.dev)
&nbsp;·&nbsp;
[**⚡ 빠른 시작**](#-빠른-시작-3-step)
&nbsp;·&nbsp;
[**🏗 아키텍처**](#-아키텍처)
&nbsp;·&nbsp;
[**📡 OAuth 가입**](https://tokenrace.baeksang.dev/ko/signup)

<br />

> 팀이 한 달에 Claude Code에 얼마를 쓰는지, 누가 토큰을 가장 많이 태우는지,
> 어떤 모델이 비용 대부분을 차지하는지 — 5초마다 갱신되는 리더보드로 한눈에.

</div>

---

## 🤔 왜 만들었나

```
"이번 주 누가 Sonnet 100만 토큰 넘게 썼지?"        — 모름
"우리 팀 Claude Code 비용 추세 좀 보여줘봐"          — 가공할 데이터 없음
"Opus랑 Sonnet 어느 게 비용 효율 좋아?"             — 감으로 답함
"Bob이 어제 Claude Code 썼나?"                      — Slack DM 보내야 알 수 있음
```

Claude Code는 OpenTelemetry로 모든 메트릭을 뽑아준다. **TokenRace는 그걸 받아서 팀이 볼 수 있게 시각화한다.** 그게 전부.

복잡한 셋업 없음. 결제 페이지 없음. 무료. OSS.

---

## ⚡ 빠른 시작 (3-step)

### 1️⃣ 관리자 — 조직 만들기 (10초)

[**tokenrace.baeksang.dev**](https://tokenrace.baeksang.dev) 접속 → **GitHub로 1초 로그인** → 조직 이름 한 번 입력. 끝.

### 2️⃣ 관리자 — 팀원 초대 (5초/명)

대시보드 → **팀 관리** → 이메일 입력 → **초대 링크 복사** → 슬랙/메신저로 전송.

받은 사람은 같은 GitHub 이메일로 로그인 한 번이면 자동 가입.

### 3️⃣ 팀원 — 에이전트 설치 (1분)

```bash
# 1. 한 줄 설치 (macOS / Linux × arm64 / amd64 자동 감지)
curl -sSL https://tokenrace.baeksang.dev/install.sh | sh

# 2. 인터랙티브 init (브라우저 자동 오픈 + 키 페이스트 안내)
token-race init
```

다음 Claude Code 세션이 끝나면 **자동으로** 대시보드 리더보드에 등록됨. 이후 영구 자동 동기화.

---

## ✨ 기능

| | 기능 | 설명 |
|--|------|------|
| 🏎 | **F1 스타일 리더보드** | 드라이버(개인) ↔ 컨스트럭터(팀) 토글, 5/60초 폴링, FL/포디움/델타 |
| ⚡ | **Zero-config 수집** | Claude Code Stop Hook 자동 등록 → 매 세션 종료 시 OTel 메트릭 전송 |
| 🏢 | **3-tier 멀티테넌시** | Organization → Team → User + RBAC (org_owner / team_manager / member) |
| 📨 | **이메일 기반 ingest** | UUID 안 쓰고 user_email만 보내면 API key의 org 범위 안에서 자동 매핑 |
| 🎫 | **GitHub OAuth-only** | 비밀번호 없음. GitHub 이메일/이름/아바타만 사용, 저장소 접근 X |
| 🔗 | **256-bit 초대 토큰** | 14일 유효, 1회용, 이메일 미스매치 자동 거부 |
| 📊 | **Single-CTE 통계** | 6개 KPI를 1개 PostgreSQL 쿼리로 (UNION ALL + kind 분기) |
| 🔥 | **Burn-rate 실시간** | 24h 토큰 소비율 차트 + 모델별 분배 시각화 |
| 🌐 | **i18n** | 한국어 / English (next-intl, 200+ 키) |
| 🛡 | **API key SHA-256** | Plaintext 키는 발급 시 1회만 표시, DB는 해시만 저장 |

---

## 🏗 아키텍처

```
┌────── 개발자 PC ──────┐                       ┌──── 팀 대시보드 ────┐
│                       │                       │                    │
│  $ claude (세션)      │                       │  Next.js 16 + RQ   │
│        ↓              │                       │  Vercel            │
│  ~/.claude/...jsonl   │                       │  F1 리더보드 UI    │
│        ↓              │                       │  ↑                 │
│  Stop Hook            │                       │  GET /v1/orgs/:id  │
│        ↓              │                       │  /leaderboard      │
│  token-race send      │   POST /v1/ingest     │  (5s/60s 캐시)     │
│  (Go binary, 12MB)    │   Bearer tr_xxx       │                    │
│        └──────────────┼───────────────────────┘                    │
└───────────────────────┘            │                               │
                                     ▼                               │
                        ┌────────────────────────┐                   │
                        │  Go + Fiber API        │◄──────────────────┤
                        │  Railway               │                   │
                        │  ─────────────────     │                   │
                        │  PostgreSQL 16         │                   │
                        │  • usage_events        │                   │
                        │    (월별 RANGE 파티션) │                   │
                        │  • Redis cache (5/60s) │                   │
                        │                        │                   │
                        │  In-process 워커 ─────┐│                   │
                        │  • hourly rollup      ││                   │
                        │  • daily rollup       ││                   │
                        │  • partitions +3개월  ││                   │
                        └───────────────────────┴┘                   │
                                                                     │
                                                                     │
                                              ┌──────────────────────┘
                                              │
                                       GitHub OAuth (web)
```

**핵심 설계 결정:**
- **In-process 워커** — 별도 worker 서비스 X, API 프로세스 안에서 goroutine ticker로 실행 (cron 1개)
- **월별 RANGE 파티션** — 일 1500만 events × 12개월 = 1.8억 row를 sub-second 쿼리로 cover
- **Single CTE** — KPI 6개를 1쿼리로 묶어서 PostgreSQL round-trip 6→1
- **Redis 캐시 (5s/60s)** — 리더보드 폴링이 DB 직격 안 하도록 + 데모 응답은 캐시 우회 (실데이터 들어오면 즉시 전환)
- **Cross-language parity** — `assignRanks` (TS) ↔ `AssignRanks` (Go), 동일 JSON fixture로 양쪽 테스트 → drift 시 CI fail

---

## 🚀 라이브 환경

| | 위치 |
|--|------|
| **웹** | [tokenrace.baeksang.dev](https://tokenrace.baeksang.dev) (Vercel) |
| **API** | [tokenrace-production.up.railway.app](https://tokenrace-production.up.railway.app) (Railway) |
| **에이전트** | [GitHub Releases](https://github.com/Mrbaeksang/tokenrace/releases?q=agent-v) |
| **OAuth** | GitHub Apps (저장소 접근 권한 X, profile + email만) |

---

## 💻 로컬 개발

```bash
git clone https://github.com/Mrbaeksang/tokenrace.git
cd tokenrace

cp .env.example .env       # DATABASE_URL, REDIS_URL, JWT_SECRET, GITHUB_OAUTH_*
make dev                   # docker-compose: postgres + redis + api + web
# → API   http://localhost:8080
# → Web   http://localhost:3000
```

**필요:** Go 1.25+ · Node.js 22+ · Docker · PostgreSQL 16 · Redis 7

---

## 📦 기술 스택

<table>
<tr>
<td align="center" width="120">
<img src="https://skillicons.dev/icons?i=go" width="48" /><br/>
<strong>Go 1.25</strong><br/>
<sub>Agent + API</sub>
</td>
<td align="center" width="120">
<img src="https://skillicons.dev/icons?i=nextjs" width="48" /><br/>
<strong>Next.js 16</strong><br/>
<sub>App Router · RSC</sub>
</td>
<td align="center" width="120">
<img src="https://skillicons.dev/icons?i=postgresql" width="48" /><br/>
<strong>PostgreSQL 16</strong><br/>
<sub>월별 파티션</sub>
</td>
<td align="center" width="120">
<img src="https://skillicons.dev/icons?i=redis" width="48" /><br/>
<strong>Redis 7</strong><br/>
<sub>5s/60s 캐시</sub>
</td>
<td align="center" width="120">
<img src="https://skillicons.dev/icons?i=tailwind" width="48" /><br/>
<strong>Tailwind 4</strong><br/>
<sub>CSS-first</sub>
</td>
</tr>
</table>

| 레이어 | 상세 |
|--------|------|
| **Agent** (Go) | cobra · JSONL parser · Claude email 자동 감지 · Stop hook 자동 등록 · 인터랙티브 init |
| **API** (Go + Fiber v2) | golang-migrate · pgx/v5 · go-redis/v9 · validator/v10 · zerolog · in-process rollup ticker |
| **Web** (Next.js) | TanStack Query · Radix UI · next-intl · Recharts · Lucide · `useTranslations()` 200+ 키 |
| **Auth** | GitHub OAuth (HS256 JWT, sub + is_super_admin 만 claim) — 비밀번호 0개, 세션 0개 |
| **CI/CD** | GitHub Actions (matrix go build × 4) → Releases · Vercel auto-deploy on push · Railway nixpacks |

---

## 🔑 API 레퍼런스 (자주 쓰는 것만)

```http
POST /v1/ingest/batch                                    # 에이전트 → 토큰 이벤트 수집
GET  /v1/orgs/:org_id/leaderboard?range=today&group=driver  # F1 리더보드
GET  /v1/orgs/:org_id/sessions/active?since=5m           # 실시간 활성 세션
GET  /v1/orgs/:org_id/sessions/history?model=&limit=50   # 커서 기반 세션 검색
GET  /v1/orgs/:org_id/models/breakdown?from=&to=         # 모델별 토큰/비용 집계
GET  /v1/orgs/:org_id/usage?from=&to=                    # 일별 사용량 시계열

POST /v1/orgs/:org_id/api-keys                           # 키 발급 (plaintext 1회만 응답)
GET  /v1/orgs/:org_id/api-keys                           # 활성 키 목록 (해시만)
DELETE /v1/orgs/:org_id/api-keys/:id                     # 폐기

POST /v1/orgs/:org_id/invitations                        # 초대 발급 (256-bit token)
GET  /v1/invitations/:token                              # 초대 미리보기 (PUBLIC)
POST /v1/invitations/:token/accept                       # 초대 수락 (JWT)

GET  /v1/auth/github                                     # OAuth 시작
GET  /v1/auth/github/callback                            # OAuth 콜백 (web으로 redirect + JWT fragment)
```

---

## 🧪 테스트

```bash
# 백엔드
cd api && go vet ./... && go test -race ./...

# 통합 (PostgreSQL 필요)
TEST_DATABASE_URL=postgres://... go test -tags=integration ./internal/repository/...

# 프론트엔드
cd web && npm test -- --run    # vitest (Go↔TS parity 포함)
npx tsc --noEmit               # 타입 체크
npm run build                  # Next.js 프로덕션 빌드
```

**Cross-language parity:**
`api/internal/domain/leaderboard/testdata/ranking_fixtures.json` 한 파일을 Go와 TS 양쪽 테스트가 공유. 한쪽 알고리즘만 바뀌면 fixture 불일치 → CI fail. 드리프트 0.

---

## 🤝 기여하기

PR / 이슈 환영. 먼저 [`CONTRIBUTING.md`](CONTRIBUTING.md) 읽어보길.

```bash
# PR 체크리스트
cd api && go vet ./... && go test -race ./...
cd web && npx tsc --noEmit && npm test -- --run && npm run build
```

---

## 📈 Star History

<div align="center">

[![Star History Chart](https://api.star-history.com/svg?repos=Mrbaeksang/tokenrace&type=Date&theme=dark)](https://star-history.com/#Mrbaeksang/tokenrace&Date)

</div>

---

## 📄 라이선스

[MIT License](LICENSE) — 자유롭게 사용, 수정, 상업적 사용 가능.

---

<div align="center">

**TokenRace** · [tokenrace.baeksang.dev](https://tokenrace.baeksang.dev) · Made by [@Mrbaeksang](https://github.com/Mrbaeksang)

<sub>Built with Claude Code, dogfooded daily.</sub>

</div>
