# Claude Squad 프로젝트 아키텍처 분석

## 개요

Claude Squad는 여러 AI 어시스턴트(Claude Code, Aider, Codex, Gemini 등)를 동시에 관리할 수 있는 터미널 기반 애플리케이션입니다. Go 언어로 작성되었으며, tmux와 git worktree를 활용하여 각 AI 세션을 독립적으로 관리합니다.

## 프로젝트 구조 및 핵심 파일 역할

### 1. 메인 진입점
#### `main.go`
- **역할**: 애플리케이션의 메인 진입점
- **주요 기능**:
  - CLI 명령어 처리 (cobra 라이브러리 사용)
  - 플래그 파싱 (`--program`, `--autoyes`, `--daemon`)
  - Git 저장소 검증
  - 데몬 프로세스 관리
  - 메인 애플리케이션 실행

#### `go.mod`
- **역할**: Go 모듈 의존성 관리
- **주요 의존성**:
  - `charmbracelet/bubbletea`: TUI 프레임워크
  - `charmbracelet/lipgloss`: 스타일링
  - `go-git/go-git`: Git 작업
  - `spf13/cobra`: CLI 프레임워크
  - `creack/pty`: 가상 터미널

### 2. 애플리케이션 로직 (`app/`)
#### `app/app.go`
- **역할**: 메인 애플리케이션 로직과 상태 관리
- **주요 기능**:
  - Bubble Tea 모델 구현
  - 홈 화면 상태 관리 (stateDefault, stateNew, statePrompt, stateHelp, stateConfirm)
  - 인스턴스 생성 및 관리
  - UI 이벤트 처리

#### `app/help.go`
- **역할**: 도움말 화면 렌더링
- **기능**: 키보드 단축키 및 사용법 표시

### 3. 설정 관리 (`config/`)
#### `config/config.go`
- **역할**: 애플리케이션 설정 관리
- **주요 설정**:
  - `DefaultProgram`: 기본 AI 프로그램 (claude, aider 등)
  - `AutoYes`: 자동 승인 모드
  - `DaemonPollInterval`: 데몬 폴링 간격
  - `BranchPrefix`: Git 브랜치 접두사
- **설정 파일 위치**: `~/.claude-squad/config.json`

#### `config/state.go`
- **역할**: 애플리케이션 상태 관리
- **기능**: 인스턴스 정보 저장/로드

### 4. 세션 관리 (`session/`)
#### `session/instance.go`
- **역할**: AI 인스턴스 추상화
- **주요 속성**:
  - `Title`: 인스턴스 제목
  - `Path`: 워크스페이스 경로
  - `Branch`: Git 브랜치명
  - `Status`: 상태 (Running, Ready, Loading, Paused)
  - `Program`: 실행할 AI 프로그램
  - `AutoYes`: 자동 승인 여부

#### `session/storage.go`
- **역할**: 인스턴스 데이터 영속성 관리
- **기능**:
  - 인스턴스 저장/로드
  - 상태 파일 관리
  - 인스턴스 생성/삭제

### 5. Git 관리 (`session/git/`)
#### `session/git/worktree.go`
- **역할**: Git worktree 관리
- **주요 기능**:
  - 각 인스턴스별 독립된 워크트리 생성
  - 브랜치 생성 및 관리
  - 변경사항 추적
- **워크트리 위치**: `~/.claude-squad/worktrees/`

#### `session/git/worktree_ops.go`
- **역할**: Git 작업 수행
- **기능**:
  - 커밋 생성
  - 브랜치 푸시
  - 변경사항 확인

#### `session/git/diff.go`
- **역할**: Git diff 생성 및 포맷팅
- **기능**: 변경사항 시각화

#### `session/git/util.go`
- **역할**: Git 유틸리티 함수
- **기능**: Git 저장소 검증, 브랜치명 검증 등

### 6. Tmux 관리 (`session/tmux/`)
#### `session/tmux/tmux.go`
- **역할**: Tmux 세션 관리
- **주요 기능**:
  - Tmux 세션 생성/삭제
  - PTY(Pseudo Terminal) 관리
  - 세션 상태 모니터링
  - 세션 연결/분리

#### `session/tmux/pty.go`
- **역할**: 가상 터미널 관리
- **기능**: PTY 생성 및 입출력 처리

### 7. 데몬 프로세스 (`daemon/`)
#### `daemon/daemon.go`
- **역할**: 백그라운드 데몬 프로세스
- **주요 기능**:
  - AutoYes 모드에서 세션 자동 처리
  - 주기적 세션 상태 확인
  - 자동 응답 처리

#### `daemon/daemon_unix.go` / `daemon/daemon_windows.go`
- **역할**: 플랫폼별 데몬 관리
- **기능**: 프로세스 시작/중지, PID 파일 관리

### 8. UI 컴포넌트 (`ui/`)
#### `ui/list.go`
- **역할**: 인스턴스 목록 UI
- **기능**: 세션 목록 표시 및 선택

#### `ui/menu.go`
- **역할**: 메뉴 UI 렌더링
- **기능**: 키보드 단축키 및 명령어 표시

#### `ui/preview.go`
- **역할**: 세션 미리보기
- **기능**: 터미널 출력 실시간 표시

#### `ui/diff.go`
- **역할**: Git diff 표시
- **기능**: 코드 변경사항 시각화

#### `ui/tabbed_window.go`
- **역할**: 탭 기반 윈도우 관리
- **기능**: 미리보기/diff 탭 전환

#### `ui/overlay/`
- **역할**: 오버레이 UI 컴포넌트
- **기능**: 모달, 확인 대화상자, 입력 폼

### 9. 키보드 입력 (`keys/`)
#### `keys/keys.go`
- **역할**: 키보드 단축키 정의 및 매핑
- **주요 키 바인딩**:
  - `n`: 새 세션 생성
  - `N`: 프롬프트로 새 세션 생성
  - `D`: 세션 삭제
  - `o/Enter`: 세션 연결
  - `c`: 체크아웃 (변경사항 커밋 후 일시정지)
  - `r`: 일시정지된 세션 재개
  - `s`: 변경사항 커밋 및 푸시
  - `tab`: 미리보기/diff 탭 전환
  - `q`: 종료

### 10. 로깅 (`log/`)
#### `log/log.go`
- **역할**: 로깅 시스템
- **기능**:
  - 다중 레벨 로깅 (Info, Warning, Error)
  - 임시 디렉토리에 로그 파일 생성
  - 데몬 모드 지원

### 11. 명령어 실행 (`cmd/`)
#### `cmd/cmd.go`
- **역할**: 외부 명령어 실행 추상화
- **기능**:
  - `exec.Cmd` 래퍼
  - 테스트를 위한 인터페이스 제공

### 12. 웹 인터페이스 (`web/`)
#### `web/package.json`
- **역할**: Next.js 웹 애플리케이션 설정
- **기술 스택**: React 19, Next.js 15.3.2, TypeScript

#### `web/src/app/`
- **역할**: Next.js 앱 라우터 기반 웹 UI
- **기능**: 터미널 앱의 웹 버전 제공

## 핵심 기능 흐름

### 1. 인스턴스 생성 과정
1. 사용자가 `n` 키 입력 → 새 인스턴스 생성 요청
2. `GitWorktree.Create()` → 독립된 Git 워크트리 생성
3. `TmuxSession.Start()` → Tmux 세션 시작
4. AI 프로그램 (claude, aider 등) 실행
5. 인스턴스 정보 저장 (`storage.SaveInstance()`)

### 2. 세션 관리 과정
1. Tmux 세션에서 AI 프로그램 실행
2. PTY를 통한 입출력 처리
3. 상태 모니터링 (Ready, Running, Loading)
4. 사용자 입력을 AI 프로그램으로 전달

### 3. Git 워크플로우
1. 각 인스턴스는 독립된 브랜치 사용
2. 변경사항은 실시간으로 추적
3. `c` 키로 변경사항 커밋 후 세션 일시정지
4. `s` 키로 GitHub에 푸시
5. `r` 키로 일시정지된 세션 재개

### 4. AutoYes 모드
1. 데몬 프로세스가 백그라운드에서 실행
2. 주기적으로 각 세션의 상태 확인
3. AI의 프롬프트를 자동으로 승인
4. 백그라운드에서 작업 진행

## 기술적 특징

### 아키텍처 패턴
- **MVC 패턴**: UI(View), 비즈니스 로직(Controller), 데이터(Model) 분리
- **이벤트 드리븐**: Bubble Tea 프레임워크의 이벤트 기반 아키텍처
- **플러그인 아키텍처**: 다양한 AI 프로그램 지원

### 핵심 기술
- **Bubble Tea**: 터미널 UI 프레임워크
- **Tmux**: 터미널 멀티플렉서
- **Git Worktree**: 독립된 작업 공간
- **PTY**: 가상 터미널
- **Cobra**: CLI 프레임워크

### 확장성
- 새로운 AI 프로그램 추가 용이
- 플랫폼별 구현 분리 (Unix/Windows)
- 웹 인터페이스 병행 제공

이 아키텍처를 통해 Claude Squad는 여러 AI 어시스턴트를 효율적으로 관리하면서도 각 작업을 독립적으로 유지할 수 있습니다.