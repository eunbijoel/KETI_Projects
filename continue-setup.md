# Continue 설정 · 사용 설명서

전체 순서: ① 준비 → ② Continue 설치 → ③ 모델 연결 → ④ 모드·기본 툴 파악 → ⑤ Deep Agent와 비교

> Continue = IDE 안에서 AI가 파일 읽고/고치고/명령 실행하게 해주는 오픈소스 코딩 에이전트  
> Deep Agents(`deepagents-code`) = 코드로 에이전트 루프·메모리·서브에이전트·커스텀 툴을 직접 조립

공식 문서: https://docs.continue.dev

---

## 0. Ollama와 사전 준비

1. [VS Code](https://code.visualstudio.com/)와 [Ollama](https://ollama.com/download/windows)를 설치합니다.
2. Ollama 실행합니다.
3. VS Code의 **터미널 → 새 터미널**을 열고 실행

```powershell
ollama serve          # 안 떠 있으면
ollama list
ollama pull  gemma3:27b
또는 
ollama pull model
```

`모델 예) gemma3:12b`**가 보이면 준비 완료.** 

---

## 1. Continue 설치

1. VS Code에서 Extensions 검색 또는 `Ctrl+Shift+X` → `Continue` 검색
2. **Continue 확장** 설치 →  **open-source AI code agent** (`Continue.continue`) 왼쪽 **Continue 아이콘** 클릭
3. 첫 화면에서 `API Key` **옆** `Local` 클릭:
<img width="694" height="525" alt="02-local" src="https://github.com/user-attachments/assets/3c0cb5bd-c7b0-4b33-9d0a-88a67c531d34" />


*① Continue 패널 열기 → ② Local 선택. 이번 실습에서는 API Key를 입력하지 않습니다.*

*③ Ollama 실행 표시 확인 → ④ 아래쪽* `Skip and configure manually` *클릭.*

**화면에 보이는 모델 3개를 모두 받을 필요는 없습니다.** 다음 단계에서 설치된 모델 하나만 등록합니다.

---

## 2. 모델 연결 · config.yaml 열기 · 최소 설정

열린 `config.yaml`에서 기존 내용을 복사해 저장한 뒤 모델 이름만 본인 Ollama에 맞게 수정 + `Ctrl+S`**로 저장**

```yaml
name: Main Config
version: 1.0.0
schema: v1

models:
  - name: Gemma 3 12B Local
    provider: ollama
    model: gemma3:12b
    apiBase: http://localhost:11434
    roles:
      - chat
      - edit
      - apply
    capabilities:
      - tool_use

  - name: local-autocomplete
    provider: ollama
    model: qwen2.5-coder:1.5b
    roles:
      - autocomplete
```

### 2.1 동작 확인

Continue에서 `+` **→ 새 대화 →** `Gemma 3 12B Local` **선택 →** `Chat` **모드**를 선택합니다.

```text
안녕. 한국어로 짧게 자기소개해줘.
```

**한국어 답변이 나오면 모델 연결 완료.** 첫 응답은 모델을 불러오느라 오래 걸릴 수 있습니다.

- **Chat** 모드: 추측할 수 있음 (툴 없음)
- **Agent** 모드: `ls` / `glob` 같은 툴을 쓰려 함 → **이게 정상**

모드 전환: 입력창 아래 모드 선택, 또는 `Ctrl+.` / `Cmd+.`

---

## 3. 활용: 모드 · 기본 툴 · 확장

모델만 연결되면 **파일 읽기/쓰기, 코드 실행,** `@`**로 파일 올리기** 같은 기능은 Agent 모드에서 **기본 제공**됨. 별도 툴 설치 없이 바로 사용 가능:

<img width="648" height="466" alt="03-code" src="https://github.com/user-attachments/assets/e1d1d676-f51b-40a9-97b2-97124d84e85c" />


| 모드 | 할 수 있는 것 |
|---|---|
| **Chat** | 설명·질문 (툴 없음) |
| **Plan** | 파일/검색 등 **읽기만** |
| **Agent** | 파일 생성·수정, 터미널 실행까지 |

**기본 툴 (자동)**  
`read_file`, `ls`/`glob`/`grep`, `create_new_file`, `edit_existing_file`, `run_terminal_command` 등. 쓰기·셸은 승인 권장 (`rm -rf`, force push 등은 Reject).

**MCP** — 브라우저·DB·GitHub처럼 IDE 밖 연동이 필요할 때만. Agent 모드에서 `.continue/mcpServers/` 또는 `config.yaml`의 `mcpServers`로 추가.  
**Rules** — 팀/프로젝트 습관을 고정하고 싶을 때 `.continue/rules/*.md`에 적어 두면 됨.
프로젝트 루트:

```bash
mkdir -p .continue/rules
```

예: `.continue/rules/01-python.md`

```md
---
name: Python project rules
globs: ["**/*.py"]
alwaysApply: false
---
- Python 3.12 기준
- 테스트는 pytest
- 시크릿/경로를 코드에 하드코딩하지 말 것
- 큰 리팩터 전에 Plan 모드로 영향 범위 먼저 말해
```

자세한 옵션: [Agent 툴](https://docs.continue.dev/ide-extensions/agent/how-it-works) · [MCP](https://docs.continue.dev/customize/deep-dives/mcp) · [Rules](https://docs.continue.dev/customize/deep-dives/rules)

---

## 4. Deep Agent vs Continue

둘 다 “모델 + 툴”이지만, **Continue는 IDE 안의 손**, **Deep Agent는 코드로 조립하는 공장**에 가깝다.

### 4-1. 구조 차이

```text
[Continue]
사람 ↔ IDE 사이드바 ↔ 모델 ↔ (기본 툴 | MCP) ↔ 워크스페이스

[Deep Agents]
사람 ↔ CLI/자체 UI ↔ create_deep_agent 루프
     ↔ 하네스 툴 + 직접 정의한 Python 툴 / MCP / Adapter
     ↔ 메모리 · Skills · 서브에이전트
```

### 4-2. 뭐가 다른가

| | Continue 제공 툴(+MCP) | Deep Agent |
|---|---|---|
| 진입 | 확장 설치 → IDE에서 바로 | venv/도커·앱 구성 필요 |
| 툴 | IDE 내장(파일·검색·실행) + MCP | 하네스 기본 툴 + **코드로 툴 정의** + MCP |
| 자동화 | 한 세션 Agent 루프 | 서브에이전트, 장시간·병렬 위임 |
| 기억 | Rules / `config.yaml` | `AGENTS.md`, memory, skills |
| 도메인 | MCP·프롬프트로 설명 | Tool Registry + Adapter로 **기존 분석 툴 재사용** |
| 강점 / 약점 | 빠른 편집 · 긴 자동화는 약함 | 플랫폼화 쉬움 · 승인·샌드박스는 직접 설계 |

**Continue 툴** = 설치만 하면 파일 읽고/고치고/실행하는 루프가 됨.  
**Deep Agent** = 같은 도메인 툴을 WebUI·배치·에이전트가 공유하거나, 없는 툴을 만들어 등록할 때 유리.

### 4-3. 언제 뭘 쓰나

- **Continue** — 지금 에디터에서 고치기, 팀원이 IDE만 열면 될 때  
- **Deep Agent** — 도메인 툴 파이프라인, 자체 UI(`coding-agent` 등), 메모리·서브에이전트가 필요할 때  
- **같이** — Continue = IDE 손, Deep Agent = 도메인 공장 (MCP는 양쪽에 붙여도 됨)

---

## 빠른 실행 명령어

```text
Ctrl+L / Cmd+L     Continue 채팅
Ctrl+. / Cmd+.     Chat ↔ Plan ↔ Agent 순환
@파일              컨텍스트 지정
⚙️ Local Config    config.yaml
.continue/rules    프로젝트 룰
.continue/mcpServers   프로젝트 MCP
```

문서 더보기:

- Continue 설정: https://docs.continue.dev/customize/deep-dives/configuration  
- Agent 툴: https://docs.continue.dev/ide-extensions/agent/how-it-works  
- MCP: https://docs.continue.dev/customize/deep-dives/mcp  
- Deep Agents Code: https://docs.langchain.com/oss/deepagents/code/overview
