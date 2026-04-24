# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] — 2026-04-24

최초 공개 (Initial open-source release).

### Added
- 나라장터 MCP 2종 프로젝트 스코프 설정 (`.mcp.json`)
  - `nara-jangteo` (Datajang/nara-mcp-server) — 키워드 검색, 부서 추천, RFP 추출
  - `data-go-mcp.pps-narajangteo` (Koomook) — 입찰공고/낙찰/계약 원시 조회
- `.env` 기반 API 키 관리 래퍼 `scripts/run-mcp.sh`
  - `.mcp.json` 에 키 0개 — 단일 소스(`.env`)에서 주입
  - `wsl.exe -e bash` 경유 호출 대비 PATH 보강
- Claude Desktop 자동 설정 스크립트 `scripts/setup-claude-desktop.sh`
  - macOS / Windows / WSL 감지, 기존 설정 병합(덮어쓰기 방지), 타임스탬프 백업
  - JSON 문법 검증 내장
- 에이전트 팀 5종 (`.claude/agents/`)
  - `orchestrator`, `mcp-operator`(MCP tool 9개 권한), `data-modeler`, `verifier`, `docs-writer`
- 문서
  - `CLAUDE.md` — 아키텍처 / 설계 결정 / 운영 규칙
  - `README.md` — Quick Start, Desktop/Code 경로 분기
  - `docs/mcp-tools.md` — 실측 기반 tool 카탈로그 (signature 포함)
  - `docs/claude-desktop-setup.md` — Desktop 설치/트러블슈팅
  - `data/raw/README.md` — 응답 캐시 파일 명명 규칙 + `_meta` 스키마

### Verified
- Claude Code 서브에이전트(`mcp-operator`) → 실제 MCP 호출로 `Regular=82, Prespec=50` 수신
- Claude Desktop stdio 시뮬레이션 → `tools/list` 3개 + `tools/call` 성공 (입찰공고 119건 검색)

### Known Issues
- `data-go-mcp.pps-narajangteo.get_bid_detail` — 업스트림 버그로 실패. 우회: `search_bid_announcements` 사용
- 사전규격 API — data.go.kr 에서 별도 활용신청 필요할 수 있음 (같은 키로 1차 시도 후 403 시 신청)

---

[Unreleased]: https://github.com/OWNER/KONEPS_sionlab/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/OWNER/KONEPS_sionlab/releases/tag/v0.1.0
