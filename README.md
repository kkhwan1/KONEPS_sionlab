# KONEPS_sionlab

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![validate](https://github.com/kkhwan1/KONEPS_sionlab/actions/workflows/validate.yml/badge.svg)](https://github.com/kkhwan1/KONEPS_sionlab/actions/workflows/validate.yml)
[![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-6B4FBB)](https://modelcontextprotocol.io)

나라장터(KONEPS/G2B) 공공 조달 데이터를 **MCP(Model Context Protocol) 경유**로 조회하는 Claude Code / Claude Desktop 공용 워크스페이스.

> **English summary** · Query Korean government procurement data (KONEPS/G2B) via MCP — works with both Claude Code (with agent team) and Claude Desktop (tool-only mode). Public MCP servers are fetched on-demand by `uvx`. API keys live in a local `.env` and are never committed.

## ✨ Highlights
- 🔐 **0 API keys in `.mcp.json`** — single source `.env` via wrapper script
- 🖥️ **Dual client support** — Claude Code (full agent workflow) + Claude Desktop (tool-only)
- 🤖 **Agent team included** — orchestrator / mcp-operator / data-modeler / verifier / docs-writer
- 📦 **Reproducible** — `uvx` on-demand, project-scope config, no global pollution
- 🇰🇷 **Korean first** docs with English summaries

## 사전 요구사항
- `uvx` 0.10 이상 (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- [공공데이터포털](https://www.data.go.kr/) 인증키 (조달청 나라장터 입찰공고정보서비스 활용신청)
- Claude Code CLI

## 초기 설정 (3단계)
```bash
# 1. 환경변수 파일 준비
cp .env.example .env
# 2. .env 에 data.go.kr 인증키(디코딩 원본) 입력
#    NARA_API_KEY=...
# 3. Claude Code 를 이 디렉토리에서 기동 → 프로젝트 MCP 승인
claude
```

첫 MCP 호출은 `uvx`가 PyPI에서 패키지를 pull 받기 때문에 10~30초 소요된다. 이후 호출은 캐시를 사용한다.

## 제공 MCP
### `nara-jangteo` (Datajang)
- `get_bids_by_keyword` — 최근 7일 용역 입찰공고 키워드 검색 (최대 20개)
- `recommend_bids_for_dept` — 부서 프로필 기반 추천 (최대 60개)
- `analyze_bid_detail` — RFP 첨부파일(HWP/PDF/DOCX) 텍스트 추출

### `data-go-mcp.pps-narajangteo` (Koomook)
- 입찰공고 / 낙찰정보 / 계약정보 조회 (원시 데이터)

자세한 tool 목록은 `docs/mcp-tools.md` 참조.

## Claude Desktop 에서 쓰기 (MCP tool 만)
Claude Desktop 앱 사용자는 서브에이전트 없이 MCP tool 2종만 붙여 쓸 수 있습니다.

```bash
bash scripts/setup-claude-desktop.sh
```

OS 자동 감지 → 절대경로 치환 → `claude_desktop_config.json` 생성/백업. 이후 Claude Desktop 재시작.

세부 절차(수동 설정, WSL, 트러블슈팅): [`docs/claude-desktop-setup.md`](./docs/claude-desktop-setup.md)

## 에이전트 팀 (Claude Code 전용)
`.claude/agents/` 에 5개 에이전트 정의:
- `orchestrator` — 요청 라우팅
- `mcp-operator` — MCP tool 실행
- `data-modeler` — 스키마 설계 (SQL 스킬 활용)
- `verifier` — 완료 전 증거 확보
- `docs-writer` — 문서 유지

세부 운영 규칙은 [CLAUDE.md](./CLAUDE.md) 참조.

## 디렉토리
```
.
├── .claude/agents/     # 에이전트 팀 정의
├── .mcp.json           # 프로젝트 스코프 MCP (글로벌 아님)
├── .env                # API 키 (git 제외)
├── data/raw/           # MCP 원시 응답 캐시
├── db/                 # (2단계) SQLite 스키마
├── scripts/
│   ├── run-mcp.sh              # Claude Code/Desktop 공용 MCP 기동 래퍼
│   └── setup-claude-desktop.sh # Desktop 설정 자동 생성
└── docs/               # 스키마/tool/런북 + Desktop 설치 가이드
```

## 트러블슈팅
| 증상 | 원인/조치 |
|---|---|
| 401/403 응답 | `data.go.kr` 활용신청 승인 대기(보통 1~2시간) 또는 키 미입력 |
| `uvx` hang | 네트워크 / `UV_LINK_MODE=copy` 확인 / `uvx --refresh` 재시도 |
| tool 미노출 | `.mcp.json` 문법 오류 / Claude Code 재시작 필요 |
| 사전규격 403 | 해당 API 별도 활용신청 필요 |

## 로드맵
- **1단계 (현재)** — 공개 MCP 2개 안정화
- **2단계** — JSON→SQLite 로더 + 스키마 정규화
- **3단계** — 자체 MCP(sionlab-mcp): 비즈니스 로직 + 자사 DB 조인

## 기여
이슈/PR 환영합니다. 절차: [CONTRIBUTING.md](./CONTRIBUTING.md) · 행동 강령: [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) · 보안 제보: [SECURITY.md](./SECURITY.md) · 변경 이력: [CHANGELOG.md](./CHANGELOG.md)

## 라이선스
[MIT](./LICENSE). 본 저장소는 data.go.kr 공공 API 클라이언트 래퍼이며, 상류 MCP 서버(`nara-mcp-server`, `data-go-mcp.pps-narajangteo`)의 라이선스는 각 프로젝트 참조.

## 크레딧
- [Datajang/narajangteo_mcp_server](https://github.com/Datajang/narajangteo_mcp_server) — 키워드 검색, RFP 추출
- [Koomook/data-go-mcp-servers](https://github.com/Koomook/data-go-mcp-servers) — 조달청 원시 데이터
- [공공데이터포털](https://www.data.go.kr/) — 나라장터 입찰공고정보서비스 API 제공
