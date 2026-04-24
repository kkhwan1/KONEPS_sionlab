# KONEPS_sionlab

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![validate](https://github.com/kkhwan1/KONEPS_sionlab/actions/workflows/validate.yml/badge.svg)](https://github.com/kkhwan1/KONEPS_sionlab/actions/workflows/validate.yml)
[![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-6B4FBB)](https://modelcontextprotocol.io)
[![Release](https://img.shields.io/github/v/release/kkhwan1/KONEPS_sionlab)](https://github.com/kkhwan1/KONEPS_sionlab/releases)

> 🇰🇷 **나라장터(KONEPS/G2B) 공공 조달 데이터**를 Claude 로 자연어 질의할 수 있게 해주는 **MCP 연결 워크스페이스**.
> Claude Code / Claude Desktop 양쪽에서 동일하게 작동하며, API 키는 `.env` 한 곳에서만 관리합니다.

> **English** · Query Korean government procurement data via MCP. Works with both Claude Code and Claude Desktop. API keys stay in a local `.env`; the `.mcp.json` is key-free.

**작성자 · Author**: KKHWAN · ✉ [lee.kkhwan@gmail.com](mailto:lee.kkhwan@gmail.com)

<sub>🔎 **키워드** · 나라장터 · 조달청 · 국가종합전자조달 · 공공조달 · 공공데이터포털 · 입찰공고 · 낙찰정보 · 계약정보 · KONEPS · G2B · data.go.kr · MCP · Model Context Protocol · Claude · Claude Code · Claude Desktop · Korea public procurement · Korean government procurement API</sub>

---

## 📑 목차
1. [이게 뭐예요?](#-이게-뭐예요)
2. [누가 쓰면 좋은가](#-누가-쓰면-좋은가)
3. [작동 구조](#-작동-구조)
4. [빠른 시작 (3분)](#-빠른-시작-3분)
5. [공공데이터포털에서 API 키 받기](#-공공데이터포털에서-api-키-받기)
6. [Claude Code 에서 쓰기](#-claude-code-에서-쓰기)
7. [Claude Desktop 에서 쓰기](#-claude-desktop-에서-쓰기)
8. [제공되는 MCP Tool](#-제공되는-mcp-tool)
9. [프로젝트 구조](#-프로젝트-구조)
10. [에이전트 팀](#-에이전트-팀-claude-code-전용)
11. [트러블슈팅](#-트러블슈팅)
12. [로드맵](#-로드맵)
13. [기여/라이선스](#-기여--라이선스)

---

## 🎯 이게 뭐예요?

**한 줄 설명** — Claude 대화창에 *"AI 관련 입찰공고 찾아줘"* 라고 입력하면, Claude 가 **나라장터 공식 API** 를 호출해 실시간으로 공고 목록을 가져오는 환경입니다.

**기존 방식**:
```
공공데이터포털 → API 문서 읽기 → 코드 작성 → 파라미터 디버깅 → 응답 파싱 ...
```

**이 프로젝트로**:
```
Claude 대화창: "나라장터에서 'AI' 관련 공고 찾아줘"
              ↓ (MCP 경유 자동 호출)
              ← 공고번호, 기관, 예산, 마감일 포함한 목록 반환
```

---

## 👥 누가 쓰면 좋은가

| 사용자 유형 | 도구 | 얻는 가치 |
|---|---|---|
| **개발자/엔지니어** | Claude Code | MCP + 에이전트 팀 + 스키마 설계 + 검증 파이프라인 풀스택 |
| **비개발 실무자** (기획/영업/입찰 담당) | Claude Desktop | 대화창에서 자연어 한 줄로 공고 검색/요약 |
| **분석가** | Desktop + Excel | MCP 응답을 복사해 가공 |
| **팀 리드** | 양쪽 모두 | Code 로 데이터 파이프라인 구축 → Desktop 으로 팀원 배포 |

---

## 🏗️ 작동 구조

### 전체 아키텍처

```
┌───────────────────────────────────────────────────────────────┐
│  사용자 대화창                                                 │
│  "나라장터에서 AI 입찰공고 찾아줘"                              │
└──────────────────────────┬────────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────────┐
│  Claude (Code 또는 Desktop)                                    │
│  - 요청 의도 파악                                              │
│  - 적절한 MCP tool 선택 (get_bids_by_keyword)                  │
└──────────────────────────┬────────────────────────────────────┘
                           │ JSON-RPC over stdio
                           ▼
┌───────────────────────────────────────────────────────────────┐
│  scripts/run-mcp.sh (래퍼)                                     │
│  - .env 에서 API 키 로드                                       │
│  - uvx 로 MCP 서버 on-demand 기동                              │
└──────────────────────────┬────────────────────────────────────┘
                           │
          ┌────────────────┴───────────────┐
          ▼                                ▼
┌───────────────────────┐    ┌───────────────────────────────┐
│  nara-jangteo         │    │  data-go-mcp.pps-narajangteo  │
│  (Datajang)           │    │  (Koomook)                    │
│  • 키워드 검색         │    │  • 입찰공고 원시             │
│  • 부서 추천           │    │  • 낙찰 정보                 │
│  • RFP 파일 추출       │    │  • 계약 정보                 │
└──────────────────────┬┘    └──────────┬────────────────────┘
                       │                │
                       └────┬───────────┘
                            ▼
              ┌─────────────────────────────┐
              │  data.go.kr 공공데이터포털   │
              │  (조달청 나라장터 공식 API)  │
              └─────────────────────────────┘
```

### 4개 핵심 설계 결정

1. **`.mcp.json` 에 API 키 0개**
   래퍼 스크립트(`scripts/run-mcp.sh`)가 `.env` 에서 키를 읽어 런타임에 주입. 키 교체는 `.env` 한 곳만 수정.

2. **프로젝트 스코프 한정**
   글로벌 `~/.claude.json` 을 절대 건드리지 않음. 다른 프로젝트에 영향 없음.

3. **`uvx` on-demand**
   MCP 서버를 글로벌 설치하지 않음. 세션 시작 시 PyPI 에서 pull. 캐시된 뒤엔 즉시 기동.

4. **에이전트 팀 분리**
   메인 Claude 는 직접 호출, `mcp-operator` 서브에이전트는 대량/반복 호출 담당. 다른 에이전트는 MCP 접근 불가로 격리.

---

## ⚡ 빠른 시작 (3분)

### 사전 요구사항
- `uvx` 0.10+ — `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Claude Code CLI **또는** Claude Desktop 앱
- `data.go.kr` 인증키 ([받는 법 ↓](#-공공데이터포털에서-api-키-받기))

### 3줄 설치
```bash
git clone https://github.com/kkhwan1/KONEPS_sionlab.git
cd KONEPS_sionlab
cp .env.example .env   # .env 열어서 본인 키 붙여넣기
```

### 각자 환경에 맞게
```bash
# Claude Code 사용자
claude                                # 프로젝트 MCP 승인 프롬프트 수락

# Claude Desktop 사용자 (자동 설치)
bash scripts/setup-claude-desktop.sh  # Desktop 재시작 필요
```

끝. Claude 에 *"나라장터에서 AI 검색해줘"* 입력하면 작동합니다.

---

## 🔑 공공데이터포털에서 API 키 받기

### 왜 필요한가
나라장터 데이터는 **공공데이터포털(`data.go.kr`)** 을 통해 제공됩니다. 무료이지만 **사용자 식별용 인증키** 가 필요합니다.

### 한눈에 보는 절차

```
① 포털 회원가입 → ② API 페이지 [활용신청] 클릭 → ③ 자동승인 즉시 → ④ 마이페이지에서 키 복사
```

### 화면으로 따라가기

**① 공공데이터포털 메인** — <https://www.data.go.kr/>

![공공데이터포털 메인](./docs/images/00-portal-main.png)

우상단 **회원가입** → 본인인증 → 가입 완료.

**② 나라장터 API 페이지** — <https://www.data.go.kr/data/15129394/openapi.do>

![API 상세 페이지](./docs/images/01-api-detail-page.png)

확인할 메타 정보:

| 항목 | 값 | 의미 |
|---|---|---|
| 비용 | **무료** | 💰 돈 안 듦 |
| 심의유형 | **자동승인** | ⚡ 즉시 승인 |
| 트래픽 | 개발계정 **1,000/일** | 초기 충분 |

**③ 우상단 파란 [활용신청] 버튼 클릭**

![활용신청 버튼](./docs/images/02-apply-button.png)

신청서에 활용목적 한 줄 작성(예: "조달 데이터 분석 MCP 도구") → 제출 → **자동승인**.

**④ 마이페이지에서 키 확인**

`마이페이지 → 오픈API → 개발계정 → 신청한 서비스 클릭` 에서 **"일반 인증키 (Decoding)"** 복사.

> ⚠️ **반드시 Decoding 키를 쓰세요.** Encoding 키는 `%2B`, `%2F` 가 들어있어 이중 인코딩으로 401 에러가 납니다.

**⑤ 프로젝트 `.env` 에 붙여넣기**
```bash
NARA_API_KEY=abc/def+ghi==...     # 복사한 Decoding 키
NARA_PRESPEC_API_KEY=abc/def+...  # 동일 키로 우선 시도
chmod 600 .env                    # 권한 제한
```

📖 **상세 가이드**: [`docs/data-portal-guide.md`](./docs/data-portal-guide.md) — 트러블슈팅 포함

---

## 💻 Claude Code 에서 쓰기

### 실행
```bash
cd KONEPS_sionlab
claude
```
첫 기동 시 *"Trust this project's MCP servers?"* 프롬프트가 뜹니다. **Yes** 선택.

### 사용 예시
```
> 나라장터에서 "AI" 관련 최근 7일 입찰공고 찾아줘

> "한국연구재단"에서 공고한 최근 용역 입찰 목록 정리
  → 공고번호, 예산, 마감일만 표로

> 다음 PDF 를 RFP 로 분석해 핵심 요구사항 추출:
  [제안요청서]AI기반 벤처펀드 모니터링 시스템 구축.pdf
```

### 에이전트 팀 활용
대량 데이터 수집이나 스키마 설계처럼 복잡한 작업은 에이전트에 위임:
```
> orchestrator 에게 위임: 최근 한 달간 "플랫폼" 키워드 공고 전부 수집 후
  기관별 예산 합계 리포트
```

---

## 🖥️ Claude Desktop 에서 쓰기

### 자동 설치 (권장)
```bash
bash scripts/setup-claude-desktop.sh
```
- OS 자동 감지 (macOS/Windows/WSL)
- 기존 MCP 서버 설정은 **병합** (덮어쓰지 않음)
- 타임스탬프 백업 자동 생성

이후 **Claude Desktop 완전 종료 → 재시작**.

### 사용 예시
Desktop 대화창에서 자연어로 요청:
```
나라장터에서 이번 주 공고 중에 "데이터 플랫폼" 관련 3개만 요약해줘
```

### 한계점
Desktop 에는 서브에이전트 개념이 없습니다. 대량 수집·스키마 설계 같은 복잡한 워크플로우는 Claude Code 를 사용하세요.

세부 가이드: [`docs/claude-desktop-setup.md`](./docs/claude-desktop-setup.md)

---

## 🛠️ 제공되는 MCP Tool

### `nara-jangteo` (Datajang 제공)
| Tool | 용도 |
|---|---|
| `get_bids_by_keyword` | 키워드로 최근 7일 입찰공고 + 사전규격 동시 검색 |
| `recommend_bids_for_dept` | 부서 프로필에 맞춰 공고 추천 (최대 60건) |
| `analyze_bid_detail` | HWP/PDF/DOCX 첨부파일에서 RFP 텍스트 추출 |

### `data-go-mcp.pps-narajangteo` (Koomook 제공)
| Tool | 용도 |
|---|---|
| `search_bid_announcements` | 입찰공고 원시 조회 (최대 1개월) |
| `get_bid_detail` | 공고번호로 상세 조회 ⚠️ *업스트림 버그, 현재 실패* |
| `search_successful_bids` | 낙찰정보 조회 (용역/물품/공사 구분) |
| `search_contracts` | 계약정보 조회 (기관/기간 필터) |

실제 시그니처/파라미터: [`docs/mcp-tools.md`](./docs/mcp-tools.md)

---

## 📂 프로젝트 구조

```
KONEPS_sionlab/
│
├── .mcp.json                  ← Claude Code 가 자동 로드 (키 0개)
├── .env.example               ← 환경변수 템플릿 (git 포함)
├── .env                       ← 실제 키 (git 제외, 직접 생성)
│
├── scripts/
│   ├── run-mcp.sh             ← MCP 기동 래퍼 (Code/Desktop 공용)
│   └── setup-claude-desktop.sh ← Desktop 설정 자동 병합기
│
├── .claude/
│   └── agents/                ← 에이전트 팀 정의 (Claude Code 전용)
│       ├── orchestrator.md
│       ├── mcp-operator.md    ← MCP tool 호출 권한 보유
│       ├── data-modeler.md
│       ├── verifier.md
│       └── docs-writer.md
│
├── docs/
│   ├── data-portal-guide.md   ← 공공데이터포털 상세 가이드
│   ├── mcp-tools.md           ← tool 카탈로그 (실측 시그니처)
│   ├── claude-desktop-setup.md ← Desktop 설치 세부
│   ├── claude-desktop-config.template.json
│   └── images/                ← 포털 스크린샷
│
├── data/
│   └── raw/                   ← MCP 원시 응답 캐시 (git 제외)
│       └── README.md          ← 명명 규칙
│
├── db/                        ← (2단계) SQLite 스키마 예정
│
├── .github/
│   ├── workflows/validate.yml ← CI: JSON/shell 검증, 키 누출 점검
│   ├── ISSUE_TEMPLATE/
│   └── PULL_REQUEST_TEMPLATE.md
│
├── CLAUDE.md                  ← Claude Code 프로젝트 지침
├── README.md                  ← 이 파일
├── LICENSE                    ← MIT
├── CONTRIBUTING.md            ← 기여 가이드
├── SECURITY.md                ← 보안 정책
├── CODE_OF_CONDUCT.md         ← 행동 강령
└── CHANGELOG.md               ← 버전 변경 이력
```

---

## 🤖 에이전트 팀 (Claude Code 전용)

`.claude/agents/` 에 정의된 5개 서브에이전트:

| 에이전트 | 역할 | MCP tool 권한 |
|---|---|---|
| `orchestrator` | 사용자 요청 분해/라우팅 | ❌ |
| `mcp-operator` | **실제 MCP 호출 담당** | ✅ 9개 tool |
| `data-modeler` | MCP 응답 → SQLite DDL 설계 | ❌ |
| `verifier` | 완료 주장 전 증거 수집 | ❌ |
| `docs-writer` | 문서/런북 유지 | ❌ |

메인 Claude 가 사용자 요청을 받으면 → `orchestrator` 에게 작업 분해 위임 → `mcp-operator` 가 MCP 호출 → `verifier` 가 응답 검증 → 결과 통합.

세부 운영 규칙: [`CLAUDE.md`](./CLAUDE.md)

---

## 🩺 트러블슈팅

| 증상 | 원인 / 조치 |
|---|---|
| **401/403 응답** | `data.go.kr` 활용신청 승인 대기(1~2시간) 또는 `.env` 에 키 미입력. Encoding 키 대신 Decoding 키 사용 확인 |
| **사전규격 403** | `NARA_PRESPEC_API_KEY` — 해당 API 별도 활용신청 필요할 수 있음 |
| **`uvx` hang** | 네트워크 또는 `UV_LINK_MODE=copy` 누락. `uvx --refresh` 로 재시도 |
| **tool 목록에 MCP 없음** | `.mcp.json` 문법 오류 또는 Claude Code/Desktop 재시작 안 됨. `python3 -m json.tool < .mcp.json` 로 검증 |
| **서브에이전트 "tool not available"** | `.claude/agents/{name}.md` frontmatter `tools:` 에 MCP tool 누락. 수정 후 Claude Code 세션 재시작 |
| **`get_bid_detail` 실패** | 업스트림 버그. `search_bid_announcements` 로 대체 조회 |
| **일일 호출 한도 초과** | 개발계정 1,000/일. `data/raw/` 캐시 먼저 확인 또는 운영계정 승인 |

더 자세한 진단: [`docs/data-portal-guide.md#자주-발생하는-문제`](./docs/data-portal-guide.md#5-자주-발생하는-문제) · [`docs/claude-desktop-setup.md`](./docs/claude-desktop-setup.md)

---

## 🗺️ 로드맵

| 단계 | 상태 | 내용 |
|---|---|---|
| **1단계 — MCP 연결** | ✅ 완료 (v0.1.0) | 공개 MCP 2개 안정화 + Desktop 지원 |
| **2단계 — 스키마/ETL** | 🚧 계획 | `data/raw/` → SQLite DDL + 정규화 로더 |
| **3단계 — 자체 MCP** | 📋 구상 | `sionlab-mcp`: 비즈니스 로직 + 자사 DB 조인 |

---

## 🤝 기여 / 라이선스

- **기여 환영**: [CONTRIBUTING.md](./CONTRIBUTING.md)
- **보안 이슈**: [SECURITY.md](./SECURITY.md) (공개 이슈 대신 비공개 제보)
- **행동 강령**: [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- **변경 이력**: [CHANGELOG.md](./CHANGELOG.md)
- **라이선스**: [MIT](./LICENSE)

### 크레딧
- [Datajang/narajangteo_mcp_server](https://github.com/Datajang/narajangteo_mcp_server) — `nara-jangteo` MCP 원본
- [Koomook/data-go-mcp-servers](https://github.com/Koomook/data-go-mcp-servers) — `data-go-mcp.pps-narajangteo` MCP 원본
- [공공데이터포털 / 조달청](https://www.data.go.kr/) — 나라장터 API 제공

### 스크린샷 출처
본 README 의 포털 스크린샷은 [공공데이터포털](https://www.data.go.kr/) 2026-04-24 버전에서 캡처한 것이며, **교육 목적 이용허락 범위 내에서 사용**합니다. 포털 UI 변경 시 오래될 수 있습니다.
