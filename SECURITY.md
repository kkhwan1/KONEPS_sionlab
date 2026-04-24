# Security Policy

## 지원 버전

초기 공개(v0.x)는 모두 지원 대상입니다. v1.0 릴리즈 이후 별도 공지.

| Version | Supported |
|---|---|
| 0.x | ✅ |

---

## 취약점 제보

**공개 이슈로 보고하지 마세요.** 다음 중 하나로 연락:

- GitHub Private Vulnerability Report (권장): 저장소 → Security → Report a vulnerability
- 이메일: 저장소 관리자의 GitHub 프로필에 공개된 이메일

72시간 내 1차 회신을 목표로 합니다. 수정/공개 일정은 영향도 평가 후 제보자와 합의합니다.

---

## 범위

### In scope
- `scripts/run-mcp.sh`, `scripts/setup-claude-desktop.sh` 등 본 저장소 코드
- 문서에서 권장하는 설정 패턴 (`.mcp.json` 구조, Desktop 병합 로직)
- 의존 MCP 서버(`nara-mcp-server`, `data-go-mcp.pps-narajangteo`) 호출 방식

### Out of scope (다른 저장소에 제보)
- 상류 MCP 서버 자체의 버그 → 각 GitHub 저장소(Datajang/Koomook)
- `uvx`/`uv` 런타임 버그 → astral-sh/uv
- `data.go.kr` API 자체 이슈 → 공공데이터포털 고객센터

---

## API 키 취급 규칙 (사용자 책임)

본 프로젝트가 강제하는 설계:
1. **`.mcp.json` 에 키 0개** — 모든 키는 `.env` 에서 로드
2. **`.env` 는 git 제외** — `.gitignore` 에 포함. 권한 600 권장 (`chmod 600 .env`)
3. **Desktop 설정에도 키 없음** — `run-mcp.sh` 래퍼가 런타임 주입

### 키가 유출되었다고 판단될 때

1. **즉시 공공데이터포털에서 해당 키 재발급/폐기**
   - https://www.data.go.kr 로그인 → 마이페이지 → 인증키 관리
2. **로컬 `.env` 및 캐시 정리**
   ```bash
   rm -f .env ~/.cache/uv/*  # 키가 캐시될 가능성은 낮으나 예방
   ```
3. **Git 히스토리 점검** — 실수로 커밋했다면
   ```bash
   git log --all -p | grep -i 'API_KEY='
   ```
   유출 발견 시 `git filter-repo` 또는 BFG Repo-Cleaner 로 히스토리 재작성 + 강제 푸시 + 팀 통지.
4. **GitHub Secret Scanning 활성화** — 저장소 Settings → Security → Secret scanning.

---

## 위협 모델 (간이)

| 위협 | 대응 |
|---|---|
| `.env` 실수 커밋 | `.gitignore` + PR 체크리스트 + GitHub Secret Scanning |
| 악성 PR 의 스크립트 변조 | 코드 리뷰 필수, `run-mcp.sh` 는 `set -euo pipefail` + 화이트리스트 TARGET |
| MCP 서버 임의 실행 | `uvx` 는 PyPI 공식 패키지만 사용. `command`/`args` 에 사용자 입력 비허용 |
| Desktop 설정 덮어쓰기 | `setup-claude-desktop.sh` 는 **병합** 방식 + 기존 파일 자동 백업 |
| WSL interop 경유 권한 상승 | `wsl.exe -e bash` 는 사용자 권한 그대로. 추가 esclation 없음 |

---

## 보안 관련 검증 명령 (PR/릴리즈 전)

```bash
# 키 누출 점검
git log --all -p | grep -iE '(api[_-]?key|service[_-]?key|nara[_-]?api)[=:][^$]' | head
# .env 추적 여부
git check-ignore .env && echo "OK"
# 실행 권한 점검
find scripts -name "*.sh" -exec ls -l {} \;
# JSON 문법
find . -name '*.json' -not -path './node_modules/*' -exec python3 -m json.tool {} \; > /dev/null
```
