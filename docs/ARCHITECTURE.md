# 오목 게임 — 필수 아키텍처 (Architecture)

> 범위: **2인 로컬 대전 오목** (한 기기에서 번갈아 두기)
> 최우선 목표: **가장 빠른 완성** — 단일 HTML 파일, 외부 의존성 0, 빌드 없음

---

## 1. 설계 원칙

| 원칙 | 내용 | 이유 |
|---|---|---|
| 의존성 0 | 프레임워크·빌드 도구·패키지 없음 (Vanilla JS) | `npm install` 없이 즉시 실행 → 가장 빠름 |
| 단일 산출물 | 모든 코드를 `index.html` 한 파일에 포함 | 더블클릭으로 브라우저에서 바로 실행, 배포 = 파일 전달 |
| 오프라인 동작 | 네트워크 불필요 | 로컬 2인 대전이라 서버가 필요 없음 |
| 관심사 분리 | 한 파일 안이라도 **데이터 / 규칙 / 렌더 / 흐름**을 논리 모듈로 분리 | 추후 AI·금수룰·온라인 확장 시 해당 모듈만 교체 |
| 순수 함수 규칙 | 게임 규칙(승리 판정 등)은 부작용 없는 순수 함수 | 단위 테스트 쉬움, AI 탐색에 재사용 가능 |

---

## 2. 기술 스택

- **마크업/렌더:** HTML5 + Canvas 2D API
- **로직:** Vanilla JavaScript (ES6+)
- **스타일:** CSS (`<style>` 인라인)
- **빌드/번들러/프레임워크:** 없음
- **실행 환경:** 최신 데스크톱 브라우저(Chrome/Edge), 파일 더블클릭(`file://`)

---

## 3. 전체 구조 (단일 파일 내부 모듈)

`index.html` 안에서 책임별로 5개의 논리 모듈(객체/IIFE)로 분리한다.

```
┌─────────────────────────────────────────────┐
│                 index.html                   │
│  ┌───────────┐                               │
│  │   main    │  부트스트랩: 초기화 + 이벤트 바인딩  │
│  └─────┬─────┘                               │
│        │ 생성·연결                             │
│        ▼                                      │
│  ┌───────────┐   읽기/쓰기   ┌─────────────┐   │
│  │   Game    │ ───────────► │  GameState  │   │
│  │(Controller)│ ◄─────────── │  (데이터)    │   │
│  └──┬─────┬──┘              └─────────────┘   │
│     │     │                                   │
│ 호출 │     │ 호출                               │
│     ▼     ▼                                   │
│ ┌────────┐ ┌──────────┐                       │
│ │ Rules  │ │ Renderer │                       │
│ │(순수함수)│ │ (Canvas) │                       │
│ └────────┘ └──────────┘                       │
└─────────────────────────────────────────────┘
```

---

## 4. 모듈별 책임 (Responsibility)

| 모듈 | 역할 | 핵심 함수(예시) | 의존 |
|---|---|---|---|
| **GameState** | 게임 데이터 보관(보드·차례·기록·상태). 로직 없음, 순수 데이터 | `board`, `currentPlayer`, `moveHistory`, `status` | 없음 |
| **Rules** | 규칙 판정. **부작용 없는 순수 함수** | `isValidMove(state,x,y)`, `checkWin(board,x,y,player)`, `isDraw(state)` | 없음 |
| **Renderer** | Canvas에 그리기만 담당. 상태를 읽어 화면 출력 | `drawBoard()`, `drawStones(state)`, `highlightLast(move)`, `drawWinLine(cells)` | Canvas, GameState(읽기) |
| **Game (Controller)** | 게임 흐름 제어. 입력→규칙→상태변경→렌더 오케스트레이션 | `handleClick(px,py)`, `placeStone(x,y)`, `switchTurn()`, `undo()`, `reset()` | GameState, Rules, Renderer |
| **main** | 진입점. DOM 준비 후 모듈 생성, 캔버스/버튼 이벤트 바인딩 | `init()` | Game |

> 핵심: **Rules와 Renderer는 GameState를 직접 바꾸지 않는다.** 상태 변경은 오직 Game(Controller)을 통해서만 일어난다 → 흐름이 단방향이라 버그 추적이 쉽다.

---

## 5. 데이터 모델

```js
GameState = {
  size: 15,                    // 보드 크기 (15x15 표준)
  board: number[15][15],       // 0=빈칸, 1=흑(Black), 2=백(White)
  currentPlayer: 1,            // 현재 차례 (1=흑, 2=백). 흑선(흑이 먼저)
  moveHistory: [               // 착수 기록 (무르기·마지막 수 표시용)
    { x, y, player }
  ],
  status: 'playing'            // 'playing' | 'win' | 'draw'
  winner: null,                // 종료 시 1 또는 2
  winLine: null                // 승리한 5개 칸 좌표 배열 (연출용)
}
```

- **좌표계:** `(x, y)` = (열 인덱스, 행 인덱스), 0~14. `board[y][x]` 로 접근.
- **돌 색 표현:** 숫자 1/2 사용 (문자열 비교보다 빠르고 단순).

---

## 6. 데이터 흐름 (단방향)

```
[사용자 클릭]
     │  (canvas mousedown, 픽셀 좌표 px,py)
     ▼
Game.handleClick(px,py)
     │  ① 픽셀 → 격자 좌표 변환 (반올림)
     ▼
Rules.isValidMove(state, x, y)   ── 유효하지 않으면 무시하고 종료
     │  ② 통과
     ▼
GameState 갱신 (board, moveHistory, currentPlayer)
     │
     ▼
Rules.checkWin(...) / isDraw(...)  ── 승/무 판정
     │  ③ 결과로 status 갱신
     ▼
Renderer.draw*()  +  상태바/버튼 UI 갱신
```

매 입력마다 이 한 방향 사이클만 돈다. 상태→화면은 항상 "다시 그리기(redraw)"로 동기화한다.

---

## 7. 핵심 알고리즘 — 5목 판정 `checkWin`

새로 놓은 돌 `(x,y)`를 기준으로 **4개 방향선**만 검사한다(전체 보드 스캔 불필요).

- 방향 4개: 가로 `(1,0)`, 세로 `(0,1)`, 대각 ↘ `(1,1)`, 대각 ↗ `(1,-1)`
- 각 방향에 대해 양쪽(정/역)으로 같은 색 돌을 세어 `연속 개수 ≥ 5` 이면 승리
- 승리 시 연속된 칸 좌표를 모아 `winLine`에 저장 → 연출에 사용

```
연속 개수 = 1(자기 자신)
          + 정방향으로 이어진 같은 색 수
          + 역방향으로 이어진 같은 색 수
```

> **장목(6목 이상) 정책:** 자유 오목룰 기준 = **5목 이상이면 승리**(장목도 승). 렌주룰 금수는 본 범위 밖(향후 확장).

---

## 8. 좌표 변환 (픽셀 ↔ 격자)

```
margin   = 보드 가장자리 여백(px)
cellSize = 격자 한 칸 픽셀 크기

격자→픽셀: pixel = margin + index * cellSize
픽셀→격자: index = round((pixel - margin) / cellSize)
           → 0~14 범위를 벗어나면 무효 클릭으로 처리
```

돌은 교차점(선이 만나는 점)에 놓는다(바둑판과 동일). 클릭 좌표는 **반올림**으로 가장 가까운 교차점에 스냅한다.

---

## 9. 파일 구조

### 권장 (기본) — 단일 파일
```
omok-game/
├─ index.html          ← 마크업 + <style> + <script> 전부 포함 (실행 본체)
├─ README.md
└─ docs/
   ├─ ARCHITECTURE.md
   ├─ FUNCTIONAL_SPEC.md
   └─ DEVELOPMENT_PLAN.md
```

### 확장 옵션 — 기능이 커질 때만 분리
```
omok-game/
├─ index.html
├─ css/style.css
└─ js/
   ├─ state.js      (GameState)
   ├─ rules.js      (Rules)
   ├─ renderer.js   (Renderer)
   ├─ game.js       (Game)
   └─ main.js       (main)
```
> 처음에는 **단일 파일**로 시작한다(가장 빠름). 모듈 경계를 위처럼 명확히 잡아두면, 나중에 파일 분리는 잘라내기/붙여넣기 수준으로 끝난다.

---

## 10. 확장성 고려 (미래 대비, 지금은 구현 안 함)

| 확장 | 추가/변경 지점 | 기존 코드 영향 |
|---|---|---|
| **AI 대전** | `AI` 모듈 추가 → `Game`이 백 차례에 `AI.bestMove(state)` 호출. **Rules 순수함수 그대로 재사용** | GameState/Renderer 무수정 |
| **렌주룰 금수** | `Rules.isValidMove`에 3·3/4·4/장목 금수 판정 추가 | Rules만 수정 |
| **온라인 대전** | GameState 동기화 계층(WebSocket) 추가, `placeStone`을 네트워크 이벤트로 트리거 | Game 흐름 일부 수정 |
| **착수 시간 제한** | `Game`에 타이머 추가 | 독립 추가 |

설계의 핵심 가치: **"데이터/규칙/렌더/흐름 분리" 덕분에 위 확장이 전부 국소 변경으로 가능**하다.
