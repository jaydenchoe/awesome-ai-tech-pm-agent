# Hermes Agent 핸즈온 세미나 튜토리얼 초안

> 작성일: 2026-05-06  
> 목적: 오늘 진행할 핸즈온 세미나에서 참가자가 **설치 → 모델 설정 → 첫 대화 → 도구/메모리/스킬 → 메시징 게이트웨이 → 운영 점검**까지 따라 하도록 돕는 상세 진행안입니다.  
> 기준 자료: Hermes Agent 공식 사이트 및 공식 문서의 Quickstart, Installation, CLI, Configuration, Tools, Memory, Skills, Messaging Gateway 문서를 확인해 재구성했습니다.

---

## 0. 세미나 개요

### 0.1 한 줄 소개

Hermes Agent는 Nous Research가 만든 오픈소스 퍼스널 AI 에이전트입니다. 공식 문서 기준으로는 단순 챗봇이나 IDE 보조 도구가 아니라, 사용자의 환경과 프로젝트를 기억하고, 도구를 호출하며, 반복 작업을 스킬로 축적하고, CLI와 여러 메시징 플랫폼에서 지속적으로 사용할 수 있는 셀프 호스팅형 에이전트로 설명됩니다.

### 0.2 오늘의 목표

세미나 종료 시점에 참가자는 다음을 직접 수행할 수 있어야 합니다.

1. Linux, macOS, WSL2 환경에서 Hermes Agent를 설치한다.
2. `hermes model` 또는 `hermes setup`으로 LLM provider를 설정한다.
3. `hermes` 또는 `hermes --tui`로 첫 대화를 시작한다.
4. 현재 디렉터리/파일/터미널 도구를 활용한 간단한 작업을 시킨다.
5. 세션 이어하기, 슬래시 명령, 컨텍스트 압축 시점을 이해한다.
6. Memory와 Skills가 어떤 역할을 하는지 실습한다.
7. Telegram/Discord/Slack 등 메시징 게이트웨이 구성을 데모 또는 선택 실습으로 진행한다.
8. `hermes doctor`, `hermes gateway status`, `hermes skills audit` 등 기본 점검 루틴을 익힌다.

### 0.3 추천 진행 시간표

| 시간 | 세션 | 산출물 |
|---:|---|---|
| 00:00-00:10 | 오리엔테이션 | Hermes Agent 개념, 보안 주의사항 공유 |
| 00:10-00:25 | 설치 | 각자 `hermes` 명령 실행 가능 |
| 00:25-00:45 | Provider 설정 | 선택한 모델로 첫 응답 확인 |
| 00:45-01:10 | CLI/TUI 기본 실습 | repo 요약, 파일 탐색, 세션 이어하기 |
| 01:10-01:35 | Tools & terminal backend | 로컬/도커/SSH 실행 방식 비교와 안전 설정 |
| 01:35-02:05 | Memory & Skills | 기억할 정보 저장, 스킬 검색/설치/호출 |
| 02:05-02:30 | Messaging Gateway | Telegram/Discord 등 연결 흐름 데모 |
| 02:30-02:45 | 장애 대응 | doctor, model 재설정, gateway status |
| 02:45-03:00 | 개인 워크플로 설계 | 참가자별 적용 과제 정의 |

---

## 1. 사전 준비

### 1.1 참가자 환경 체크리스트

Hermes Agent 공식 설치 문서는 Linux, macOS, WSL2, Android Termux 경로를 안내합니다. 오늘 핸즈온에서는 **Linux/macOS/WSL2**를 기본으로 합니다.

참가자는 시작 전에 아래를 확인합니다.

```bash
git --version
uname -a
printf "$SHELL\n"
```

권장 환경:

- macOS 또는 Linux 노트북
- Windows 사용자는 WSL2 Ubuntu 권장
- Git 사용 가능
- 안정적인 인터넷 연결
- API key 또는 OAuth로 연결 가능한 LLM provider 계정
- 선택: Docker Desktop 또는 Docker Engine
- 선택: Telegram/Discord/Slack 중 하나의 bot 테스트 권한

### 1.2 Provider/API 준비

공식 Quickstart는 `hermes model`을 가장 중요한 설정 단계로 봅니다. 첫 세미나에서는 다음 중 하나를 미리 준비하게 합니다.

| 옵션 | 장점 | 주의 |
|---|---|---|
| Nous Portal | Hermes와 네이티브 통합, Tool Gateway 사용 가능 | 계정/구독 상태 확인 필요 |
| OpenRouter | 여러 모델 라우팅 가능 | API key 필요 |
| OpenAI-compatible custom endpoint | 사내 vLLM, SGLang, Ollama 등에 적합 | base URL, model name, context size 확인 필요 |
| Anthropic/OpenAI/GitHub Copilot 등 | 익숙한 모델 사용 가능 | 인증 방식과 과금 정책 확인 필요 |

중요: 공식 Quickstart 기준으로 Hermes Agent는 최소 64K context window 모델을 요구합니다. 로컬 모델 사용자는 Ollama, llama.cpp, vLLM 등의 context 길이를 65,536 토큰 이상으로 맞추는지 확인해야 합니다.

### 1.3 보안 안내

Hermes는 terminal, file edit, browser, messaging, memory, skill 등 강력한 도구를 사용할 수 있습니다. 세미나에서는 다음 원칙을 적용합니다.

- 개인 노트북의 민감 디렉터리에서 시작하지 않는다.
- 실습용 빈 폴더를 만들고 그 안에서 시작한다.
- API key는 화면 공유하지 않는다.
- `.env`, token, SSH key를 채팅 프롬프트에 붙여넣지 않는다.
- 위험 명령 승인 여부를 묻는 상황에서는 강사가 먼저 설명한다.
- 실습 과제는 삭제/배포/결제/외부 전송을 포함하지 않는다.

실습용 폴더:

```bash
mkdir -p ~/hermes-seminar-lab
cd ~/hermes-seminar-lab
cat > README.md <<'TXT'
# Hermes Seminar Lab

This is a safe demo workspace for Hermes Agent hands-on practice.
TXT
```

---

## 2. Hermes Agent 개념 설명

### 2.1 왜 퍼스널 에이전트인가?

일반 챗봇은 대화창 안에서만 반응하고, 새 대화를 시작하면 맥락이 쉽게 끊깁니다. Hermes Agent는 다음 요소를 결합해 개인 작업 환경에 더 가깝게 동작하도록 설계되어 있습니다.

- **Persistent memory**: 사용자 선호, 프로젝트 관례, 환경 정보를 세션 간 유지
- **Tools**: 터미널, 파일, 웹, 브라우저, 이미지/음성, 메시징, 자동화 도구 호출
- **Skills**: 반복 가능한 절차 지식을 `SKILL.md` 문서로 저장하고 재사용
- **Gateway**: Telegram, Discord, Slack, WhatsApp 등 다양한 채널에서 접근
- **Sessions**: CLI와 메시징 플랫폼 대화를 기록하고 이어서 진행
- **Terminal backends**: local, Docker, SSH, Modal, Daytona, Vercel Sandbox 등 실행 위치 선택

### 2.2 오늘 강조할 핵심 멘탈 모델

Hermes를 “똑똑한 채팅창”으로 보면 장점이 잘 드러나지 않습니다. 대신 다음처럼 이해합니다.

```text
사용자 요청
  ↓
Hermes Agent
  ├─ 모델/provider: 생각하고 응답 생성
  ├─ tools/toolsets: 필요한 행동 수행
  ├─ memory: 오래 유지할 사실 관리
  ├─ skills: 반복 절차와 노하우 재사용
  ├─ sessions: 대화 기록과 검색
  └─ gateway: 터미널 밖 채널로 확장
```

---

## 3. 설치 실습

### 3.1 공식 one-line install

Linux/macOS/WSL2에서 다음 명령을 실행합니다.

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

설치가 끝나면 shell 설정을 다시 읽습니다.

```bash
source ~/.bashrc
# zsh 사용자는:
# source ~/.zshrc
```

정상 설치 확인:

```bash
which hermes
hermes --help
```

### 3.2 설치 중 수행되는 일

공식 Installation 문서에 따르면 installer는 필요한 의존성, repository clone, virtual environment, 전역 `hermes` 명령 설정, provider 설정 흐름을 처리합니다. 의존성에는 Python 3.11, Node.js v22, ripgrep, ffmpeg 등이 포함됩니다.

강사용 설명 포인트:

- Git만 직접 준비하면 나머지는 installer가 처리하는 흐름입니다.
- Windows native는 공식 문서상 지원 대상이 아니므로 WSL2 안에서 진행합니다.
- 설치 후 `hermes`가 PATH에 잡히지 않으면 shell reload 또는 PATH 확인이 우선입니다.

### 3.3 설치 문제 대응

| 증상 | 원인 후보 | 해결 |
|---|---|---|
| `hermes: command not found` | shell reload 미실행, PATH 문제 | `source ~/.bashrc` 후 `which hermes` |
| Git 없음 | 최소 prerequisite 미충족 | OS별 Git 설치 후 재시도 |
| 네트워크 실패 | GitHub 접근/방화벽 문제 | 네트워크 변경, proxy 확인 |
| WSL2가 아님 | Windows native shell에서 실행 | WSL2 Ubuntu terminal에서 재실행 |

---

## 4. Provider 설정 실습

### 4.1 기본 wizard

가장 권장되는 첫 설정은 다음입니다.

```bash
hermes model
```

전체 설정을 한 번에 훑고 싶으면 다음을 사용합니다.

```bash
hermes setup
```

진행 중 참가자에게 확인시킬 항목:

- provider 선택 완료 여부
- model name 확인
- context window가 64K 이상인지 확인
- API key/OAuth 인증 성공 여부
- custom endpoint인 경우 base URL과 model ID 정확성

### 4.2 설정 파일 구조

Hermes 설정은 기본적으로 `~/.hermes/` 아래에 저장됩니다.

```text
~/.hermes/
├── config.yaml     # model, terminal, compression 등 일반 설정
├── .env            # API key, token, password 등 secrets
├── auth.json       # OAuth provider 인증 정보
├── SOUL.md         # 에이전트 기본 정체성/톤
├── memories/       # MEMORY.md, USER.md
├── skills/         # skill source of truth
├── cron/           # scheduled jobs
├── sessions/       # session transcripts
└── logs/           # logs
```

핵심 규칙:

- secrets는 `.env`
- 일반 설정은 `config.yaml`
- `hermes config set`은 값의 성격에 따라 적절한 파일로 저장합니다.

예시:

```bash
hermes config set model anthropic/claude-opus-4.6
hermes config set terminal.backend docker
hermes config set OPENROUTER_API_KEY sk-or-REDACTED
```

### 4.3 설정 점검

```bash
hermes config
hermes config check
hermes doctor
```

강사용 설명:

- `hermes doctor`는 원인 추적의 첫 번째 도구로 소개합니다.
- Provider 문제가 의심될 때는 고급 기능을 추가하지 말고 `hermes model`로 기본 대화 성공부터 복구합니다.

---

## 5. 첫 대화 실습

### 5.1 CLI/TUI 시작

```bash
cd ~/hermes-seminar-lab
hermes
```

또는 modern TUI:

```bash
cd ~/hermes-seminar-lab
hermes --tui
```

공식 CLI 문서는 Hermes CLI가 terminal 기반 인터페이스이며 multiline editing, slash-command autocomplete, conversation history, streaming tool output 등을 제공한다고 설명합니다.

### 5.2 첫 프롬프트

아래 중 하나를 사용합니다.

```text
현재 디렉터리를 확인하고, 이 폴더가 어떤 실습 공간인지 README.md를 바탕으로 5줄로 요약해줘.
```

```text
내 현재 작업 디렉터리의 파일 목록을 보고, 오늘 Hermes Agent 핸즈온에서 사용할 수 있는 간단한 실습 아이디어 3개를 제안해줘.
```

성공 기준:

- welcome banner에 model/provider가 표시된다.
- Hermes가 오류 없이 응답한다.
- 필요 시 file/terminal tool을 사용한다.
- 한 번 더 후속 질문을 해도 대화가 이어진다.

### 5.3 non-interactive single query

자동화나 스크립트형 사용을 보여주려면 다음을 실행합니다.

```bash
hermes chat -q "현재 디렉터리의 README.md를 한 문단으로 요약해줘."
```

특정 toolset을 지정하는 예:

```bash
hermes chat --toolsets "terminal,file" -q "현재 폴더 구조를 확인하고 안전한 실습 과제를 제안해줘."
```

---

## 6. CLI 핵심 기능 실습

### 6.1 Slash commands

대화창에서 `/`를 입력해 autocomplete를 확인합니다.

자주 쓰는 명령:

| 명령 | 용도 | 실습 |
|---|---|---|
| `/help` | 사용 가능한 명령 확인 | 처음 실행 |
| `/tools` | tool 목록 확인 | 어떤 권한이 열려 있는지 확인 |
| `/model` | 모델 변경 | provider fallback 설명 시 사용 |
| `/usage` | token/cost 확인 | 장시간 대화 중 확인 |
| `/compress` | context 압축 | context bar가 높아질 때 설명 |
| `/new` 또는 `/reset` | 새 대화 시작 | 세션 분리 실습 |
| `/retry` | 마지막 응답 재시도 | 품질 비교 |
| `/undo` | 마지막 exchange 제거 | 잘못된 요청 회수 |

### 6.2 multiline 입력

공식 Quickstart는 여러 줄 입력을 위해 `Alt+Enter` 또는 `Ctrl+J`를 안내합니다.

실습 프롬프트:

```text
다음 조건을 만족하는 실습 계획을 만들어줘.

조건:
1. 초보자도 따라할 수 있어야 함
2. 위험한 삭제 명령 금지
3. 결과물은 markdown 파일
4. 마지막에 검증 명령 포함
```

### 6.3 중단/방향 전환

에이전트가 오래 걸리는 작업을 수행 중이면 새 메시지를 입력해 방향을 바꾸거나 `Ctrl+C`를 사용합니다.

강사용 데모:

1. “현재 폴더와 하위 폴더를 자세히 분석해줘” 요청
2. 중간에 “멈추고, 파일 수정은 하지 말고 요약만 해줘” 입력
3. interrupt-and-redirect 동작 설명

---

## 7. 세션 이어하기 실습

### 7.1 최근 세션 이어하기

현재 대화를 종료한 뒤 아래를 실행합니다.

```bash
hermes --continue
# 또는
hermes -c
```

공식 Sessions 문서 기준으로 Hermes는 CLI와 메시징 플랫폼 대화를 session으로 자동 저장하며, SQLite metadata와 JSONL transcript를 사용합니다.

### 7.2 세션 확인 프롬프트

```text
방금 전 대화에서 우리가 만든 실습 폴더 이름과 README.md 내용에 대해 기억나는 것을 말해줘.
```

성공 기준:

- 이전 대화 맥락을 이어서 답변한다.
- 새 세션이 아니라 continuation임을 확인할 수 있다.

### 7.3 세션 운영 팁

- 주제별로 `/new`를 사용해 context를 분리합니다.
- 긴 대화는 `/usage`로 context 사용량을 확인합니다.
- context가 80% 이상이면 `/compress`를 고려합니다.
- 프로젝트별로 세션 title을 정해두면 resume이 편합니다.

---

## 8. Tools & Toolsets 실습

### 8.1 Toolset 개념

공식 Tools 문서는 tool을 agent 기능 확장 함수로, toolset을 platform별로 켜고 끌 수 있는 논리적 묶음으로 설명합니다. 대표 범주는 다음과 같습니다.

| 범주 | 예시 | 실습 아이디어 |
|---|---|---|
| Web | web search/extract | 공식 문서 찾아 요약 |
| Terminal & Files | terminal, process, read_file, patch | 실습 폴더 분석 |
| Browser | navigate, snapshot, vision | 웹 UI 탐색 데모 |
| Media | vision, image, TTS | 선택 데모 |
| Agent orchestration | todo, clarify, execute_code, delegate | 작업 계획 수립 |
| Memory & recall | memory, session_search | 선호 저장/검색 |
| Automation | cronjob, send_message | 데일리 리포트 구상 |
| Integrations | MCP, Home Assistant 등 | 확장 시나리오 설명 |

### 8.2 활성 도구 확인

```bash
hermes tools
```

특정 toolset으로 실행:

```bash
hermes chat --toolsets "terminal,file,skills" -q "현재 폴더에 세미나 체크리스트 markdown 초안을 만들어도 되는지 계획만 세워줘."
```

### 8.3 Terminal backend 비교

공식 문서는 local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox 등을 안내합니다.

| backend | 설명 | 세미나 추천도 |
|---|---|---|
| local | 현재 머신에서 명령 실행 | 빠르지만 신뢰 환경에서만 |
| docker | 격리 컨테이너에서 실행 | 실습/재현성에 좋음 |
| ssh | 원격 서버에서 실행 | agent가 자기 코드/로컬 환경을 건드리지 않게 분리 |
| modal/daytona/vercel_sandbox | cloud/serverless 실행 | 고급 운영 시나리오 |

Docker backend 설정 예:

```bash
hermes config set terminal.backend docker
```

SSH backend 개념 예:

```yaml
terminal:
  backend: ssh
```

```bash
# ~/.hermes/.env 예시: 실제 값은 화면 공유 금지
TERMINAL_SSH_HOST=my-server.example.com
TERMINAL_SSH_USER=myuser
TERMINAL_SSH_KEY=~/.ssh/id_rsa
```

강사용 주의:

- local backend는 가장 직관적이지만 민감 파일 접근 위험을 설명합니다.
- Docker backend는 persistent sandbox처럼 동작할 수 있어, 한 번 설치한 package나 생성 파일이 같은 Hermes process 동안 유지될 수 있음을 설명합니다.
- SSH backend는 agent 실행 공간을 분리하려는 보안 목적에 적합합니다.

---

## 9. Memory 실습

### 9.1 Memory가 저장하는 것

공식 Persistent Memory 문서는 Hermes memory를 세션 간 유지되는 bounded curated memory로 설명합니다. 주요 파일은 다음 두 개입니다.

| 파일 | 목적 | 용량 감각 |
|---|---|---|
| `MEMORY.md` | 환경 사실, 프로젝트 관례, 배운 점 | 약 2,200 chars |
| `USER.md` | 사용자 선호, 커뮤니케이션 스타일, 기대치 | 약 1,375 chars |

두 파일은 `~/.hermes/memories/` 아래에 저장되며, 세션 시작 시 system prompt에 snapshot 형태로 주입됩니다. 세션 중 memory가 바뀌어도 다음 세션 시작 전까지 prompt 주입 내용은 갱신되지 않을 수 있으므로, “방금 저장한 메모리가 즉시 모든 답변에 반영된다”고 오해하지 않도록 설명합니다.

### 9.2 기억시킬 정보 예시

대화창에서 다음을 입력합니다.

```text
앞으로 이 실습 환경에 대해 기억해줘: 나는 Hermes Agent 세미나를 위해 ~/hermes-seminar-lab 폴더를 사용하고, 위험한 삭제 명령은 피하고, 설명은 한국어로 단계별로 받는 것을 선호해.
```

그다음 새 세션을 시작하거나 continue 후 확인합니다.

```bash
hermes --continue
```

확인 프롬프트:

```text
내 Hermes 세미나 실습 환경과 응답 선호에 대해 기억하고 있는 내용을 요약해줘.
```

### 9.3 무엇을 저장하고 무엇을 저장하지 않을까?

저장하기 좋은 정보:

- “이 프로젝트는 Go 1.22, chi router, sqlc를 사용한다.”
- “테스트는 `make test`로 실행한다.”
- “사용자는 장황한 설명보다 표와 체크리스트를 선호한다.”
- “이 서버는 Docker 명령에 sudo가 필요 없다.”

저장하지 말아야 할 정보:

- 일회성 log dump
- API key, password, token
- 너무 큰 코드 블록
- 다시 검색 가능한 일반 지식
- 이미 AGENTS.md, SOUL.md, README에 안정적으로 적힌 중복 정보

### 9.4 Memory 운영 팁

- 기억은 짧고 구체적이어야 합니다.
- 80% 이상 차면 비슷한 항목을 통합합니다.
- 틀린 기억은 “그 기억은 삭제/수정해줘”라고 명시합니다.
- 개인정보나 secret은 memory에 넣지 않습니다.

---

## 10. Skills 실습

### 10.1 Skills란?

공식 Skills 문서는 skill을 agent가 필요할 때 로드하는 on-demand knowledge document로 설명합니다. 모든 skill은 기본적으로 `~/.hermes/skills/` 아래에 위치하며, fresh install 시 bundled skills가 복사되고, hub 설치 skill과 agent-created skill도 여기에 저장됩니다.

핵심 포인트:

- slash command처럼 호출할 수 있습니다.
- agent가 복잡한 작업을 성공적으로 해결한 뒤 재사용 가능한 절차를 skill로 저장할 수 있습니다.
- `SKILL.md` frontmatter와 본문 절차로 구성됩니다.
- reference, template, script, asset 폴더를 함께 둘 수 있습니다.

### 10.2 Skill 탐색

```bash
hermes skills browse
hermes skills search kubernetes
hermes skills inspect openai/skills/k8s
```

대화창 안에서는 다음처럼 시도합니다.

```text
/skills
```

또는:

```text
/plan 오늘 Hermes Agent 세미나 이후 내가 개인 업무 자동화에 적용할 3단계 계획을 만들어줘.
```

### 10.3 Skill 설치

예시:

```bash
hermes skills install openai/skills/k8s
```

공식 문서의 보안 모델에 따르면 hub-installed skills는 보안 스캔을 거치며, community source는 더 주의해서 검토해야 합니다. 세미나에서는 실제 설치 전 `inspect`를 먼저 수행하도록 안내합니다.

권장 흐름:

```bash
hermes skills search react --source skills-sh
hermes skills inspect skills-sh/vercel-labs/json-render/json-render-react
# 내용을 검토한 뒤에만 설치
hermes skills install skills-sh/vercel-labs/json-render/json-render-react --force
```

주의:

- `--force`는 무조건 안전을 의미하지 않습니다.
- dangerous verdict는 override 대상이 아닙니다.
- 조직 내부 skill tap을 운영할 때는 repo 신뢰도와 review process를 둡니다.

### 10.4 나만의 Skill 설계 실습

오늘은 실제 Hermes 내부 skill 생성까지 강제하지 않고, skill 초안을 설계합니다.

파일 예시: `~/hermes-seminar-lab/SKILL.draft.md`

```markdown
---
name: weekly-pm-briefing
description: Weekly product/tech PM briefing workflow for collecting updates and producing an action-oriented summary.
version: 0.1.0
metadata:
  hermes:
    tags: [pm, briefing, weekly]
    category: productivity
---

# Weekly PM Briefing

## When to Use
- Weekly product/engineering sync preparation
- Before stakeholder update meetings

## Procedure
1. Ask for the project/repo/channel scope.
2. Collect recent changes from approved sources only.
3. Summarize decisions, blockers, metrics, and next actions.
4. Flag uncertain claims as assumptions.
5. Produce a concise markdown brief.

## Pitfalls
- Do not invent dates or owners.
- Do not include secrets or private tokens.

## Verification
- Every action item has owner, due date, and source.
```

실습 프롬프트:

```text
위 SKILL.draft.md를 Hermes Agent의 SKILL.md 형식에 맞게 개선해줘. 단, 실제 ~/.hermes/skills에는 아직 쓰지 말고 초안만 제안해줘.
```

---

## 11. Messaging Gateway 실습/데모

### 11.1 Gateway 개념

공식 Messaging Gateway 문서는 gateway를 여러 메시징 platform을 연결하는 단일 background process로 설명합니다. 플랫폼 예시는 Telegram, Discord, Slack, WhatsApp, Signal, SMS, Email, Home Assistant, Matrix, Mattermost, Microsoft Teams 등입니다.

### 11.2 기본 설정 흐름

```bash
hermes gateway setup
```

wizard에서 플랫폼을 선택하고 token/secret을 입력합니다. 설정이 끝나면 다음 중 하나로 실행합니다.

```bash
hermes gateway
```

서비스로 설치:

```bash
hermes gateway install
hermes gateway start
hermes gateway status
```

중지:

```bash
hermes gateway stop
```

### 11.3 Messaging 안에서 쓰는 명령

| 명령 | 용도 |
|---|---|
| `/new` 또는 `/reset` | 새 대화 시작 |
| `/model [provider:model]` | 모델 확인/변경 |
| `/personality [name]` | personality 설정 |
| `/retry` | 마지막 응답 재시도 |
| `/undo` | 마지막 exchange 제거 |
| `/status` | 현재 session 상태 확인 |
| `/stop` | 실행 중 agent 중단 |
| `/approve`, `/deny` | 위험 명령 승인/거절 |
| `/sethome` | home channel 설정 |
| `/compress` | context 압축 |
| `/usage` | token 사용량 확인 |

### 11.4 세미나 데모 시나리오: Telegram bot

> 실제 bot token 생성은 플랫폼 정책과 개인 계정 상태에 따라 달라질 수 있으므로, 오늘은 강사가 준비한 테스트 bot 또는 화면 녹화로 대체할 수 있습니다.

1. `hermes gateway setup`
2. Telegram 선택
3. bot token 입력
4. allowlist 또는 authorized user 설정
5. `hermes gateway` foreground 실행
6. Telegram에서 “오늘 내 Hermes 세미나 실습 상태를 요약해줘” 메시지 전송
7. `/status`, `/new`, `/usage` 테스트

운영 주의:

- bot token은 `.env` 또는 secure config에 저장합니다.
- 공개 채널에 연결하지 않습니다.
- 개인 DM allowlist를 먼저 설정합니다.
- gateway log를 확인하되 secret redaction 여부를 신뢰만 하지 말고 화면 공유에 주의합니다.

---

## 12. 자동화/cron 확장 아이디어

오늘 세미나에서는 cron까지 깊게 실습하지 않더라도, Hermes의 scheduled automation 방향을 소개합니다.

예시 과제:

```text
매주 월요일 오전 9시에 지난주 GitHub PR, 열린 이슈, 미팅 액션아이템을 요약하는 PM 브리핑 자동화 계획을 세워줘. 실제 cron 등록은 하지 말고 필요한 입력, 권한, 실패 대응만 정리해줘.
```

운영 설계 체크리스트:

- 데이터 소스: GitHub, Slack, Notion, Google Calendar 등
- 권한: read-only token 우선
- 출력 채널: Telegram DM, Slack private channel, email
- 실패 시 알림: gateway status, log, retry 기준
- 개인정보/secret 제외 규칙
- 월 1회 skill/memory 정리 루틴

---

## 13. 장애 대응 플레이북

### 13.1 기본 복구 순서

공식 Quickstart의 Recovery Toolkit 흐름을 세미나용으로 재구성하면 다음 순서가 좋습니다.

```bash
hermes doctor
hermes model
hermes setup
hermes sessions list
hermes --continue
hermes gateway status
```

### 13.2 문제별 대응

| 증상 | 원인 후보 | 대응 |
|---|---|---|
| Hermes가 빈 응답/깨진 응답 | provider auth/model 문제 | `hermes model` 재실행 |
| custom endpoint 응답 이상 | base URL/model name/context mismatch | 별도 OpenAI-compatible client로 endpoint 검증 |
| `hermes --continue` 실패 | 다른 profile, session 저장 실패 | `hermes sessions list` 확인 |
| gateway는 켜졌지만 메시지 미수신 | token, allowlist, platform 설정 | `hermes gateway setup`, `hermes gateway status` |
| tool이 예상대로 안 보임 | toolset 비활성화 | `hermes tools` 확인 |
| skill 설치 실패 | 보안 스캔, GitHub rate limit | `inspect`, `audit`, `GITHUB_TOKEN` 설정 검토 |
| context 초과 경고 | 긴 대화, 파일 과다 주입 | `/usage`, `/compress`, `/new` |

### 13.3 강사용 live-debug 질문

참가자에게 아래 정보를 요청합니다. 단, secret은 절대 공유하지 않게 합니다.

```bash
which hermes
hermes --help | head -40
hermes doctor
hermes config check
```

질문:

- OS가 Linux/macOS/WSL2 중 무엇인가요?
- `hermes model`이 완료되었나요?
- 첫 `hermes chat -q "hello"`가 성공하나요?
- 어떤 provider/model을 선택했나요? API key 값은 말하지 마세요.
- gateway 문제라면 foreground `hermes gateway` 로그에 어떤 non-secret 오류가 보이나요?

---

## 14. 오늘의 최종 개인 과제

각 참가자는 자신에게 맞는 Hermes Agent 개인 워크플로 하나를 설계합니다.

### 14.1 과제 템플릿

```markdown
# My Hermes Agent Workflow

## 목적
- 내가 반복적으로 처리하는 작업:

## 입력 데이터
- 필요한 파일/서비스/채널:

## 사용할 Hermes 기능
- CLI/TUI:
- Tools/toolsets:
- Memory:
- Skills:
- Gateway:
- Cron/automation:

## 보안 제약
- 절대 읽지 말아야 할 경로:
- 사용 가능한 token scope:
- 승인 필요한 명령:

## 성공 기준
- 결과물 형식:
- 검증 방법:
- 실패 시 복구 명령:
```

### 14.2 예시 워크플로

1. **Daily PM Briefing**
   - 아침마다 GitHub/Slack/Calendar 요약
   - Telegram DM으로 전달
   - blockers와 action item 중심

2. **Repo Onboarding Assistant**
   - 새 repository 구조 요약
   - entrypoint, test command, deployment path 찾기
   - AGENTS.md/README/CI 파일 기반 convention 정리

3. **Meeting Follow-up Agent**
   - 회의록 markdown 입력
   - decision/action/risk 분리
   - 다음 회의 agenda 생성

4. **Personal Research Assistant**
   - 특정 기술 주제 공식 문서 검색
   - citation 포함 요약
   - reusable skill로 절차화

---

## 15. 강사용 데모 스크립트

### 15.1 5분 elevator demo

```bash
mkdir -p ~/hermes-seminar-lab
cd ~/hermes-seminar-lab
cat > README.md <<'TXT'
# Hermes Seminar Lab

A safe workspace for testing Hermes Agent.
TXT
hermes --tui
```

프롬프트:

```text
이 폴더를 확인하고, 오늘 핸즈온 참가자에게 보여줄 수 있는 안전한 Hermes Agent 실습 5개를 제안해줘. 파일을 수정하지 말고 계획만 말해줘.
```

### 15.2 10분 memory + session demo

프롬프트 1:

```text
내 선호를 기억해줘: Hermes Agent 설명은 한국어로, 표와 체크리스트 중심으로, 위험 명령은 실행 전 반드시 확인해줘.
```

종료 후:

```bash
hermes --continue
```

프롬프트 2:

```text
내가 Hermes Agent 설명을 어떤 방식으로 받기 원하는지 기억나는 대로 말해줘.
```

### 15.3 10분 skill demo

```bash
hermes skills browse
hermes skills search github
```

프롬프트:

```text
/plan 내 GitHub repository를 대상으로 PR 준비, 테스트 실행, 릴리즈 노트 초안 작성을 돕는 Hermes workflow를 설계해줘.
```

### 15.4 10분 gateway demo

```bash
hermes gateway setup
hermes gateway
```

메시징 앱에서:

```text
/status
```

```text
오늘 Hermes Agent 세미나에서 내가 다음으로 해야 할 실습을 3단계로 알려줘.
```

---

## 16. 운영 체크리스트

### 16.1 세미나 시작 전 강사 체크

- [ ] 공식 문서 URL 열어 최신 Quickstart 확인
- [ ] 설치 명령 정상 접근 확인
- [ ] 테스트 provider 인증 확인
- [ ] 예비 API key 또는 데모 계정 준비
- [ ] 실습 폴더 준비
- [ ] 화면 공유 시 secret 노출 방지
- [ ] gateway 데모용 bot/token 준비 또는 녹화본 준비
- [ ] Docker 데모 여부 결정

### 16.2 참가자 최종 확인

```bash
hermes doctor
hermes chat -q "Hermes Agent가 정상 동작하는지 한 문장으로 답해줘."
hermes --continue
```

### 16.3 세미나 종료 후 권장 정리

```bash
hermes update
hermes skills check
hermes skills audit
hermes config check
```

개인 보안 정리:

- 불필요한 bot token 폐기
- 테스트 API key 삭제 또는 rotation
- `~/.hermes/.env` 권한 확인
- gateway service를 켜둘지 끌지 결정
- memory에 저장된 민감 정보가 없는지 점검

---

## 17. 공식 문서 기반 출처 메모

아래 출처는 2026-05-06 기준으로 세미나 초안 작성 시 확인한 공식 사이트/문서입니다.

1. Hermes Agent 공식 사이트: <https://hermes-agent.org/>
2. 공식 문서 홈: <https://hermes-agent.nousresearch.com/docs>
3. Quickstart: <https://hermes-agent.nousresearch.com/docs/getting-started/quickstart>
4. Installation: <https://hermes-agent.nousresearch.com/docs/getting-started/installation>
5. CLI Interface: <https://hermes-agent.nousresearch.com/docs/user-guide/cli>
6. Configuration: <https://hermes-agent.nousresearch.com/docs/user-guide/configuration>
7. Sessions: <https://hermes-agent.nousresearch.com/docs/user-guide/sessions>
8. Tools & Toolsets: <https://hermes-agent.nousresearch.com/docs/user-guide/features/tools>
9. Persistent Memory: <https://hermes-agent.nousresearch.com/docs/user-guide/features/memory>
10. Skills System: <https://hermes-agent.nousresearch.com/docs/user-guide/features/skills>
11. Messaging Gateway: <https://hermes-agent.nousresearch.com/docs/user-guide/messaging>

---

## 18. 초안 상태 및 보강 TODO

오늘 바로 사용할 수 있도록 상세 실습 흐름을 먼저 구성했습니다. 세미나 전 시간이 더 있으면 다음을 보강하면 좋습니다.

- [ ] 실제 화면 캡처 추가: 설치 완료, `hermes --tui`, `hermes tools`, gateway status
- [ ] provider별 인증 화면 캡처 추가
- [ ] Telegram/Discord 각각 별도 상세 guide 분리
- [ ] Docker backend 실습을 별도 appendix로 확장
- [ ] 조직 내부 보안 정책에 맞춘 allowed/blocked command 예시 추가
- [ ] 참가자 설문 기반 use case별 breakout 과제 추가
