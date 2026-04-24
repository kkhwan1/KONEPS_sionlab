# KONEPS_sionlab

나라장터(KONEPS/G2B) 데이터를 MCP(Model Context Protocol)로 조회하는 Claude Code 기반 워크스페이스.

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

## 에이전트 팀
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
└── docs/               # 스키마/tool/런북
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
