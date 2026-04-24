## 변경 요약
(1~3줄)

## 변경 유형
- [ ] 버그 수정
- [ ] 새 기능
- [ ] 문서
- [ ] 리팩터링
- [ ] 기타

## 검증
실행한 명령어와 결과를 붙여넣기:
```bash
python3 -m json.tool < .mcp.json > /dev/null && echo OK
bash -n scripts/*.sh && echo OK
```

## 체크리스트
- [ ] `.mcp.json` / 설정 JSON 문법 검증 통과
- [ ] shell 스크립트 문법 검증 통과
- [ ] 새 환경변수는 `.env.example` 에 placeholder 추가
- [ ] 새 MCP tool 은 `docs/mcp-tools.md` 에 기재
- [ ] diff 에 API 키/토큰/개인정보 없음
- [ ] `data/raw/*.json` 실제 응답을 커밋하지 않음
- [ ] CHANGELOG.md `[Unreleased]` 섹션에 항목 추가 (사용자 영향 있는 변경)

## 관련 이슈
Closes #
