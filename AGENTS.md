# AGENTS.md — MedMap

## 작업 시작
1. `HANDOFF.md`를 먼저 읽고 현재 상태와 남은 작업을 확인한다.
2. `git status`와 현재 브랜치를 확인한다.
3. HANDOFF.md와 실제 코드나 Git 상태가 다르면 실제 코드와 Git을 따른다.

## 토큰과 컨텍스트 절약
- 프로젝트 전체를 탐색하지 않는다. 파일명, 함수명, 오류 메시지로 먼저 검색하고 관련 파일만 읽는다.
- `.venv/`, `node_modules/`, `local-cache/`, `dist/` 같은 생성물·캐시·의존성 폴더는 기본적으로 읽거나 검색하지 않는다. `data/`를 포함한 프로젝트 데이터는 현재 작업에 필요한 경우에만 확인한다.
- 이미 읽었고 바뀌지 않은 파일은 다시 읽지 않는다. 긴 로그는 필요한 부분만 본다.

## 코드 수정
- 요청한 문제를 해결하는 데 필요한 최소한만 수정한다.
- 요청하지 않은 리팩토링을 하지 않고, 관련 없는 파일을 수정하지 않는다.
- 기존 코드 스타일과 구조를 따르고, 새 파일이나 라이브러리를 만들기 전에 기존 구현을 확인한다.

## 검증
- API: `cd apps/api; .\.venv\Scripts\python.exe -m pytest -q -p no:cacheprovider`
- 웹: `cd apps/web; corepack pnpm build` (이 PC에는 pnpm이 PATH에 없다)
- 수정한 범위에 맞는 테스트부터 실행한다. 실행하지 않은 테스트는 통과했다고 쓰지 않는다.

## Git
- 사용자가 요청하지 않으면 `git commit`과 `git push`를 하지 않는다.
- 이 프로젝트는 저장소 두 곳에 올린다. 사용자가 push를 요청하면 **두 곳 모두** 올린다. (상세: HANDOFF.md 2절)
  1. 개인 저장소 `minu2246/MedMap`(비공개): `feature/stt-rebuild`에 커밋하고, 브랜치를 옮기지 않은 채
     `git fetch . feature/stt-rebuild:main`으로 `main`을 fast-forward한 뒤 `git push origin main feature/stt-rebuild`
  2. 팀 통합 저장소 `KYU-SW/Medmap`(**공개**)의 `Medmap_minwoo/` 폴더:
     `powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\sync_team_repo.ps1`
     (개인 저장소 `main`의 현재 파일을 아래 "개인 저장소 전용" 파일만 빼고 복사해 커밋 하나로 올린다.)
- 팀 저장소는 공개라 올리기 전에 비밀번호·토큰·실제 환자 정보·PC 경로·내부 IP가 없는지 확인한다.
- 팀 저장소의 팀원 폴더(`MedMap/`)와 최상위 파일은 수정하지 않는다.
- **개인 저장소 전용 파일**(2026-10-03 사용자 요청): AI 에이전트와 일하기 위한 규칙·작업 방식·기록 문서는
  개인 저장소에는 올리고 **팀 저장소에는 올리지 않는다**.
  `AGENTS.md`, `CLAUDE.md`, `HANDOFF.md`, `PROJECT_BRIEF.txt`, `TEAM_PROGRESS_REVIEW_2026-09-29.md`,
  `FILE_MANAGEMENT.md`, `LOCAL_STORAGE_PLAN.md`, `SETUP_GIT_AUTH.ps1`.
- `README.md`는 개인 저장소와 팀 저장소에 똑같이 올라간다(2026-10-09 사용자 요청). `README.team.md`(DDXPlus 단계 설명)도 그 이름 그대로 올라간다.
  이 목록은 `scripts/sync_team_repo.ps1`의 `$personalOnly`가 기준이다. AI용 문서를 새로 만들면 그 목록에도 추가한다.

## 팀원 코드 통합
- 팀원 진행 상황은 https://github.com/KYU-SW/Medmap/tree/main/MedMap 에서 확인한다(읽기만 한다).
- 사용자가 통합하자고 말하기 전에는 통합 작업을 하거나 먼저 제안하지 않는다.
- 사용자가 통합을 시작하면 HANDOFF.md 8절의 통합 쟁점을 보여주고 어떻게 할지 먼저 물어본다.

## 삭제
- 프로젝트 원본, 데이터, 설정 파일을 마음대로 삭제하지 않는다.
- 정리나 삭제가 필요하면 삭제할 후보 목록을 먼저 보여주고, 사용자 승인을 받은 뒤 삭제한다.

## 개인정보
- 원본 음성과 변환한 문장을 로그나 Git에 남기지 않는다. (docs/PRIVACY.md)

## 작업 종료
- 기능이나 계획이 바뀌는 커밋을 올릴 때는 `README.md`의 `현재 할 수 있는 것`, `개발 기록`(날짜별), `앞으로 할 일`도 함께 고친다(2026-10-09 사용자 지적).
- `HANDOFF.md`에 변경한 파일, 한 일, 테스트 결과, 남은 작업을 갱신한다.
- 끝내지 못한 작업을 완료했다고 기록하지 않는다.

## 응답
- 사용자에게는 한국어로 간결하게 답한다. 코드, 명령어, 오류 원문은 영어 그대로 둔다.
