# 오목게임

단일 `index.html` 파일로 만든 자유 오목 게임입니다.  
내부는 책임별 5개 모듈로 나누어 구현했습니다.

> 이 프로젝트는 Codex를 사용해 구현하고 정리했습니다.

## 실행 방법

1. `index.html`을 브라우저로 엽니다.
2. 또는 GitHub Pages 배포 주소로 접속합니다.

## 조작 방법

- 바둑판 교차점을 마우스로 클릭해 돌을 놓습니다.
- 검은 돌이 먼저 둡니다.
- `다시 시작` 버튼으로 새 게임을 시작할 수 있습니다.
- `무르기` 버튼으로 마지막 한 수를 취소할 수 있습니다.

## 규칙

- 자유 오목룰을 사용합니다.
- 금수(3-3, 4-4 등)는 적용하지 않습니다.
- 마지막 착수점 기준으로 4방향만 검사해 5목 이상이면 승리합니다.

## 구조

이 프로젝트는 단일 HTML 안에서 다음 5개 모듈로 분리되어 있습니다.

- `main`: 초기화와 이벤트 바인딩
- `Game`: 흐름 제어와 상태 변경
- `GameState`: 보드, 차례, 착수기록, 상태만 보관
- `Rules`: `isValidMove`, `checkWin`, `isDraw` 같은 순수 함수
- `Renderer`: Canvas 렌더링 전담

## 주요 기능

- 15x15 오목판 렌더링
- 흑선공 / 백후공
- 승리 판정
- 승리선 표시
- 다시 시작
- 무르기

## 배포 주소

- GitHub Pages: <https://fkrn75.github.io/omok-game/>
- GitHub Repository: <https://github.com/fkrn75/omok-game>

## 문서

- [아키텍처](docs/ARCHITECTURE.md)
- [개발 계획](docs/DEVELOPMENT_PLAN.md)
- [기능 명세](docs/FUNCTIONAL_SPEC.md)

