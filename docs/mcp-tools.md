# MCP Tool 카탈로그 (실측 기반)

**검증 시각**: 2026-04-24 KST. MCP 서버 재시작 후 노출된 tool 7개의 실제 시그니처.

## `nara-jangteo` (Datajang/nara-mcp-server)

### 1. `mcp__nara-jangteo__get_bids_by_keyword`
키워드로 **일반 입찰공고 + 사전규격 공고**를 동시에 조회 (용역/컨설팅/개발/SI).

| 인자 | 타입 | 기본 | 설명 |
|---|---|---|---|
| `keyword` | string | — | 공고명 검색어 (예: "AI", "플랫폼") |
| `days` | integer | 7 | 조회 기간 (일). 30/60/90 확장 가능 |

**반환**: 일반 입찰공고 리스트 + 사전규격 리스트 (공고번호, 기관명, 예산, 마감일, RFP URL 포함)
**검증 상태**: ✅ 실제 응답 확인 (일반 541건 + 사전규격 335건, 30일)

### 2. `mcp__nara-jangteo__recommend_bids_for_dept`
부서 프로필 기반 Top 5 맞춤 추천.

### 3. `mcp__nara-jangteo__analyze_bid_detail`
공고번호로 첨부 RFP(HWP/HWPX/PDF/DOCX/XLSX/ZIP) 자동 다운로드 + 텍스트 추출.

---

## `data-go-mcp.pps-narajangteo` (Koomook)

### 4. `mcp__data-go-mcp_pps-narajangteo__search_bid_announcements`
나라장터 입찰공고 정보 검색.

| 인자 | 타입 | 기본 | 비고 |
|---|---|---|---|
| `start_date` | string\|null | null=오늘 | `YYYY-MM-DD` or `YYYYMMDD` |
| `end_date` | string\|null | null=오늘 | 조회 기간 최대 1개월 |
| `num_of_rows` | int | 10 | 최대 999 |
| `page_no` | int | 1 | |

**반환 필드**(주요): `bidNtceNo`, `bidNtceNm`, `bidNtceSttusNm`, `ntceInsttNm`, `dmndInsttNm`, `asignBdgtAmt`(배정예산), `presmptPrce`(추정가), `bidClseDate`, `bidNtceUrl`
**검증 상태**: ✅ 실제 응답 확인 (812건/당일)

### 5. `mcp__data-go-mcp_pps-narajangteo__get_bid_detail`
개별 입찰공고 상세 조회.

### 6. `mcp__data-go-mcp_pps-narajangteo__search_successful_bids`
낙찰정보 검색 (낙찰자, 낙찰금액, 낙찰일).

### 7. `mcp__data-go-mcp_pps-narajangteo__search_contracts`
계약정보 검색 (계약번호, 계약금액, 계약기간).

---

## MCP 메타 tool (Claude Code 내장)
- `ListMcpResourcesTool` — 서버 리소스 목록
- `ReadMcpResourceTool` — 리소스 내용 읽기

## 응답 저장 규칙
모든 호출 결과는 `data/raw/{source}_{tool}_{extra?}_{YYYYMMDD_HHMMSS}.json` 형식으로 저장. 세부는 `data/raw/README.md` 참조.

## Rate limit 정책
동일 쿼리 1분 내 3회 이상 호출 금지. 캐시(= 최근 `data/raw/` 파일) 우선 조회 후 없을 때만 신규 호출.
