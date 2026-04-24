# KONEPS_sionlab — 프로젝트 지침

## 목적
나라장터(KONEPS/G2B) 데이터를 **MCP 경유**로 조회하고, 향후 자체 비즈니스 로직(추천/매칭/스코어링)을 얹을 플랫폼의 기반을 구축한다.

## 현재 단계
**1단계 — MCP 데이터 채널 안정화.** 공개 MCP 2개(`nara-jangteo`, `data-go-mcp.pps-narajangteo`)가 stable running 상태인 것을 목표로 한다. 2단계(자체 MCP)는 이후 계획.

## 스코프 정책
- 모든 MCP 설정은 **프로젝트 스코프**(`.mcp.json`)만 사용
- 글로벌 `~/.claude.json` 수정 **절대 금지**
- `claude mcp add` 등 user-scope로 등록하는 CLI **사용 금지**

## 팀 구조 (.claude/agents/)
| 에이전트 | 책임 |
|---|---|
| `orchestrator` | 요청 분해 → 배분 (코드 직접 작성 안 함) |
| `mcp-operator` | MCP tool 실행, `data/raw/` 저장 |
| `data-modeler` | JSON→DDL, SQL 스킬 활용 |
| `verifier` | 완료 주장 전 증거 수집 |
| `docs-writer` | README/런북/ERD 유지 |

**동시성**: 서브에이전트 최대 2개 (메인 포함 Opus 3개). 3개+ 필요 시 2+1 패턴.

## MCP 사용 지침
- tool 네이밍: `mcp__nara-jangteo__*`, `mcp__data-go-mcp__pps-narajangteo__*`
- 원시 응답은 `data/raw/{source}_{endpoint}_{ts}.json`으로 보존
- Rate limit 보호: 동일 쿼리 1분 내 3회 이상 호출 금지 → 캐시 우선

## SQL 스킬 활용
- 스키마 설계: `/database-architect`
- 인덱스/튜닝: `/database-optimizer`
- Postgres 이관 시: `/supabase-postgres-best-practices`
- DDL 작성 플로우: MCP 응답 샘플 → 필드 매핑표 → SQLite DDL → 검증

## 코드 스타일
- 주석/커밋 메시지: **한국어**
- 시간대: ISO-8601, DB는 UTC 저장 (조회 시 KST 변환)
- 민감정보는 `.env` 외 저장 금지

## 검증 원칙 (verification-before-completion)
"완료" 주장 전 반드시:
1. 명령어 출력 확인
2. `verifier` 에이전트 호출
3. 증거 파일 경로 제시

## Out of Scope (지금은 하지 않음)
- 자체 MCP 개발
- 웹/앱 UI
- 비즈니스 추천 로직
- 자사 DB 조인

## 참고 링크
- 입찰공고 API: https://www.data.go.kr/data/15129394/openapi.do
- Datajang MCP: https://github.com/Datajang/narajangteo_mcp_server
- Koomook MCP: https://github.com/Koomook/data-go-mcp-servers
