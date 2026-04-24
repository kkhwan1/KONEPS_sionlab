# MCP Tool 카탈로그

> 실제 tool 시그니처는 MCP 서버 기동 후 `mcp-operator` 가 첫 호출로 확인하여 이 문서를 갱신한다.

## `nara-jangteo` (Datajang/nara-mcp-server)

### `get_bids_by_keyword`
- **입력**: `keyword: string` (필수), `limit?: int`
- **출력**: 최근 7일 용역 입찰공고 리스트 (공고번호, 공고명, 수요기관, 마감일시, 금액, URL)
- **제한**: 최대 20개, 마감되지 않은 공고만

### `recommend_bids_for_dept`
- **입력**: 부서 프로필 (전문 분야, 키워드 리스트)
- **출력**: 상위 5개 맞춤 추천 + 추천 근거

### `analyze_bid_detail`
- **입력**: `bid_no: string`
- **출력**: 첨부파일(HWP/HWPX/PDF/DOCX/XLSX/ZIP) 추출 텍스트 + 요약

## `data-go-mcp.pps-narajangteo` (Koomook)

### 입찰공고 조회
- data.go.kr 엔드포인트 1 대 1 매핑 (구체 tool 명은 서버 기동 후 확정)

### 낙찰정보 조회
- 낙찰자, 낙찰금액, 낙찰일

### 계약정보 조회
- 계약번호, 계약금액, 계약기간

## 호출 예시 (mcp-operator 전용)
```
mcp__nara-jangteo__get_bids_by_keyword(keyword="AI")
mcp__nara-jangteo__analyze_bid_detail(bid_no="20260424123")
```

## 응답 저장 규칙
- 경로: `data/raw/{source}_{endpoint}_{YYYYMMDD_HHMMSS}.json`
- 원본 그대로 저장. 가공 전 단계 보존 필수
