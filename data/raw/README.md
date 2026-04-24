# data/raw

MCP 서버가 반환한 **원시 JSON 응답**을 가공 없이 그대로 보관한다. 스키마 리버스, 재현, 디버깅의 단일 진실 원본.

## 명명 규칙
```
{source}_{tool}_{extra?}_{YYYYMMDD_HHMMSS}.json
```

| 토큰 | 의미 | 예시 |
|---|---|---|
| `source` | MCP 서버 식별자 | `nara-jangteo`, `data-go-mcp` |
| `tool` | 호출한 tool 명 (`mcp__` 접두사 제거) | `get_bids_by_keyword`, `search_bid_announcements` |
| `extra` | 주요 파라미터 요약 (선택) | 키워드/기관/공고번호 등 |
| `YYYYMMDD_HHMMSS` | 호출 시각 (KST) | `20260424_134121` |

### 예시
```
nara-jangteo_get_bids_by_keyword_AI_20260424_134121.json
data-go-mcp_search_bid_announcements_20260424_134121.json
data-go-mcp_get_bid_detail_R26BK01487335_20260424_140500.json
```

## 파일 스키마 규약
각 파일의 최상위에 `_meta` 필드를 둔다:

```json
{
  "_meta": {
    "source": "nara-jangteo",
    "tool": "get_bids_by_keyword",
    "called_at": "2026-04-24T13:41:21+09:00",
    "params": {"keyword": "AI", "days": 30},
    "note": "..."
  },
  "result": { /* MCP 원시 응답 */ }
}
```

## 금지
- 수정/가공된 데이터 저장 금지 (별도 `data/processed/` 사용)
- 같은 호출 결과 중복 저장 금지 (timestamp 로 자연스럽게 구분되지만 의도적 덮어쓰기 X)
- Git 커밋 금지 — 대용량/변동성. `data/` 는 `.gitignore` 대상
