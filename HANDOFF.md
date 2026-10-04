# MedMap 작업 인계 (HANDOFF)

작성일: 2026-10-01 (최종 갱신 2026-10-04)
브랜치: `feature/stt-rebuild`
마지막 커밋: `git log -1`로 확인한다. 2026-10-04 증상 추출 보강 3차·UI 다듬기·요약의 사라진 증상(4절 26번)까지 두 저장소에 올렸다.
다음 작업: 혼자 할 수 있는 진료 전 작업은 대부분 끝났다. 남은 것은 사용자 결정이 필요한 일이다(아래 "결정 대기"). 사용자가 휴대폰으로 보고 고칠 점을 주면 UI를 더 다듬는다.

결정 대기 (2026-10-03):
- 진료 중/후 기능: 팀원과 방식 합의 후 시작(7절, 8절)
- 팀원 코드 통합: 사용자가 하자고 할 때 8절 쟁점부터 묻는다
- 서버 저장: 사용자 방향은 "각자 휴대폰에 저장, 서버 저장 안 함"(2026-10-04). 남는 과제는 기기 분실 대비 백업, 기기 안 STT·추출, 진료 중 의사 화면으로 전달 방식
- 위험 증상 기준: 배뇨 곤란·쌕쌕거림을 빨간 경고에 넣을지 등은 의료진 검토 필요
- `recordStorage.ts` 테스트: 새 개발 패키지 `fake-indexeddb`를 넣을지

## 1. 프로젝트 개요

MedMap은 환자가 말한 증상과 시간에 따른 변화를 연결하고, 현재 진단과 중요한 정보가
서로 맞지 않는 부분을 다시 확인하도록 돕는 진단 안전망 프로젝트다.
(상세: README.md, PROJECT_BRIEF.txt)

## 2. 저장소와 올리는 방법 (2026-10-02부터)

이 프로젝트는 저장소 두 곳에 함께 올린다.

| 저장소 | 공개 | 위치 | 역할 |
|---|---|---|---|
| https://github.com/minu2246/MedMap | 비공개 | 저장소 전체 | 개인 저장소. 작업 원본 |
| https://github.com/KYU-SW/Medmap | **공개** | `Medmap_minwoo/` 폴더 | 팀 통합 저장소. 팀원 작업은 `MedMap/` 폴더 |

올리는 순서 (사용자가 커밋·push를 요청했을 때만):
1. `feature/stt-rebuild`에 커밋하고 `git fetch . feature/stt-rebuild:main`으로 `main`을 맞춘 뒤(checkout 쓰지 않음) 두 브랜치를 `origin`(개인)에 push한다.
2. `powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\sync_team_repo.ps1`을 실행한다.
   - 개인 저장소 `main`의 현재 파일에서 **개인 저장소 전용 파일**을 뺀 나머지를 팀 저장소 `Medmap_minwoo/`에 복사해
     커밋 하나로 올린다. 커밋 메시지에 반영한 개인 커밋 목록과 `Source-Commit: <개인 main 해시>`를 남긴다.
     (2026-10-02~03에는 `git subtree`로 커밋 이력째 옮겼다. 그 이력은 팀 저장소에 그대로 있다.)
   - 팀 저장소 작업용 복사본은 `local-cache/team-repo/`에 둔다(Git 제외).
   - 로컬 `main`과 `origin/main`이 다르면 멈춘다.

주의:
- 팀 저장소는 공개라 올리는 순간 공개된다. 비밀번호·토큰·실제 환자 정보, PC 경로·내부 IP를 넣지 않는다.
  2026-10-02에 README, `scripts/setup_api.ps1`, `docs/MOBILE_TEST.md`의 사용자 경로와 내부 IP를 일반 표기로 바꿨다.
  예전 커밋에는 원래 값이 남아 있고, 커밋 작성자 이메일(`kmw2246@naver.com`)도 이력과 함께 공개된다(사용자가 이력 전체 공개를 선택).
- 2026-10-02 첫 업로드: 팀 저장소 `4db0537`(개인 `ca4d15c`까지). 이 병합 커밋의 작성자 이름은 `unknown`으로 찍혔다
  (PC 전역 Git 설정에 이름이 없음). 이후 스크립트가 개인 저장소의 user.name·user.email을 팀 복사본에 설정한다.
- 팀원의 `MedMap/` 폴더는 건드리지 않는다.
- **개인 저장소 전용 파일**(2026-10-03 사용자 요청): AI 에이전트와 일하기 위한 규칙·작업 방식·기록 문서는
  개인 저장소에만 올리고 팀 저장소에는 올리지 않는다. `AGENTS.md`, `CLAUDE.md`, `HANDOFF.md`, `PROJECT_BRIEF.txt`,
  `TEAM_PROGRESS_REVIEW_2026-09-29.md`, `FILE_MANAGEMENT.md`, `LOCAL_STORAGE_PLAN.md`, `SETUP_GIT_AUTH.ps1`.
  `README.md`도 개인 저장소 전용이다(2026-10-03, 이모티콘·개발 기록이 있는 소개 페이지). 팀 저장소에는 예전 README를 옮긴
  `README.team.md`가 `README.md`라는 이름으로 올라간다. 팀 README를 고칠 때는 `README.team.md`를 고친다.
  기준 목록은 `scripts/sync_team_repo.ps1`의 `$personalOnly`다. AI용 문서를 새로 만들면 거기에 추가한다.
  개인 GitHub에서 저장소를 새로 받는 에이전트도 이 규칙을 볼 수 있다. 팀 저장소의 예전 커밋(`62487bb`까지)에는
  이 파일들이 남아 있고, 이력은 고쳐 쓰지 않았다.
- 2026-10-03 경과: 처음에는 `.gitignore`로 두 저장소 모두에서 뺐다(`c84ae2d`). 그때 `git checkout main`이 무시 파일을
  지워 `131060d`에서 복구했다. 이후 사용자 요청으로 개인 저장소에는 다시 올리고, 팀 저장소만 스크립트로 제외하게 바꿨다.
  `main` 갱신은 계속 `git checkout` 없이 `git fetch . feature/stt-rebuild:main`으로 한다.


## 2-1. Git 상태 (2026-10-02 갱신)
- 2026-10-04 증상 추출 보강 3차·UI 다듬기·요약의 사라진 증상(4절 26번)은 커밋하고 두 저장소에 push했다. 변경 파일: API `app/services/intake_extractor.py`,
  `tests/test_intake.py`, `tests/test_intake_expanded.py`, `tests/test_intake_scenarios.py`, `tests/test_intake_more_symptoms.py` /
  웹 `src/App.tsx`, `src/styles.css`, `src/symptomOptions.ts`, `src/symptomOptions.test.ts`, `src/visitSummary.ts`, `src/visitSummary.test.ts`,
  `src/recordQuality.test.ts` / `docs/INTAKE_EXTRACTION.md`, `README.md`, `HANDOFF.md`
- 2026-10-03 `recordGroups.ts` 테스트(새 파일 `apps/web/src/recordGroups.test.ts`)는 커밋하고 두 저장소에 push했다. 웹 테스트 54개 통과.
- 2026-10-03 증상 추출 보강 2차·확인 화면 정리(4절 25번)는 커밋하고 두 저장소에 push했다. 변경 파일: API `app/services/intake_extractor.py`,
  `tests/test_intake_more_symptoms.py` / 웹 `src/App.tsx`, `src/styles.css`, `src/symptomOptions.ts`, `src/symptomOptions.test.ts` /
  `docs/INTAKE_EXTRACTION.md`, `README.md`, `HANDOFF.md`
- 2026-10-03 README 개편(4절 24번)은 커밋하고 두 저장소에 push했다. 변경 파일: `README.md`(새로 씀), `README.team.md`(예전 README 이름 변경),
  `scripts/sync_team_repo.ps1`, `AGENTS.md`, `HANDOFF.md`
- 2026-10-03 증상 추출 보강(4절 23번)은 커밋하고 두 저장소에 push했다. 변경 파일: API `app/services/intake_extractor.py`,
  `tests/test_intake.py`, `tests/test_intake_expanded.py`, `tests/test_intake_scenarios.py`, 새 파일 `tests/test_intake_more_symptoms.py` /
  웹 `src/symptomOptions.ts`, `src/symptomOptions.test.ts` / 문서 `docs/INTAKE_EXTRACTION.md`, `HANDOFF.md`
- 2026-10-03 기본 정보·백업 Chrome 확인 기록과 삭제 문구 수정(7절)은 커밋하고 두 저장소에 push했다. 변경 파일: 웹 `src/App.tsx`, `HANDOFF.md`
- 2026-10-03 지난 기록 달력(4절 22번)은 커밋하고 두 저장소에 push했다.
  변경 파일: 웹 `src/App.tsx`, `src/styles.css`, 새 파일 `src/calendar.ts`, `src/calendar.test.ts` / `HANDOFF.md`
- 2026-10-03 화면 나누기(4절 18번), 재디자인(19번), 증상 구분·변화 표시(20번), 요약 화면 정리(21번)는 커밋하고 두 저장소에 push했다.
  변경 파일: 웹 `src/App.tsx`, `src/styles.css`, `src/timeline.ts`, `src/timeline.test.ts`, `index.html`, `HANDOFF.md`
- 2026-10-03 UI 정리 1차(4절 17번)와 진료 전 기능 비교(8절)는 커밋하고 두 저장소에 push했다. 변경 파일: 웹 `src/App.tsx`, `src/styles.css`, `HANDOFF.md`
- 2026-10-03 `scripts/run_web_desktop.ps1`, `run_web_mobile.ps1`가 PATH에 없는 `pnpm` 대신 `corepack pnpm`을 쓰게 고치고,
  `docs/MOBILE_TEST.md`에 인증서 없는 안드로이드 시험 방법을 추가했다. 커밋하고 두 저장소에 push했다.
- 2026-10-03 추가 질문(4절 16번)은 커밋하고 두 저장소에 push했다. 변경 파일: 웹 `src/App.tsx`, `src/styles.css`, `src/symptomOptions.ts`,
  새 파일 `src/followUp.test.ts` / 문서 `docs/INTAKE_EXTRACTION.md`
- 2026-10-03 AI 규칙·기록 문서 8개: 팀 저장소에서만 빼고 개인 저장소에는 다시 올렸다(2절, 커밋·두 저장소 push).
  변경 파일: `.gitignore`, `scripts/sync_team_repo.ps1`, 다시 추적하는 문서 8개
- 2026-10-02 말에서 기본 정보 찾기(4절 15번)는 커밋하고 두 저장소에 push했다. 변경 파일: API `app/services/intake_extractor.py`,
  `app/schemas/intake.py`, 새 파일 `tests/test_intake_profile.py` / 웹 `src/App.tsx` / 문서 `docs/INTAKE_EXTRACTION.md`, `HANDOFF.md`
- 2026-10-02 STT 힌트·기본 정보(4절 14번)는 커밋하고 두 저장소에 push했다. 변경 파일: API `app/services/stt_service.py`, `scripts/transcribe_file.py`,
  `.env.example` / 웹 `src/App.tsx`, `src/styles.css`, `src/recordGroups.ts`, `src/backup.ts`, `src/visitSummary.ts`, `src/recordQuality.test.ts` /
  문서 `docs/STT_SPEC.md`, `HANDOFF.md`
- 2026-10-02 기록 품질 보완(4절 13번)은 커밋하고 두 저장소에 push했다. 변경 파일: API `app/services/intake_extractor.py`, `app/schemas/intake.py`,
  `app/api/routes/intake.py`, `tests/test_intake.py`, 새 파일 `tests/test_intake_record_quality.py` / 웹 `src/App.tsx`, `src/styles.css`,
  `src/recordStorage.ts`, `src/symptomOptions.ts`, `src/symptomEpisodes.ts`, `src/timeline.ts`, `src/visitSummary.ts`, `src/symptomOptions.test.ts`,
  새 파일 `src/recordQuality.test.ts` / 문서 `docs/INTAKE_EXTRACTION.md`, `HANDOFF.md`

- CLAUDE.md는 `291f13e`에서 커밋했다. HANDOFF.md와 AGENTS.md는 `22aae26`에서 커밋했다.
- 2026-10-01 작업(아래 4절 8~9번, 6절, 7절)은 `47d78ee`에서 커밋하고 push했다. 변경 파일:
  - 문서: `HANDOFF.md`, `docs/INTAKE_EXTRACTION.md`, `docs/PRIVACY.md`
  - API: `app/services/intake_extractor.py`, `app/schemas/intake.py`, `tests/test_intake.py`,
    `tests/test_intake_scenarios.py`, 새 파일 `tests/test_intake_expanded.py`
  - 웹: `package.json`, `pnpm-lock.yaml`, `src/App.tsx`, `src/styles.css`, `src/recordStorage.ts`,
    `src/recordGroups.ts`, `src/visitSummary.ts`, 새 파일 `src/symptomOptions.ts`, `src/backup.ts`,
    `src/testRecords.ts`, 테스트 `src/*.test.ts` 5개
- 2026-10-02: `feature/stt-rebuild`를 main에 fast-forward로 병합하고 원격 main에도 푸시했다(`7b989e5`).
  같은 날 `f5acbec`까지 다시 병합·푸시했다. main과 feature/stt-rebuild는 같은 커밋을 가리킨다.
- 2026-10-02 합성어·구어 인식 확장, 주어 이어받기, 경고 증상 추가, 불확실·다른 사람·위험 증상 처리(아래 4절 10~12번)는
  `f5acbec`에서 커밋하고 push했다. 변경 파일:
  - API: `app/services/intake_extractor.py`, `app/schemas/intake.py`, `tests/test_intake.py`, `tests/test_intake_scenarios.py`,
    `tests/test_intake_expanded.py`, 새 파일 `tests/test_intake_red_flags.py`, `tests/test_intake_safety.py`
  - 웹: `src/App.tsx`, `src/styles.css`, `src/recordStorage.ts`, `src/symptomOptions.ts`, `src/symptomEpisodes.ts`,
    `src/timeline.ts`, `src/visitSummary.ts`, `src/backup.ts`, `src/backup.test.ts`, `src/symptomOptions.test.ts`, 새 파일 `src/safety.test.ts`
  - 문서: `docs/INTAKE_EXTRACTION.md`, `HANDOFF.md`
- 원격 저장소: https://github.com/minu2246/MedMap (비공개)

## 3. 구성

- `apps/api`: FastAPI 서버
  - `GET /health`
  - `POST /v1/stt/transcribe`: faster-whisper 1.2.1로 음성을 글자로 바꾼다.
    모델은 첫 요청 때 불러오고, NVIDIA 라이브러리가 있으면 GPU를 쓴다.
  - `POST .../intake/extract`: 규칙 기반으로 증상 정보를 뽑는다(`app/services/intake_extractor.py`).
- `apps/web`: React와 Vite 화면. 녹음, 텍스트 입력, 증상 확인, 기록 저장, 타임라인,
  진료 전 요약, PDF와 QR 공유를 제공한다.
  - `recordStorage.ts` 기록 저장(브라우저 IndexedDB) / `recordGroups.ts` 증상 그룹 / `symptomEpisodes.ts` 에피소드 묶음
  - `timeline.ts` 증상 변화 타임라인 / `visitSummary.ts` 진료 전 요약
  - `symptomOptions.ts` 지원 증상 목록, 횟수 기록 대상 / `backup.ts` 기록 백업 파일 만들기와 검증
- 루트의 `01_download.py`, `02_inspect.py`, `03_audit.py`: DDXPlus 다운로드, 구조 확인, 통계 감사 (연구 트랙)
- `scripts/`: 실행, 설치, 모바일 HTTPS, 임시 터널, 저장소 감사와 정리, 팀 저장소 올리기(`sync_team_repo.ps1`)용 PowerShell 스크립트

## 4. 지금까지 진행한 작업 (feature/stt-rebuild)

1. STT 1차 기능: 로컬 Whisper 연동, 브라우저 WebM 코덱 허용, 녹음 중 실시간 자막
2. 증상 추출 v1: 지원 증상 7개(두통, 발열, 기침, 호흡곤란, 가슴 답답함, 복통, 구토)
   - 있음과 없음, 확인되지 않음을 구분하고 원문 근거를 남긴다.
   - 시작 시점, 정도(반복 표현 포함), 복용약, 알레르기, 숫자 통증 점수를 뽑는다.
   - 시간과 정도는 가장 가까운 증상에만 연결한다.
   - 지원하지 않는 표현은 "자동으로 정리하지 못한 표현"으로 따로 보여준다.
3. 기록 기능: 사용자가 확인한 기록을 브라우저에 저장, 증상 그룹 분리와 삭제, 에피소드 묶음, 변화 타임라인
4. 증상 경과: 좋아짐, 나빠짐, 사라짐 표현 인식. 빈도는 구토에만 기록했다(2026-10-01부터 설사도 기록).
5. 진료 전 요약: PDF와 QR로 공유한다. 기록 그룹을 바꾸면 QR을 지운다.
6. 최근 수정: 같은 기록이 두 번 저장되거나 그룹 없이 저장되는 문제를 막았다(`f06d3cf`).
7. 개발 환경: HTTP 데스크톱 모드, 모바일 HTTPS, 원격 시험용 임시 터널, 저장소 감사와 정리 스크립트
8. 2026-10-01 증상 추출 확장 (상세: docs/INTAKE_EXTRACTION.md)
   - 지원 증상 7개 → 21개: 흉통, 인후통, 콧물, 코막힘, 가래, 오한, 근육통, 요통, 어지러움,
     메스꺼움, 설사, 변비, 발진, 피로 추가
   - 과거력(`medical_history`): 정해진 질환 이름 + 있다/진단/앓다, 수술 경험. 부정하면 제외
   - 발열에 체온(`38.5℃`) 기록, 횟수는 구토와 설사에 기록
   - 버그 수정: `열흘`·`해열제`·`열심히`를 발열로 잡던 문제, `하루 3번`의 `하루`를 시작 시점 `1일`로 잡던 문제
   - 미인식 표현을 문장 전체가 아니라 해당 부분만 보여준다
9. 2026-10-01 진료 전 화면 보완
   - 증상 직접 추가와 삭제(이름이 비면 저장 막음), 복용약·알레르기·과거력 직접 수정
   - 요약, PDF/QR 글, 저장 기록에 과거력 표시
   - 전체 기록 백업 파일(JSON) 내보내기와 가져오기. 같은 id 기록은 건너뛴다. 서버로 보내지 않는다.
10. 2026-10-02 합성어·구어 인식 확장 (상세: docs/INTAKE_EXTRACTION.md)
   - 지원 증상 21개 → 32개: 두근거림, 저림, 소화불량, 속쓰림, 식욕부진, 불면, 부종, 귀 통증, 눈 통증, 치통, 식은땀
   - `머리통증`·`허리 통증` 같은 부위+통증, `목감기`·`코감기`·`배탈`·`위경련`·`체했어요`, `숨이 가빠요`, `몸이 무거워요`
   - `살살`·`쿡쿡`·`욱신욱신`·`엄청` 같은 꾸밈말이 사이에 있어도 인식
   - 버그 수정: `두통약`·`기침약`·`설사약`·`변비약`을 증상 있음으로 잡던 문제
   - 미인식 경고 신호어 확대: `감기에 걸렸어요`, `아이가 아파요` 같은 말이 경고 없이 사라지지 않게 함
11. 2026-10-02 주어 이어받기와 경고 증상 확장 (상세: docs/INTAKE_EXTRACTION.md)
   - `가슴이 답답하고 아파요`처럼 주어 없는 뒷부분은 앞 절의 주어를 이어받아 흉통도 찾는다. `많이`, `지금은`은 주어로 보지 않는다
   - 지원 증상 32개 → 49개: 가려움, 시야 이상, 떨림, 이명, 코피, 객혈, 토혈, 혈변, 혈뇨, 배뇨통, 빈뇨, 기절, 마비, 말 어눌함, 쉰 목소리, 체중 감소, 경련
   - 피부 가려움은 발진에서 분리해 가려움으로 기록한다
   - 과거력: `위염으로 진단받았어요`처럼 목록 밖 병명도 기록. 녹내장, 대상포진 등 질환 이름 추가, `대상포진을 앓았어요`의 `을`도 인식
   - 버그 수정: 같은 증상이 문장에 두 번 나오면 두 번째 표현을 미인식으로 보여주던 문제
12. 2026-10-02 불확실한 말, 다른 사람 증상, 위험 증상 안내 (상세: docs/INTAKE_EXTRACTION.md)
   - 증상 상태에 `uncertain`(확실하지 않음) 추가. `잘 모르겠어요`, `같기도 하고` 등. 원인을 모르겠다는 말은 제외
   - `남편도 기침을 해요`, `엄마가 고혈압이 있어요`는 환자 증상·과거력이 아니라 `others_symptoms`로 분리. 화면에서 본인 증상으로 옮길 수 있음
   - 위험 증상(흉통, 객혈, 토혈, 혈변, 기절, 마비, 말 어눌함, 경련, 심한 호흡곤란·두통·복통)이 있으면 확인 화면과 요약에 119·응급실 안내
   - 마침표 없는 STT 문장도 `~요` 뒤와 `근데`·`그런데`·`그래서`에서 절을 나눈다
   - 버그 수정: `기침을 안 해요`·`설사를 안 했어요`(을/를 + 부정)를 있음으로 잡던 문제, `아이고 머리야`의 두통을 놓치던 문제
13. 2026-10-02 기록 품질: 시작 날짜, 정도, 위치, 약 (상세: docs/INTAKE_EXTRACTION.md)
   - 요청에 `reference_date`(사용자 오늘 날짜)를 받아 하루가 정해지는 시작 시점을 `onset_date`로 변환. `10월 1일`, `지난주 월요일` 인식
   - 주어를 이어받은 증상이 앞 절의 시작 시점도 이어받는다
   - 정도: `참을 만해요`·`살짝`(경미함), `보통`(중간), `못 참겠어요`·`잠을 못 잘 정도`·`데굴데굴`(심함)
   - 위치: `오른쪽 아랫배`, `윗배`, `뒷머리`, `왼쪽 무릎`, 저림·부종 등은 주어에서 부위(`왼쪽 팔`). `관절 통증` 추가(증상 50개)
   - 약: 흔한 약 이름·약 이름 끝말 인식, 용량·복용 시점 기록(`타이레놀 500mg 하루 2회`). `페니실린 알레르기`는 약에서 제외
   - 웹: 날짜 표시(`어제부터 (10월 1일)`), 부위 입력칸과 기록·타임라인·요약의 부위 표시, `중간` 정도 비교
14. 2026-10-02 STT 의료 용어 힌트와 기본 정보
   - STT: `MEDMAP_STT_HOTWORDS=medical`로 켜면 증상·약 이름을 Whisper `hotwords`로 넘긴다. **기본 꺼짐, 효과 미측정**.
     실제 녹음으로 `scripts.transcribe_file --hotwords medical` 켠/끈 결과를 비교한 뒤 기본값을 정한다(docs/STT_SPEC.md).
     실제 모델에 무음 1초로 호출해 `hotwords`를 받아들이는 것만 확인했다.
   - 기본 정보: 기록 묶음마다 나이·성별·임신 가능성(여성만)·흡연·음주를 고르는 접이식 칸. localStorage에 묶음과 함께 저장하고,
     진료 전 요약 첫 줄(`기본 정보: 34세 / 여성 / ...`)과 백업 파일에 들어간다. 바꾸면 만든 QR은 지운다.
     **브라우저에서 아직 열어 보지 않았다.**
15. 2026-10-02 말에서 기본 정보 찾기 (상세: docs/INTAKE_EXTRACTION.md)
   - 응답에 `profile`(나이·성별·임신·흡연·음주) 추가. 다른 사람이 주어인 말(`남편이 담배를 피워요`)과 `3살 때`는 제외
   - 웹: 확인 화면에 `말에서 찾은 기본 정보` 표시, 저장하면 기록 묶음의 기본 정보 칸에 반영(`반영하지 않기` 가능)
   - 버그 수정: `34살이에요`를 체중 감소 신호어(`살이`)로 오인해 미인식으로 띄우던 문제
   - 2026-10-03 Chrome(PC)에서 확인: `저는 34살 여자고 담배는 안 피워요. 술은 가끔 마셔요. 어제부터 머리가 아파요` → 확인 화면에
     `34세 / 여성 / 비흡연 / 음주`, 저장 후 홈 `기본 정보` 버튼에 반영. API로 `남편이 담배를 피워요`·`3살 때`는 제외, `임신 중`·`끊었어요`(과거 흡연) 인식 확인
16. 2026-10-03 진료 전 추가 질문 (상세: docs/INTAKE_EXTRACTION.md)
   - 확인 화면에 `추가로 알려 주세요`: 증상마다 빠진 시작 시점·정확한 부위·정도·횟수를 질문. 선택지 또는 직접 입력, 건너뛰기 가능
   - 시작 시점 답은 추출 API로 다시 읽어 날짜까지 채운다
   - Chrome 확인(저장 안 함): `배가 아파요` → 질문 3개, `오른쪽 아랫배` 선택, `어제 저녁부터` 입력 → `어제 저녁부터 (10월 2일)`
17. 2026-10-03 UI 정리 1차 (기능 변경 없음, 배치만)
   - 맨 위에 1 증상 말하기 → 2 정리된 내용 확인 → 3 진료 전 요약 보여 주기 단계 표시(현재 단계 강조)
   - 각 영역 제목을 `1. 증상을 말하거나 적어 주세요`, `2. 정리된 내용을 확인해 주세요`, `3. 진료 전 요약`으로 바꿈
   - 기록 묶음 선택은 `현재 기록: …` 한 줄로 접고, 발생 기간·타임라인·저장된 기록·백업은 `지난 기록 보기` 안에 접음
   - 인쇄(PDF)는 그대로 진료 전 요약만 나온다(요약 영역은 감싸지 않음). Chrome 데스크톱 화면으로 배치를 확인했다.
18. 2026-10-03 화면 나누기 (사용자가 직접 보고 판단 예정)
   - 한 화면을 주소(`#/…`)로 나눈 여러 화면으로 바꿈: 홈, `#/record`(증상 말하기), `#/review`(확인하기), `#/summary`(진료 전 요약),
     `#/history`(지난 기록·백업), `#/profile`(기본 정보). 휴대폰 뒤로 가기가 동작한다. 새 라이브러리 없음.
   - 정리하기를 누르면 확인하기로, 저장하면 진료 전 요약으로 자동 이동. 홈에는 `새 증상 기록하기` 큰 버튼과 요약·지난 기록·기본 정보 버튼
   - Chrome으로 홈 → 기록 → 확인(추가 질문 3개) → 뒤로 가기 → 지난 기록 → 기본 정보 → 요약 이동을 확인했다(저장은 안 함).
   - 주의: 이 대화에서 백그라운드로 띄운 dev 서버는 작업을 멈춰도 node·python 프로세스가 남았다(5173~5176, 8000 확인).
     테스트 후에는 `Get-NetTCPConnection -State Listen -LocalPort 5173,8000`으로 확인하고 PID를 직접 끈다.
19. 2026-10-03 재디자인 (UI/UX Pro Max 스킬, 의료 서비스용 신뢰감·가독성)
   - 디자인 시스템 추천(healthcare, Minimalism & Swiss, 차분한 청록+건강 녹색)을 `styles.css` 색·간격 토큰(`:root --color-*`)으로 적용.
     추천 청록 `#0891B2`는 흰 글자 대비 약 3.7:1이라 버튼은 `#0E7490`(약 5.4:1)로 진하게 했다.
   - 글꼴: 추천 Atkinson Hyperlegible은 한글이 없고 외부 로딩이 필요해 쓰지 않음. 기기 한글 글꼴(Pretendard → Apple SD Gothic Neo →
     맑은 고딕 → Noto Sans KR), 본문 17px·줄 간격 1.6. 추천 패턴(Hero+후기)은 랜딩용이라 제외.
   - 버튼·입력칸 높이 48px 이상, 키보드 포커스 테두리, 동작 줄이기 설정 존중, 작은 화면에서 카드 테두리 없이 전체 폭.
   - 안내 상자 4종(추가 질문 청록, 다른 사람 파랑, 미인식 주황, 위험 증상 빨강)을 같은 모양+제목으로 통일(색만으로 구분하지 않음).
   - `다음: 증상 정리하기`를 주 버튼으로. `index.html`에 `theme-color`. 클래스 이름·기능·인쇄 규칙은 그대로.
   - Chrome에서 홈·증상 말하기·확인하기 화면을 확인했다(저장 안 함). 휴대폰 실기기는 사용자가 확인 예정.
20. 2026-10-03 증상 구분과 변화 표시 (사용자 요청: 증상 이름이 잘 구분되지 않고, 변화가 글로만 되어 있음)
   - 확인 화면: 증상 카드 맨 위에 이름(22px)과 상태 배지(있음·없음·확실하지 않음)·부위 배지. 위험 증상 카드는 왼쪽 빨간 테두리
   - 진료 전 요약·타임라인: 긴 한 줄 글 대신 증상별 카드 + 상태·변화 배지 + 시작·정도·횟수 칸(`dl.facts`)
   - 변화 배지: `timeline.ts`의 `changeTone`(new·worse·better·same·unknown) → `+`·`↑`·`↓`·`=`·`?`와 색. `있음 → 없음`은 화면에 `사라짐`,
     `없음 → 있음`은 `다시 생김`으로 보인다(저장되는 글은 그대로). 웹 테스트 44개(`changeTone` 추가). Chrome으로 확인(저장 안 함).
21. 2026-10-03 진료 전 요약 화면 정리 (사용자 요청: 버튼과 복용약·알레르기·과거력이 글로만 되어 있음)
   - 기록 기간은 맨 위 정보 줄, 기본 정보는 칩 블록으로 맨 위에
   - 알레르기·복용약·과거력(+ 확실하지 않은 증상, 주변 사람)을 제목 있는 블록 + 칩(`ChipBlock`)으로. 알레르기는 있을 때만 빨간 강조
   - 요약 복사·PDF로 저장·QR 만들기를 SVG 아이콘 + 짧은 설명이 있는 타일 버튼(`action-tile`)으로. 기능은 그대로
   - Chrome으로 요약 화면 확인(저장 안 함). 웹 테스트 44개 통과
22. 2026-10-03 지난 기록을 달력으로 (사용자 요청: 지난 기록 화면이 지저분함)
   - `#/history` 맨 위에 월 달력(`calendar.ts`의 `monthGrid`, 라이브러리 없음). 기록한 날에 점: 그날 가장 눈에 띄는 변화
     (악화 > 새 증상 > 비슷함 > 호전) 색. 증상 기간(첫 기록 ~ 사라짐 또는 마지막 기록)은 칸 배경을 연한 주황으로 칠한다
   - 증상 발생 기간 카드 → 달력 아래 한 줄 요약(이름·진행 중/종료됨·기간·최고 정도)
   - 날짜를 누르면 그날 기록만 아래에 표시(처음엔 가장 최근 기록한 날). 전체 타임라인 나열은 없앰
   - 저장된 기록 목록·삭제·백업은 맨 아래 `저장된 기록 관리·백업` 접기 안으로
   - `calendar.test.ts` 4개 추가, 웹 테스트 48개 통과, `pnpm run build` 통과. 사용자가 휴대폰으로 확인했다(이전보다 낫다는 평가).
23. 2026-10-03 증상 추출 보강 (상세: docs/INTAKE_EXTRACTION.md)
   - 오분류 수정: `소변이 안 나와요`가 변비(`소변`의 `변`), `눈이 충혈됐어요`가 눈 통증으로 기록되던 문제
   - 새 증상 6개(50 → 56): 입안 통증, 재채기, 쌕쌕거림, 배뇨 곤란, 눈 충혈, 생리통. 관절 통증에 손가락·발가락·턱 추가
   - 위험 증상(빨간 경고)에는 넣지 않았다. 배뇨 곤란·쌕쌕거림을 경고로 할지는 의료진 검토가 필요하다
   - 기존 테스트 3개는 `혀가 아파요` 등을 지원하지 않는 예로 쓰고 있어 `입술이 부었어요`·`겨드랑이가 아파요`·`목이 뻐근해요`로 바꿨다.
     새 테스트 `tests/test_intake_more_symptoms.py`. API 테스트 342개, 웹 테스트 48개 통과. 웹 증상 목록(`symptomOptions.ts`)도 56개로
   - `2주 전`·`엊그제`는 하루로 정해지지 않아 지금처럼 날짜로 바꾸지 않는다(정확하지 않은 날짜가 기록에 찍히는 것을 피함)
24. 2026-10-03 개인 저장소 README 개편 (사용자 요청: 이모티콘, 가독성, 배경·목표, 일자별 개발 기록)
   - `README.md`: 배경 → 목표(진료 전·중·후·연구 상태 표) → 현재 기능 → 구성 → 실행 → 개인정보 → 날짜별 개발 기록 → 참고 문서
   - 개인 저장소에만 둔다. 예전 README(DDXPlus 단계 설명)는 `README.team.md`로 옮겼고, 팀 저장소에는 그 파일이 `README.md`로 올라간다
     (`sync_team_repo.ps1`이 복사 뒤 이름을 바꿈). 새 기능을 만들면 README의 개발 기록에도 날짜별로 한 줄 추가한다
25. 2026-10-03 증상 추출 보강 2차 + 확인 화면 정리
   - 띄어쓰기 없는 문장: 입력을 정리할 때 몸 부위+이·가 뒤, `~부터`·`~째`·`~전` 뒤의 몸 부위 앞, 아픔 표현 앞의 `~고` 뒤에
     공백을 넣는다(`_space_glued_words`). `가슴이답답하고아파요` → 가슴 답답함 + 흉통, `어제부터가슴이…`도 시작 시점까지.
     `발가락`·`손가락`은 나누지 않는다
   - 새 증상 2개(56 → 58): 청력 저하(`귀가 잘 안 들려요`), 입마름(`입이 말라요`, `갈증`). `저릿저릿`은 저림
   - 확인 요청 신호어 추가: `배가 불편해요`, `혹이 생겼어요`, `입술이 부르텄어요`가 이제 확인 요청으로 나온다(`혹시`는 제외)
   - 확인 화면: 증상 카드는 상태·부위·변화 배지와 시작·정도(·횟수)만 보이고 입력칸은 `고치기`(접기) 안으로.
     직접 추가한 증상은 펼친 채로 나온다. `자동으로 정리하지 못한 표현` 경고를 위험 증상 경고 바로 아래로 올림
   - 버그 수정: 카드 key가 증상 이름이라 이름을 고칠 때 한 글자마다 입력칸 커서가 빠졌다 → 순번으로
   - API 테스트 355개, 웹 테스트 48개 통과, 빌드 통과. Chrome(PC)으로 확인 화면·고치기·직접 추가·이름 입력을 확인(저장 안 함)
26. 2026-10-04 증상 추출 보강 3차 + UI 다듬기
   - 새 증상 7개(58 → 65): 복부 팽만, 목 이물감, 눈 분비물, 목 결림, 손발 차가움, 우울감, 불안감(`불안정` 제외).
     트림 → 소화불량, `몸에 힘이 없어요` → 피로, `침 삼킬 때 아파요` → 인후통, 입술·눈 부음 → 부종
   - 과거력: `혈압·혈당·콜레스테롤이 높아요`(○○ 높음, `높지 않아요` 제외), `간·신장·심장·폐·갑상선이 안 좋아요`(○○ 질환),
     `○○으로 입원했어요`(○○(입원), `병원으로` 제외), `시술 받았어요`. `우울증 진단`은 과거력만(지금 증상 아님).
     약: `고지혈증 약`처럼 띄어 쓴 약 이름
   - 기존 테스트 3개의 "지원하지 않는 증상" 예시를 `엉덩이가 아파요`·`옆구리가 결려요`로 바꿈. API 테스트 382개, 웹 55개 통과
   - UI: 홈에 `진행 중인 증상` 카드(증상 · 시작일, 마지막 기록 날짜), 달력에 오늘 표시(밑줄, `aria-current`),
     증상 기간 줄은 진행 중이면 `10월 1일부터`, 하루짜리는 날짜 하나. 맨 위 `현재 기록:` → `기록 묶음:`
   - 진료 전 요약(사용자와 상의, 2026-10-04): 사라진 증상은 없애지 않고 `지금 있는 증상`(카드) 아래 `사라진 증상`에 한 줄씩
     (`구토 · 10월 1일 · 4회`). 사라졌어도 위험 증상이었으면 빨간 `위험 증상` 표시(심한 두통 포함). 복사·PDF·QR 글도 같은 순서.
     이유: 경과 자체가 진단 단서이고, 잠깐 있다 사라진 위험 증상(기절·혈변 등)을 숨기면 안 된다. 기간 글은 `visitSummary.ts`의 `episodePeriod`
   - Chrome(PC, 폭 약 650px)으로 홈·지난 기록 확인(저장·삭제 안 함, 그 브라우저에 이미 있던 기록 2개로 봄)

## 5. 실행 방법

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\setup_api.ps1        # 처음 한 번
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run_api.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run_web_desktop.ps1  # http://127.0.0.1:5173
```

- 휴대폰 시험: docs/MOBILE_TEST.md, docs/TEMPORARY_REMOTE_TEST.md
- 모델, 가상환경, 캐시는 Git에서 제외한 `local-cache/`와 `apps/api/.venv`에 둔다. (LOCAL_STORAGE_PLAN.md)

## 6. 테스트 결과 (2026-10-01 실행)

### API 테스트: 통과

```powershell
cd apps\api
.\.venv\Scripts\python.exe -m pytest -q -p no:cacheprovider
```

- 결과: **325 passed, 1 warning** (2026-10-02 기본 정보 찾기 추가 후). 기록 품질 보완 후 307, 불확실·다른 사람 처리 후 260, 경고 증상 확장 후 242, 합성어 확장 후 192, 2026-10-01 확장 후 143, 그 전 99였다.
- 경고: `StarletteDeprecationWarning`. `starlette.testclient`에서 `httpx`를 쓰는 방식이 deprecated라는 내용이다. 테스트 결과에는 영향이 없다.
- 테스트 파일: `test_api.py`, `test_intake.py`, `test_intake_combinations.py`, `test_intake_scenarios.py`, `test_intake_expanded.py`
- 기존 테스트 2개를 고쳤다. "허리 통증, 어지러움, 설사, 발진은 지원하지 않는다"를 확인하던 테스트로,
  이제 지원하므로 손 저림, 눈 침침함, 두근거림처럼 아직 지원하지 않는 표현으로 바꿨다.

### 웹 빌드: 성공

```powershell
cd apps\web
corepack pnpm build   # 이 PC에는 pnpm이 PATH에 없어 corepack으로 pnpm 11.19.0을 실행했다
```

- 결과: **성공 (exit code 0)**. `tsc -b`와 vite v8.3.1 빌드를 모두 통과했다. 모듈 49개, JS 268.89 kB(gzip 84.80 kB).
- 빌드 결과물 `dist/`는 Git에서 제외되어 있다. 빌드 후 `git status`에 변경 사항이 없었다.
- 2026-10-01 화면 보완 후 다시 빌드: 성공. JS 275.22 kB(gzip 86.62 kB). 테스트 파일도 `tsc -b` 타입 검사 대상이다.

### 웹 단위 테스트: 통과 (2026-10-01 추가)

```powershell
cd apps\web
corepack pnpm test   # vitest run
```

- 결과: **8 files, 44 tests passed** (2026-10-03 변화 배지 후). 추가 질문 후 43. 기본 정보 추가 후 40, 기록 품질 보완 후 37
- 대상: `buildTimeline`, `buildSymptomEpisodes`, `buildVisitSummary`, `visitSummaryText`, `backup.ts`, `symptomOptions.ts`(위험 증상 판정 포함)
- 아직 자동 테스트하지 않은 것: `recordStorage.ts`(IndexedDB), `recordGroups.ts`(localStorage), `App.tsx` 화면

### PC 브라우저 수동 확인 (2026-10-01, Chrome, 텍스트 입력)

- 확인한 것: 새 증상 인식(인후통·발열 38.5℃·설사 하루 3회), 과거력 `고혈압`, 미인식 표현 `손이 저려요.`만 표시,
  증상 직접 추가, 이름이 비면 저장 차단, 알레르기 직접 수정, 저장 후 요약에 과거력 표시
- 시험용으로 만든 기록 묶음은 확인 후 삭제했다. 기존 기록 묶음은 건드리지 않았다.
- 확인하지 않은 것: 백업 내보내기/가져오기 버튼 실제 동작(파일 다운로드·선택 창이 필요해 단위 테스트로만 확인), 음성 입력, 휴대폰 화면

### PC 브라우저 수동 확인 (2026-10-02, Chrome, 텍스트 입력, 저장하지 않음)

- 문장 `어제부터 가슴이 답답하고 아파요 열이 있는 것 같기도 하고 잘 모르겠어요 남편도 기침을 해요`
  → 가슴 답답함·흉통 있음, 발열 확실하지 않음, 남편 기침은 따로 표시, 흉통 위험 증상 안내 표시
- `본인 증상으로 옮기기`를 누르면 기침이 환자 증상으로 옮겨지는 것을 확인했다.
- 진료 전 요약의 위험 증상 안내는 기존 기록 묶음의 심한 복통으로 표시되는 것을 확인했다.


### PC 브라우저 수동 확인 (2026-10-02 기록 품질, Chrome, 저장하지 않음)

- `어제부터 오른쪽 아랫배가 참을 만하게 아파요 타이레놀 500mg을 하루 두 번 먹었어요`
  → 복통·부위 오른쪽 아랫배·경미함·`어제부터 (10월 1일)`, 복용약 `타이레놀 500mg 하루 2회`
- `어제부터 오른쪽 아랫배가 아프고 왼쪽 팔이 저려요` → 확인 화면 부위 칸에 `오른쪽 아랫배`, `왼쪽 팔`
### 실제 기기 확인 (기존 문서 기준, 이번에 다시 확인하지 않음)

- PC 브라우저: 실제 목소리 한국어 변환과 의료 테스트 문장 5개 인식을 확인했다. (PROJECT_STRUCTURE.md)
- iPhone Safari: 로컬 HTTPS, 마이크 녹음, 음성 변환을 확인했다. (PROJECT_STRUCTURE.md)
- 지원 증상 7개가 화면에서 구조화되는 것을 2026-09-30에 확인했다. (docs/INTAKE_EXTRACTION.md)
- Android Chrome (2026-10-03, 사용자 직접 확인): HTTP + `chrome://flags` 예외 설정으로 접속·녹음·음성 변환이 됐다.
  사용자가 고른 기능 일부를 시험했고 시험한 기능은 정상 동작했다. 모든 항목을 시험하지는 않았다(시험하지 않은 항목은 기록 없음).

## 7. 남은 작업과 주의점 (STT 재구축 트랙)

- [x] Android Chrome에서 실제 확인 (2026-10-03, 일부 기능. 위 6절)
- [ ] **UI 정리 (사용자 요청 2026-10-03)**: 화면 나누기(18번), 재디자인(19~21번), 지난 기록 달력(22번), 확인 화면 접기(25번)까지 했다.
      사용자가 휴대폰으로 보고 더 고칠 점을 주면 이어서 한다.
      기능은 유지하고 화면 흐름(입력 → 확인 → 저장 → 요약)과 배치를 깔끔하게 정리한다.
- [x] 웹 자동 테스트 1차: Vitest 5.0.3을 도입하고 순수 함수 테스트 13개를 추가했다. (2026-10-01)
- [x] `recordGroups.ts` 테스트 6개 (2026-10-03, 가짜 localStorage로, 새 패키지 없음)
- [ ] 웹 테스트 확장: `recordStorage.ts`(fake-indexeddb 필요, IndexedDB를 얇게 감싼 코드라 보류), 화면 테스트
- [x] `apps/web/package.json`의 `latest` 버전을 지금 설치된 버전으로 고정했다. (2026-10-01)
      react/react-dom/@types 19.3.0, @vitejs/plugin-react 6.1.1, typescript 7.0.2, vite 8.3.1. lockfile은 specifier만 바뀌었다.
- [x] docs/INTAKE_EXTRACTION.md의 저장 설명을 IndexedDB 저장 기준으로 고쳤다. (2026-10-01)
- 주의: `apps/web/node_modules`는 `local-cache/pnpm-store`와 연결되어 있다. pnpm 기본 store와 경로가 달라
  `pnpm add/install`에서 `ERR_PNPM_UNEXPECTED_STORE`가 나면 `--store-dir ../../local-cache/pnpm-store`를 붙인다.
- [ ] 기록은 브라우저에만 있다(백업 파일로 옮길 수는 있음). 서버나 PatientState 연동과 진단 엔진은 아직 없다.
      서버 저장은 docs/PRIVACY.md 원칙(서버에 보관하지 않음)을 바꿔야 하므로 사용자 결정이 필요하다.
- [x] 백업 내보내기/가져오기 확인 (2026-10-03, Chrome PC): 내보낸 JSON에 기록 1개와 기록 묶음(기본 정보 포함),
      기록 삭제 후 가져오기로 복구, 같은 파일을 다시 가져오면 `이미 있는 기록 1개는 건너뛰었습니다`. 다운로드 대신 페이지 안에서 파일 내용을 받아
      가져오기 입력칸에 넣는 방식으로 시험했다(실제 파일 선택 창은 쓰지 않음).
- [x] 기록 삭제 메시지를 `기록을 삭제했습니다`로 고치고(전에는 `선택한 테스트 기록을…`), 가져오기를 하면 지운다. 삭제하면 백업 메시지를 지운다 (2026-10-03, Chrome 확인)
- [ ] 증상 추출 남은 한계 (2026-10-03 갱신)
      - 규칙 기반이라 65개 목록에 없는 증상은 확인 요청으로만 보여준다(겨드랑이·엉덩이 통증, 옆구리 결림, 잇몸 출혈, 멍 등).
      - (2026-10-03 해결) 띄어쓰기 없는 문장(`가슴이답답하고아파요`). 단 정해 둔 몸 부위 단어에만 공백을 넣는다.
      - 과거력은 정해진 질환 이름이나 `OO 진단을 받았어요` 형태만 인식한다.
      - (2026-10-02 해결) 주어를 이어받은 증상도 앞 절의 시작 시점을 이어받는다.
      - 시작 날짜는 하루로 정해지는 말만 바꾼다(`2주 전`, `엊그제`는 글자로만). 약 용량·시점은 약 이름 바로 뒤 30자 안만 본다.
      - 위험 증상 기준은 일반적인 응급 신호를 참고해 정한 것으로, 의료진 검토를 받지 않았다.
      - 해결함: 주어 없이 이어지는 뒷부분(`가슴이 답답하고 아파요` → 가슴 답답함 + 흉통), 경고 증상 17개 추가,
        `위염으로 진단받았어요` 같은 목록 밖 병명 기록
- 진료 중/후 기능은 **보류**한다(사용자 결정, 2026-10-02). 구현 방식을 팀원과 아직 합의하지 않았고, 진료 전 단계를 먼저 완성한다.
  2026-10-02에 DDXPlus 기반 설계 초안(`apps/api/app/knowledge/catalog.py`)을 만들다가 중단하고 삭제했다.
- [x] `StarletteDeprecationWarning` 검토(2026-10-02): FastAPI TestClient가 내는 외부 라이브러리 경고이고 결과에 영향이 없다.
      없애려면 새 패키지 `httpx2`가 필요해 그대로 둔다. starlette가 httpx 지원을 끊으면 바꾼다.
- [x] `feature/stt-rebuild`를 main에 병합했다 (2026-10-02, fast-forward). 팀원 코드 통합 방식은 아직 정하지 않았다.

## 8. 팀원 코드베이스 통합 트랙

상세: TEAM_PROGRESS_REVIEW_2026-09-29.md

- **팀원 진행 상황은 https://github.com/KYU-SW/Medmap/tree/main/MedMap 에서 확인한다** (사용자 지정, 2026-10-02).
  코드는 `MedMap/code/`, 주차별 기록은 `MedMap/week*/`, 전체 설명은 저장소 최상위 `README.md`다.
  팀원 상황을 확인할 때는 이 폴더의 최신 커밋과 README를 다시 읽고, 이 문서의 이전 기록보다 그쪽을 우선한다.

- 팀원은 별도 코드베이스에서 DDXPlus 진단 엔진, Information Gain 기반 다음 질문, PatientState와 세션 API,
  한국어 mapper, Whisper STT, Clinical Summary, HTTPS 데모 등을 구현했다고 보고했다.
  팀원 보고의 마지막 master 병합 커밋은 `ebc854f`다.
- **통합은 보류한다(사용자 결정, 2026-10-02).** 사용자가 통합하자고 하기 전에는 통합 작업을 하거나 먼저 제안하지 않는다.
  사용자가 통합을 시작하면 아래 쟁점을 보여주고 어떻게 할지 먼저 물어본다.
- 2026-10-02 팀원 공개 저장소(`KYU-SW/Medmap`)의 README·PRODUCT.md를 읽고 확인한 내용이다. 코드는 실행하지 않았으므로
  구현 내용과 테스트 수치(백엔드 404, 프런트 293)는 **README 기준**으로만 다룬다.
- 통합 전에 맞춰야 할 쟁점:
  1. 저장 원칙: 팀원은 "환자 정보 영구 저장 금지"로 타임라인·회복 추적을 보류했고, 우리는 브라우저 IndexedDB에 저장해
     타임라인·발생 기간·백업을 만들었다.
  2. 증상 정리: 팀원 매퍼는 DDXPlus 질문 코드로 출력(정밀도 0.963, 재현율 0.61)해 진단 엔진에 바로 들어가고,
     우리 규칙은 한국어 증상 이름(50개)과 불확실·다른 사람·위험 증상을 출력한다. 합치려면 연결 층이 필요하다.
  3. STT: 양쪽 모두 Whisper 기반이다. 팀원 쪽은 실시간 스트리밍·VAD가 더 갖춰져 있다고 보고했다. 성능은 직접 비교해야 한다.
  4. 제품 방향: 팀원은 의사용 진단 안전망(환자 쪽은 정보 공급), 우리는 진료 전 환자 기록 중심이다.
  5. 위험 증상 안내: 팀원 PRODUCT.md는 응급도 판정을 범위 밖으로 둔다. 우리는 흉통·객혈 등에서 119·응급실 안내를 띄운다(진단 아님).
- **진료 전 기능 비교 (2026-10-03, 팀원 문서 기준·코드 미실행, 사용자가 팀원과 상의 예정 — 물어보면 다시 설명한다)**
  - 같은 점: 텍스트·음성 입력 → 한국어 규칙 정리 → 환자 확인. 환자 화면에 진단 후보·확률 없음. 이름·연락처·음성·원문 저장 안 함.
  - 정리 결과: 팀원은 진단 엔진 입력용 DDXPlus 질문 코드(initial 1 + 추가 정확히 3, 매퍼 v1.2 정밀도 0.963·재현율 0.61, 41개),
    우리는 의사가 읽는 기록(증상 50개, 부위·시작 날짜·정도·횟수, 불확실·다른 사람 구분, 과거력·약·기본 정보).
  - 추가 질문: 팀원은 엔진이 고르는 예/아니오 질문 최대 3개(처음엔 열→통증→숨참 고정 순서), 우리는 빠진 정보(언제부터·어디·얼마나·몇 번).
  - 저장: 팀원은 **영구 저장 없음** — 진행 중 sessionStorage(탭 닫으면 사라짐), 인계는 서버 메모리 15분·1회용 8자리 번호,
    요약도 저장 안 함 → 이전 기록·타임라인을 볼 수 없다. 나중 계획(설계 S2, 보류): 병원 내부 서버(SQLite)에 Episode 단위 저장 +
    의사 로그인·보존 기간·삭제·감사·암호화. 우리는 환자 휴대폰 IndexedDB에 저장하고 백업 파일로 옮긴다.
    → 팀원은 "병원이 보관하고 의사가 본다", 우리는 "환자가 기기에 들고 다닌다". 둘을 합치는 안(기기에 쌓고 진료 때만 넘김)도 가능.
  - 인계: 팀원은 8자리 번호 → 의사 화면, 우리는 PDF·QR·요약 복사.
  - 판단: 각자 구현하기로 한 상태에서는 그대로 가도 된다(대부분 보완 관계). 통합 때 맞출 것 = 위 쟁점 1(저장), 5(위험 증상 안내),
    2(우리 증상 → DDXPlus 코드 연결 표).
- 다음 할 일(검토 문서 Phase 0~1, 통합을 시작할 때):
  - [x] 팀원 저장소 확인 (2026-10-02): https://github.com/KYU-SW/Medmap (공개, main 1개, 커밋 2개 `4f69e8b`·`87a0cf8`).
        원본 개발 저장소 이력(`08cbbd3`, `fb2d7f0`)은 들어 있지 않다. 코드는 `MedMap/code/`. 테스트 수치(백엔드 404, 프런트 293)는 README 기준이며 직접 실행하지 않았다.
  - [ ] 실행 및 테스트 명령 확보
  - [ ] STEP16B/17A 재현 자료 확보
  - [ ] 공용 저장소에 통합 브랜치 생성
  - [ ] 전체 테스트를 한 번 다시 실행
- 현재 `feature/stt-rebuild`의 STT와 증상 추출은 팀원 구현과 기능이 겹친다. 통합할 때 무엇을 남길지 결정해야 한다.

## 9. DDXPlus 연구 트랙

상세: README.md, RESULTS.md, 04_partial_observation_design.md, TEAM_PROGRESS_REVIEW_2026-09-29.md

- 완료: DDXPlus 영문 v2 공식 파일 5개 다운로드와 MD5 검증(`01_download.py`), 구조 확인(`02_inspect.py`),
  전체 통계와 split 중복 감사(`03_audit.py`), 부분 관찰 실험 설계 문서
- 원본 데이터는 `data/`에 있고 Git에서 제외되어 있다.
- 미완료 (검토 문서 Phase 3, 캡스톤 연구 핵심):
  - [ ] 자연 발생 working diagnosis의 correct/incorrect 구성
  - [ ] 현재 진단의 불일치 탐지기
  - [ ] 낮은 오경보율에서의 오류 탐지율
  - [ ] DDXPlus와 독립적인 외부 의료지식 매핑
- 지금 보류한 항목: Galaxy Watch / Health Connect, 실제 의료데이터와 임상 검증, 증거함 전체 구현,
  완전한 회복 모드, 복잡한 의사용 대시보드

## 10. 개인정보 원칙

docs/PRIVACY.md 원문 기준:

- 음성은 요청을 처리하는 동안만 사용한다.
- 원본 음성을 데이터베이스, 로그, Git 저장소에 저장하지 않는다.
- 변환된 문장도 사용자가 확인하기 전에는 환자 기록에 저장하지 않는다.
- 사용자가 확인한 기록은 현재 브라우저의 IndexedDB에만 저장하고 서버로 보내 보관하지 않는다.
- 다른 브라우저나 기기에는 기록이 자동 동기화되지 않는다.
- 사용자는 저장된 개별 기록을 화면에서 삭제할 수 있다.
- 파일명, 오류 종류, 처리 시간처럼 내용이 없는 운영 정보만 기록한다.
- 개발용 테스트 음성에는 실제 환자 정보나 개인식별정보를 넣지 않는다.
- 실제 배포 전에 보관 기간, 접근 권한, 삭제 방식과 사용자 동의 문구를 별도로 검토한다.

## 11. 참고 문서

README.md · PROJECT_STRUCTURE.md · docs/STT_SPEC.md · docs/INTAKE_EXTRACTION.md · docs/PRIVACY.md ·
docs/MOBILE_TEST.md · docs/TEMPORARY_REMOTE_TEST.md · LOCAL_STORAGE_PLAN.md · FILE_MANAGEMENT.md ·
RESULTS.md · 04_partial_observation_design.md · TEAM_PROGRESS_REVIEW_2026-09-29.md

## 12. AI 작업 규칙

- 작업을 시작할 때 HANDOFF.md와 `git status`를 먼저 확인한다.
- HANDOFF.md와 실제 코드나 Git 상태가 다르면 실제 코드와 Git을 따른다.
- 작업을 마칠 때 변경한 파일, 한 일, 테스트 결과, 남은 작업을 HANDOFF.md에 갱신한다.
- 끝내지 못한 작업을 완료했다고 기록하지 않는다.
- 사용자가 요청하지 않으면 `git commit`과 `git push`를 하지 않는다.
- push를 요청받으면 **개인 저장소와 팀 저장소 두 곳 모두** 올린다(2절). 팀 저장소는 공개라 올리기 전에 민감 정보를 확인한다.
- 팀원 코드 통합은 사용자가 통합하자고 할 때만 시작하고, 그때 8절 쟁점을 먼저 물어본다.
- 프로젝트 원본, 데이터, 설정 파일을 마음대로 삭제하지 않는다.
- 정리나 삭제가 필요하면 삭제할 후보를 먼저 사용자에게 보여주고, 승인을 받은 뒤 삭제한다.
