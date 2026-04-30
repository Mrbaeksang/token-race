<div align="center">

<img src="logo.svg" alt="TokenRace Logo" width="80" />

# TokenRace 🏎

**팀의 Claude Code 토큰 사용량을 F1 레이싱처럼 실시간으로 추적하세요**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![Go Version](https://img.shields.io/badge/Go-1.23-00ADD8?style=flat-square&logo=go)](https://go.dev)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?style=flat-square&logo=next.js)](https://nextjs.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Deploy on Railway](https://img.shields.io/badge/Deploy-Railway-7B2FBE?style=flat-square&logo=railway)](https://railway.app)
[![Vercel](https://img.shields.io/badge/Vercel-Deployed-black?style=flat-square&logo=vercel)](https://vercel.com)

<br />

[**🌐 데모 보기**](https://tokenrace.baeksang.dev) &nbsp;·&nbsp;
[**⚡ 빠른 시작**](#-빠른-시작) &nbsp;·&nbsp;
[**📖 문서**](docs/) &nbsp;·&nbsp;
[**English**](README.en.md)

<br />

<!-- 배포 후 실제 대시보드 GIF로 교체 -->
<img src="docs/assets/demo.gif" alt="TokenRace F1 Ranking Dashboard" width="720" style="border-radius:12px;border:1px solid #374151" />

</div>

---

## 왜 TokenRace인가?

팀에서 Claude Code를 쓰다 보면 이런 일이 생깁니다.

> 💬 *"이번 달 누가 $200 썼어요?" "나는 $100 요금제에서 10%도 못 씀" "토큰 낭비 어떻게 줄이지?"*

**TokenRace는 이 질문에 실시간으로 답합니다.**

팀원 PC에 에이전트 하나만 설치하면, Claude Code 세션이 끝날 때마다 자동으로 토큰 사용량을 수집해서 F1 스타일 랭킹 대시보드에 표시합니다. 별도 설정 없이.

---

## ✨ 주요 기능

| | 기능 | 설명 |
|--|------|------|
| 🏆 | **F1 실시간 랭킹** | 팀원별 토큰 소비량 포디움 시각화, 30초마다 자동 갱신 |
| ⚡ | **Zero-config 수집** | Stop Hook → 세션 종료 시 자동 전송. `--user-id` 플래그 불필요 |
| 🏢 | **멀티테넌시** | Organization → Team → Member 계층 + RBAC (org_owner/manager/member) |
| 💳 | **Per-seat 결제** | Stripe 연동, **$1/멤버/월**, 멤버 추가 시 자동 과금 조정 |
| 📊 | **비용 분석** | 일별/주별/월별 토큰 트렌드 + USD 비용 자동 산출 + Recharts 시각화 |
| 🔗 | **초대 링크 방식** | 이메일 없이 1회용 초대 링크로 팀원 온보딩 |
| 🌐 | **다국어** | 한국어 / English (next-intl) |
| 🔒 | **보안** | API 키 SHA-256 해시 저장, JWT RBAC, SameSite 쿠키 |

---

## 🏗 아키텍처

```
┌──────────────── 개발자 PC ─────────────────┐
│                                             │
│  $ claude  →  ~/.claude/projects/**.jsonl  │
│                        │                   │
│               [Stop Hook 자동 실행]         │
│                        │                   │
│              token-race send               │
└────────────────────────┼───────────────────┘
                         │  POST /v1/ingest
                         │  Bearer <API_KEY>
                         ▼
              ┌─────────────────────┐
              │   Go + Fiber API    │◄── Stripe Webhook
              │   Railway           │
              │   PostgreSQL 16     │
              │   (월별 파티션)     │
              └──────────┬──────────┘
                         │  GET /ranking (30s poll)
                         ▼
              ┌─────────────────────┐
              │   Next.js 16        │
              │   Vercel            │
              │   🏎 F1 랭킹 UI    │
              └─────────────────────┘
```

---

## 🚀 빠른 시작

### 팀원 — 에이전트 설치

```bash
# 1. 설치 (macOS / Linux)
curl -sSL https://tokenrace.baeksang.dev/install.sh | sh

# 2. 초기화 (관리자에게 API Key 받기)
token-race init --api-key sk_tr_xxxxxxxxxxxx

# 3. 확인
token-race send --dry-run
# ✓ 이후 Claude Code를 쓰면 세션 종료 시 자동 전송됩니다
```

### 관리자 — 팀 등록

1. [tokenrace.baeksang.dev/onboarding](https://tokenrace.baeksang.dev/onboarding) 접속
2. 팀 규모 선택 → Stripe 결제 ($1/멤버/월)
3. 조직 생성 완료 → 팀원에게 초대 링크 발송
4. 팀원이 초대 수락 → 에이전트 설치 → 🏁 완료

### 개발 환경

```bash
git clone https://github.com/Mrbaeksang/tokenrace.git
cd tokenrace

cp .env.example .env      # 환경변수 설정

make dev                  # Docker로 전체 실행
# → API  http://localhost:8080
# → Web  http://localhost:3000
```

**필요 환경:** Go 1.23+ · Node.js 22+ · Docker · Stripe CLI

---

## 📸 스크린샷

<table>
<tr>
<td width="50%">
<img src="docs/assets/screenshot-ranking.png" alt="팀 랭킹" />
<p align="center"><sub>🏆 F1 스타일 실시간 팀 랭킹</sub></p>
</td>
<td width="50%">
<img src="docs/assets/screenshot-dashboard.png" alt="개인 대시보드" />
<p align="center"><sub>📊 개인 사용량 분석 + 비용 추이</sub></p>
</td>
</tr>
<tr>
<td width="50%">
<img src="docs/assets/screenshot-org.png" alt="조직 관리" />
<p align="center"><sub>🏢 팀 / 멤버 관리 + 초대 링크</sub></p>
</td>
<td width="50%">
<img src="docs/assets/screenshot-onboarding.png" alt="온보딩" />
<p align="center"><sub>💳 Stripe 결제 → 즉시 조직 생성</sub></p>
</td>
</tr>
</table>

---

## 📦 기술 스택

<table>
<tr>
<td align="center" width="110">
<img src="https://skillicons.dev/icons?i=go" width="40" /><br/>
<strong>Go 1.23</strong><br/>
<sub>Agent + API</sub>
</td>
<td align="center" width="110">
<img src="https://skillicons.dev/icons?i=nextjs" width="40" /><br/>
<strong>Next.js 16</strong><br/>
<sub>Dashboard</sub>
</td>
<td align="center" width="110">
<img src="https://skillicons.dev/icons?i=postgresql" width="40" /><br/>
<strong>PostgreSQL</strong><br/>
<sub>월별 파티션</sub>
</td>
<td align="center" width="110">
<img src="https://skillicons.dev/icons?i=tailwind" width="40" /><br/>
<strong>Tailwind v4</strong><br/>
<sub>CSS-first</sub>
</td>
<td align="center" width="110">
<img src="https://skillicons.dev/icons?i=stripe" width="40" /><br/>
<strong>Stripe</strong><br/>
<sub>Per-seat 결제</sub>
</td>
</tr>
</table>

| 레이어 | 상세 |
|--------|------|
| **Agent** | Go · cobra · JSONL 파싱 · `claude auth status` 이메일 자동 감지 |
| **API** | Fiber v2 · golang-migrate · SHA-256 API Key · JWT · Rate Limit 100/min |
| **Web** | React Query · Radix UI · CVA · Recharts · react-hook-form + Zod |
| **Infra** | Railway (API + PostgreSQL) · Vercel (Web) · GitHub Actions CI |

---

## 📈 Star History

<div align="center">

[![Star History Chart](https://api.star-history.com/svg?repos=Mrbaeksang/token-race&type=Date&theme=dark)](https://star-history.com/#Mrbaeksang/token-race&Date)

</div>

---

## 🔑 API 레퍼런스

```
POST /v1/ingest                              # 토큰 수집 (Agent → API)
GET  /v1/orgs/:id/teams/:tid/ranking        # 팀 랭킹
GET  /v1/orgs/:id/usage/me                  # 내 사용량 통계
POST /v1/orgs/:id/teams/:tid/invites        # 멤버 초대
POST /v1/auth/login                          # 로그인
POST /v1/auth/invite/accept                  # 초대 수락
POST /v1/payments/checkout                   # Stripe 결제 시작
```

전체 스펙: [`docs/api-spec.yaml`](docs/api-spec.yaml)

---

## 🤝 기여하기

PR과 이슈를 환영합니다!

```bash
# 이슈: https://github.com/Mrbaeksang/tokenrace/issues

# PR 체크리스트
go vet ./... && go test ./... -race    # API / Agent
npm run lint && npm run build          # Web
```

자세한 내용: [`CONTRIBUTING.md`](CONTRIBUTING.md)

---

## 📄 라이선스

[MIT License](LICENSE) — 자유롭게 사용, 수정, 배포하세요.

---

<div align="center">

**TokenRace** · [tokenrace.baeksang.dev](https://tokenrace.baeksang.dev) · Made with ❤️

<sub>Claude Code를 더 스마트하게 쓰는 팀을 위해</sub>

</div>
