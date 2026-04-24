# Claude Desktop 에서 사용하기

Claude Desktop 앱에서도 본 프로젝트의 MCP 2종(`nara-jangteo`, `data-go-mcp.pps-narajangteo`)을 간단한 "외부 tool" 처럼 붙여 쓸 수 있습니다.

> ⚠️ Claude Desktop 에는 서브에이전트(`orchestrator`, `mcp-operator` 등) 개념이 없습니다. 본 가이드는 **MCP tool 연결만** 다룹니다. 에이전트 워크플로우가 필요하면 Claude Code 를 사용하세요.

## 사전 요구사항 (Claude Code 와 동일)
1. `uvx` 설치 — `curl -LsSf https://astral.sh/uv/install.sh | sh`
2. `data.go.kr` 인증키 발급 → 프로젝트 루트 `.env` 에 입력
3. 프로젝트 경로가 Claude Desktop 이 **파일 실행 가능한 위치**에 있어야 함
   - macOS/Windows: 로컬 디스크면 OK
   - WSL 사용자: 아래 "WSL 주의사항" 참고

---

## 방법 A — 자동 설치 스크립트

프로젝트 루트에서:

```bash
bash scripts/setup-claude-desktop.sh
```

동작:
- OS 감지 (macOS/Windows/WSL)
- `claude_desktop_config.json` 경로 자동 결정
- 기존 설정 있으면 타임스탬프 백업 후 확인 프롬프트
- 절대 경로를 치환하여 저장

설치 후 **Claude Desktop 완전 종료 → 재시작**.

---

## 방법 B — 수동 설정

### 1. 설정 파일 위치

| OS | 경로 |
|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | 공식 빌드 없음 |

### 2. 템플릿 내용

`docs/claude-desktop-config.template.json` 의 `__PROJECT_ROOT__` 를 **본 프로젝트 절대 경로**로 치환:

```json
{
  "mcpServers": {
    "nara-jangteo": {
      "command": "bash",
      "args": ["/절대/경로/KONEPS_sionlab/scripts/run-mcp.sh", "nara-jangteo"]
    },
    "data-go-mcp.pps-narajangteo": {
      "command": "bash",
      "args": ["/절대/경로/KONEPS_sionlab/scripts/run-mcp.sh", "data-go-mcp"]
    }
  }
}
```

### 3. 기존 `mcpServers` 가 있다면
덮어쓰지 말고 **`mcpServers` 객체 내부에 항목만 병합**하세요.

### 4. Claude Desktop 재시작
설정 변경은 앱 재시작 후 반영됩니다.

---

## 검증

재시작 후 Claude Desktop 대화창에서:

```
나라장터에서 "AI" 키워드로 최근 7일 입찰공고 검색해줘
```

→ `mcp__nara-jangteo__get_bids_by_keyword` tool 이 호출되어 공고 리스트가 반환되면 정상.

---

## WSL 주의사항 (Windows 사용자 중 Claude Desktop 을 Windows 네이티브로 쓰는 경우)

**문제**: Claude Desktop 은 Windows 프로세스, `run-mcp.sh` 는 WSL bash 필요. 게다가 `wsl.exe -e bash` 로 호출 시 **non-login/non-interactive shell** 이라 `~/.bashrc` 가 로드되지 않아 `uvx` 를 못 찾음.

**해결**: 자동 설치 스크립트가 이미 반영 — WSL 감지 시 설정을 아래 형태로 렌더링합니다.
```json
{
  "mcpServers": {
    "nara-jangteo": {
      "command": "wsl.exe",
      "args": ["-e", "bash", "/home/USER/projects/KONEPS_sionlab/scripts/run-mcp.sh", "nara-jangteo"]
    }
  }
}
```
그리고 `run-mcp.sh` 상단에서 `PATH=~/.local/bin:~/.cargo/bin:$PATH` 보강하여 `uvx` 접근을 보장합니다.

**검증**: Desktop 재시작 후 tool 이 안 보이면 Linux 쪽에서
```bash
/mnt/c/Windows/System32/wsl.exe -e bash -c 'uvx --version'
```
이 `uvx <버전>` 을 출력해야 합니다. `command not found` 면 `~/.local/bin` 에 uvx 가 있는지 확인.

**대안** — 프로젝트를 Windows 쪽(`C:\...`)으로 옮긴 뒤 Git Bash/MSYS bash 사용. 단, WSL 기반 CLAUDE.md 규약과 충돌하므로 권장하지 않음.

---

## 트러블슈팅

| 증상 | 조치 |
|---|---|
| Desktop 에서 tool 목록에 MCP 가 보이지 않음 | ① 설정 JSON 문법 검증 `python3 -m json.tool < claude_desktop_config.json` ② Desktop 완전 종료(트레이 아이콘 포함) 후 재시작 |
| `command not found: bash` (Windows) | `"command": "C:\\Program Files\\Git\\bin\\bash.exe"` 로 전체 경로 지정 |
| `uvx: command not found` | Claude Desktop 이 상속하는 PATH 에 `~/.local/bin` 이 없을 수 있음. `run-mcp.sh` 상단에 `export PATH="$HOME/.local/bin:$PATH"` 추가 |
| 401/403 | `.env` 키 확인. Desktop 과 Code 는 같은 `.env` 를 공유함 |

---

## 보안 참고

- `.env` 는 여전히 **프로젝트 로컬에만** 존재. Desktop 설정에도 키를 직접 기재하지 않음.
- Desktop 설정 파일(`claude_desktop_config.json`)은 사용자 홈 디렉토리에 저장되며 Git 과 무관.
- 여러 사용자가 같은 기기를 쓰는 경우 `.env` 파일 권한을 `chmod 600` 으로 제한.
