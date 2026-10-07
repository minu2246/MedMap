# MedMap 작업 인계 (HANDOFF)

작성일: 2026-10-01 (최종 갱신 2026-10-07)
브랜치: `feature/stt-rebuild`
마지막 커밋: `git log -1`로 확인한다. 2026-10-07 정리하기 기다림 없애기·정밀 인식 요약 상자·자연스러운 긴 말 규칙까지 두 저장소에 올렸다.
다음 작업: 사용자 실제 녹음으로 규칙 계속 보완(4절 ⑥ 끝의 시험 기록), 정답 기준 평가용 녹음 받기. 진료 중/후와 팀원 통합은 보류.

결정 대기 (2026-10-06 갱신):
- 진료 중/후 기능: 팀원과 방식 합의 후 시작(7절, 8절). 의사에게 넘기는 방법으로 "서버 없이 QR" 안을 사용자와 상의했다(2026-10-04):
  QR 한 장은 최대 약 2,950바이트(한글 약 980자), 휴대폰 화면끼리 안정적으로 읽으려면 그 1/3~1/2. 진료 전 요약 글은 들어가지만
  기록 전체는 안 들어간다. 현실적인 안은 증상·날짜를 짧은 코드로 바꾸고 압축해 한 장에 넣고, 의사 쪽 MedMap 화면이 읽어 펼치는 것
  (서버 보관 없음). 원문까지 넘기려면 여러 장을 차례로 보여 주는 방식이 필요하다
  병원 서버 문제(2026-10-04 상의, 팀원과 정할 안): 병원마다 EMR이 다르고 내부망이 막혀 있어(망분리) 직접 연동은 캡스톤 범위 밖.
  ① 진료 전 요약은 지금처럼 글자 그대로 QR(병원 바코드 스캐너가 키보드처럼 EMR 메모 칸에 입력, 연동 불필요)
  ② 진료 중 기능은 짧은 코드로 압축한 QR + 서버 없이 기기 안에서 도는 의사용 MedMap 화면
  ③ 나중 EMR 연동 대비로 내부 데이터 구조를 HL7 FHIR(QuestionnaireResponse, Observation 등)에 맞춰 둔다.
  한계: 진료실 PC에 카메라가 없거나 개인 기기 사용이 막힌 병원이 있고, QR은 EMR 자동 저장까지는 해결하지 못한다
- 팀원 코드 통합: 사용자가 하자고 할 때 8절 쟁점부터 묻는다
- 서버 저장: 사용자 방향은 "각자 휴대폰에 저장, 서버 저장 안 함"(2026-10-04). 남는 과제는 기기 분실 대비 백업, 기기 안 STT·추출, 진료 중 의사 화면으로 전달 방식
- 위험 증상 기준: 배뇨 곤란·쌕쌕거림을 빨간 경고에 넣을지 등은 의료진 검토 필요.
  2026-10-07 사용자 의견 "심한 복통에 119 빨간 경고는 응급 분위기를 만든다" → 사용자 선택으로 두 단계로 나눔:
  빨간 경고(119)는 위험 신호 8개(흉통·객혈·토혈·혈변·기절·마비·말 어눌함·경련)만. 심한 복통·두통·호흡곤란(심함 또는 7/10점 이상)은
  파란 안내 `복통이 심하다고 하셨어요. 진료 때 의사에게 먼저 알려 주세요. 갑자기 더 심해지면 빨리 진료를 받으세요.`
  (확인 화면·진료 전 요약, 카드는 빨간 테두리 없음, 사라진 증상 목록의 "위험 증상" 표시도 위험 신호만, 요약 글에는 `심하다고 한 증상:`).
  `symptomOptions.ts` `urgentSymptoms`/`severeSymptoms`/`severeNotice`, 웹 테스트 85개 통과, 폰 설치함(화면은 사용자 확인 전)
- `recordStorage.ts` 테스트: 새 개발 패키지 `fake-indexeddb`를 넣을지
- 폰 STT(2026-10-06): 하이브리드(기기 내장 인식 + turbo 검증)를 넣었다. 실제 마이크 확인과 정답 기준 평가 후
  turbo 검증을 계속 둘지(확인 요청 빈도·놓치는 비율) 정한다. 기기 내장 인식은 Android 13+와 구글 한국어 오프라인 언어팩이
  있어야 하므로, 시연 폰 외 기종에서는 언어팩 유무를 먼저 확인한다. 비교 후 버린 Zipformer는 모델 라이선스 표기가 없었다
- 증상 정리 규칙의 남은 오인 대응: 체온 발열·용량 있는 모르는 약·`폭통`·`콧막힘`·`타이륜` 등은 처리함(4절 ⑥).
  남은 것은 추측이 큰 오인(`수정 복용`=두 정 등)이라 정답 녹음을 받은 뒤 정한다

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
- 2026-10-06 체온 발열·모르는 약 후보 규칙과 마침표 없는 문장의 시점 경계 수정(4절 27번 ⑥)은 커밋하고 두 저장소에 push했다. 변경 파일: API
  `app/services/intake_extractor.py`, `tests/test_intake_more_symptoms.py` / `HANDOFF.md`
- 2026-10-06 하이브리드 STT(4절 27번 ⑤)는 커밋하고 두 저장소에 push했다. 변경 파일: `apps/web/android/.../DeviceSttPlugin.java`(새),
  `MainActivity.java`, `AndroidManifest.xml` / 웹 `src/App.tsx`, `src/phoneStt.ts`, 새 파일 `src/transcriptCheck.ts`, `src/transcriptCheck.test.ts` /
  새 파일 `apps/api/scripts/bench_device_stt.py`, `scripts/bench_phone_stt.py` / `docs/ANDROID_APP.md`, `HANDOFF.md`
- 2026-10-06 GPU 시험 기록·속도 시도 정리·진행 막대(4절 27번 ④)는 커밋하고 두 저장소에 push했다. 변경 파일: 웹 `src/App.tsx`, `src/styles.css` /
  `docs/ANDROID_APP.md`, `HANDOFF.md`
- 2026-10-06 빠른 코어 고정·자막·자동 측정(4절 27번 ④)은 커밋하고 두 저장소에 push했다. 변경 파일: `apps/web/android/app/src/main/cpp/whisper_jni.cpp`,
  `WhisperPlugin.java` / 웹 `src/App.tsx`, `src/phoneStt.ts`, 새 파일 `src/speechAudio.ts`, `src/speechAudio.test.ts` /
  새 파일 `apps/api/scripts/bench_phone_stt.py` / `scripts/install_android_app.ps1`, `docs/ANDROID_APP.md`, `HANDOFF.md`
- 2026-10-06 폰 STT 측정과 q8_0 기본화(4절 27번 ③)는 커밋하고 두 저장소에 push했다. 변경 파일: `apps/web/android/app/build.gradle`,
  `app/src/main/cpp/whisper_jni.cpp`, `WhisperPlugin.java` / API `app/services/intake_extractor.py`(`구구`), `tests/test_intake_more_symptoms.py` /
  `scripts/install_android_app.ps1`, `docs/ANDROID_APP.md`, `docs/STT_SPEC.md`, `README.md`, `HANDOFF.md`
- 2026-10-05 폰 안 STT 준비·안드로이드 앱(4절 27번)은 커밋하고 두 저장소에 push했다(폰 설치·실행은 아직). 변경 파일: API `app/main.py`(CORS), `app/services/intake_extractor.py`(`쎅쎅`, 한글 숫자 체온, `N일 전 저녁부터`),
  `tests/test_intake_more_symptoms.py`, 새 파일 `scripts/compare_whisper_cpp.py` / 웹 `package.json`, `pnpm-lock.yaml`, `src/App.tsx`, 새 파일
  `src/phoneStt.ts`, `capacitor.config.json`, `android/` / `scripts/install_android_app.ps1`, `docs/ANDROID_APP.md`, `docs/STT_SPEC.md`, `HANDOFF.md`
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
   - 주의(2026-10-03~04): dev 서버(Vite)가 같은 파일을 연달아 고치면 두 번째 수정을 놓친 적이 두 번 있다(App.tsx, styles.css).
     화면이 코드와 다르면 해당 파일을 `touch`하고 새로고침한다. 스크립트는 작업 폴더와 상관없이 절대 경로로 실행한다
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
27. 2026-10-05 폰 안 STT 준비 (안드로이드 중심, 사용자 결정: Mac이 없어 iOS는 나중)
   - 계획: ① PC에서 폰용 whisper.cpp(압축 turbo)와 서버 faster-whisper 비교 → ② 안드로이드 앱으로 감싸고 녹음만 폰 STT로
     (증상 정리는 당분간 PC 서버) → ③ 실제 폰에서 정확도·속도 측정 → ④ 증상 정리 규칙을 TypeScript로 옮겨 서버 없이
   - ① 준비 완료: `local-cache/whisper-cpp/`(Git 제외)에 whisper.cpp Windows CPU 빌드(b5130, `bin/Release/whisper-cli.exe`)와
     `models/ggml-large-v3-turbo-q5_0.bin`(약 550MB). 비교 스크립트 `apps/api/scripts/compare_whisper_cpp.py`
     (`cd apps/api; .venv\Scripts\python.exe -m scripts.compare_whisper_cpp` → 기본으로 `local-cache/stt-samples/`의 녹음을 읽음).
     두 엔진의 글자, 처리 시간, 증상 정리 결과가 같은지를 보여 준다
   - Windows 합성 음성(TTS) 2개로 동작만 확인(정확도 판단용 아님): 글자는 거의 같음. whisper.cpp는 PC CPU 4스레드에서 7~8초 녹음에
     약 9초 → 폰은 더 느릴 수 있어 실측 필요. 서버 모델이 `쌕쌕`을 `쎅쎅`으로 적어 규칙에 `쎅쎅` 추가(API 테스트 383개 통과)
   - 남은 준비: 사용자 실제 녹음 3~5개를 `local-cache/stt-samples/`에 넣고 비교. 안드로이드 빌드용 **NDK와 CMake가 SDK에 없다**
     (Android Studio → SDK Manager → SDK Tools에서 설치). 있는 것: Android Studio, SDK(build-tools 34~36, platforms 34·36), Java
   - 사용자 실제 녹음 3개 비교(2026-10-05, 서버 쪽은 PC CPU int8로 돌림. 실제 서버는 GPU float16이라 조금 다를 수 있다):
     폰용(whisper.cpp q5 turbo)이 증상은 더 잘 잡았다. 서버는 `쿡쿡`→`구구`, `복통`→`폭통`으로 적어 복통을 2번 놓쳤고, 폰용은 3개 모두 맞음.
     대신 폰용은 숫자를 한글로 적는다(`삼십팔 도`, `이 일 전`, `두 정`). 둘 다 틀린 것: `타이레농`, `투정·수정`(두 정), `사매`(3회).
     처리 시간: 폰용 PC CPU 4스레드에서 8~10초 녹음에 약 9초(서버 엔진 CPU는 3~6초)
   - 그래서 규칙을 고침: 한글 숫자 체온(`삼십팔 점 오 도` → 38.5℃, 35~49도만), `2일 전 저녁부터`·`이틀 전 저녁부터`·`이 일 전 저녁부터`를
     시작 시점 하나로(전에는 `저녁부터`만 남음) + 날짜 계산. API 테스트 390개 통과
   - whisper.cpp에 예시 문장(`--prompt`)을 주면 결과가 크게 나빠졌다(`아랍 배`, 글자 깨짐). **prompt는 쓰지 않는다.** 스크립트에 옵션만 남김
   - 판단: 폰용 압축 turbo로 정확도 손해는 없어 보인다(표본 3개라 단정은 이르다). 남은 위험은 폰에서의 속도
   - ② 안드로이드 앱(2026-10-05, 상세: docs/ANDROID_APP.md): Capacitor 8.5.2로 웹 화면을 감쌌다(`apps/web/android/`, appId `kr.medmap.app`).
     whisper.cpp v1.9.4를 CMake가 빌드 때 받아 JNI로 연결(`app/src/main/cpp/`, `WhisperPlugin.java`). arm64만, 항상 Release(-O3).
     웹은 `src/phoneStt.ts`: 앱 안에서는 녹음 중 실시간 자막을 끄고, 녹음 종료 때 16kHz 음성을 `Whisper.transcribe`로 넘긴다.
     증상 정리는 `http://localhost:8000`(adb reverse로 PC API), API에 CORS(`http://localhost`만) 추가.
     모델은 APK에 넣지 않고 adb로 `/sdcard/Android/data/kr.medmap.app/files/`에 복사. `scripts/install_android_app.ps1`이
     웹 빌드 → cap sync → gradle → 설치 → 모델 복사 → adb reverse를 한 번에 한다
   - 확인한 것: `assembleDebug` 성공, APK 안에 `libmedmap_whisper.so`. 웹 테스트 55개, API 390개 통과.
     **아직 폰에 설치·실행하지 않았다**(폰 연결 대기). 최신 ARM 명령(dotprod·fp16)은 켜지 않았다 — 느리면 폰 기종 확인 후 켠다
   - 주의: `android/local.properties`의 `sdk.dir`은 슬래시(`C:/Users/...`)로 쓴다. 역슬래시 하나는 깨진다(Git 제외 파일)
   - ③ 실제 폰 측정(2026-10-06, 갤럭시 노트20 SM-N981N · 스냅드래곤 865 · RAM 8GB · 안드로이드 13, 상세: docs/ANDROID_APP.md "속도"):
     처음 q5_0은 녹음 4~7초에 38~62초 → 원인은 encoder(30초 분량 고정 계산 + q5_0이 ARM 고속 경로를 못 탐).
     JNI에서 `audio_ctx` = 녹음+5초(최소 15초, 더 줄이면 PC 비교에서 단어가 바뀜), ARMv8.2 dotprod·fp16 빌드,
     원본 모델을 받아 직접 변환(MSVC로 `whisper-quantize` 빌드). 결과: q4_0 5.1~5.9초, **q8_0 5.7~6.2초(기본)**.
     q4_0은 PC 비교에서 `복통`→`폭통`, q8_0은 증상 단어 모두 맞음. 앱은 q8_0 → q4_0 → q5_0 순서로 있는 모델을 쓴다(폰에 셋 다 있음).
     whisper.cpp 타이밍은 `adb logcat -s whisper:I`. 규칙에 `구구`·`꾹꾹`(쿡쿡 오인) 추가, API 테스트 391개 통과
   - 고친 것: 설치 스크립트가 모델 복사 전에 조용히 멈췄다(Windows PowerShell에서 `2>$null` + `Stop`이 stat 실패를 예외로 만듦)
   - 다음(사용자 요청 2026-10-06): **더 빠르게**와 **실시간 자막**. 녹음 중 자막은 지금 앱에서 꺼 둔 상태(폰에서 반복 변환이 느려서)
   - ④ 빠른 코어 고정 + 자막(2026-10-06, 상세: docs/ANDROID_APP.md):
     - JNI가 `/proc/cpuinfo` CPU part로 느린 코어(A55·Kryo Silver 등)를 빼고 빠른 코어 4개에 고정(앱은 cpufreq 파일을 못 읽음).
       PC에서 명령줄 whisper.cpp(NDK 빌드, `local-cache/whisper-cpp/src-v1.9.4/build-android/`)로 재 보니 고정 안 하면 10초 이상도 나옴
     - 자동 측정 도구 `apps/api/scripts/bench_phone_stt.py`: USB로 디버그 앱 웹 화면에 접속해 PC의 WAV로 `Whisper.transcribe`·`preview` 호출.
       **사용자가 폰을 누르지 않아도** 측정 가능(폰은 잠금 해제·USB 연결). 사용자 녹음은 `local-cache/stt-samples/`
     - 구절로 끊어 turbo로 미리 변환하는 방법은 시험 후 버림(짧은 구절에서 `복통`→`폭통`, 구절마다 약 5초라 짧은 녹음은 이득 없음)
     - 자막(A안, 사용자 선택): 녹음 중 1.5초마다 base 모델(`ggml-base-q8_0.bin`, 80MB, greedy, 재시도 없음, 글자 수 상한)로 대략 변환,
       화면에서는 반복 구절을 합침(`tidyCaption`). 최종 결과는 녹음 앞뒤 조용한 부분을 자르고(`trimSilence`) turbo로 전체 변환.
       자막 0.1~0.9초, 녹음 종료 후 최종 6.3~6.7초(녹음 8~10초), 최종 결과는 전과 같음. 웹 `src/speechAudio.ts`(+테스트)
     - **실제 마이크로 자막이 뜨는 것은 아직 사용자가 확인하지 않았다**(자동 측정은 녹음 파일로 흉내 낸 것)
     - 폰 GPU(Vulkan) 시험(2026-10-06): 노트20 Adreno 650에서 모델 로드 중 매번 Segmentation fault(모델·flash attention 바꿔도 같음) → 중단.
       남은 후보는 Hexagon NPU(큰 작업). 사용자에게 "지금 상태(약 6초 + 자막) 유지"를 추천함. 빌드 방법은 docs/ANDROID_APP.md
     - 새 녹음 9~13번(2026-10-06) 측정: 받아 적기 거의 정확, 17초 녹음은 12.5초. 13번에서 `테렌`(타이레놀), `페네실린`(페니실린) 오인 →
       흔한 오인을 규칙에서 받아 줄 것(아직 안 함)
     - 속도 추가 시도(2026-10-06, 사용자 요청 "진짜 구현"): medium·small 모델(증상 4/8·2/8로 탈락), greedy(7/8, 긴 녹음 반복으로 탈락),
       말이 멈추면 미리 turbo 변환(구현·측정 후 되돌림: 이득 1~2초, encoder 중 취소 불가로 다시 말하면 최대 5초 손해, 자막과 동시면 6.3→9.4초),
       NPU(Qualcomm 공식은 SD888 이상). 결론: 노트20에서 약 5~6초가 바닥. 남긴 것은 녹음 종료 후 진행 막대(`.stt-progress`)와 안내 문구
   - ⑤ 하이브리드(2026-10-06, 사용자 결정, 상세: docs/ANDROID_APP.md "기기 내장 인식 + turbo 검증"):
     외부 검토 요청에 따라 한국어 스트리밍 Zipformer(sherpa-onnx)를 PC에서 비교 → 증상 핵심어 오인으로 탈락.
     Android 기기 내 인식(`createOnDeviceSpeechRecognizer`, 노트20에 ko-KR 설치됨, 비행기 모드 확인)이 첫 자막 0.6~2.6초,
     종료 후 확정 0.02~0.10초. turbo 대비 8개 중 2개에서 복통을 놓쳐 **turbo 검증 유지**: 기기 결과를 바로 보여 주고,
     정리하기를 누를 때 turbo 결과와 증상·약·알레르기가 다르면 두 문장을 보여 주고 고르게 함.
     새 파일 `DeviceSttPlugin.java`, `src/transcriptCheck.ts`(+테스트), `scripts/bench_device_stt.py`. 웹 테스트 63개 통과.
     **실제 마이크 시험과 사람이 확인한 정답 기준 평가는 아직**(사용자가 녹음 가능할 때 짧은 말·10~20초·30~60초 녹음 필요)
   - ⑥ 사용자 테스트 전 연구(2026-10-06):
     - 규칙 보완(커밋함): `체온이 38.5도까지`처럼 체온만 말해도 37.5도 이상이면 발열(°표기 포함, 정상 체온은 기록 안 함).
       모르는 약 이름 + 알약 용량(`테렌을 500mg`, `타이륜을 두정`)은 들린 그대로 약 후보로 남김(비슷한 약으로 바꾸지 않음,
       `어제/아파서 두 알`·`물 500ml` 제외). API 테스트 401개 통과
     - 어려운 조건 시험(커밋 안 함, `local-cache/stt-research/`, Git 제외): `make_variants.py`로 녹음 27개 생성
       (긴 연속 발화 43~44초 2개, 끊어 말하기·느리게 0.85배·빠르게 1.2배·소음 15/5dB 각 5개).
       `run_phone.py`가 폰에서 기기 인식(live)과 turbo로 돌려 `results.jsonl`에 저장(메모리·배터리 온도 포함),
       `analyze.py`가 임시 기준(깨끗한 녹음의 turbo 결과, 사람 정답 아님)과 비교
       (`cd apps\api; .venv\Scripts\python.exe ..\..\local-cache\stt-research\analyze.py`).
     - 결과(27개 모두 완료, 2026-10-06): 앱 메모리 1.16~1.27GB로 안정, 배터리 29→35°C, 오류·멈춤 없음.
       기기 인식은 모든 조건에서 첫 자막 0.8~3.1초·종료 후 0.02~0.11초 유지. turbo는 6~34초(43초 녹음 34초).
       기기 인식 약점: 끊어 말하기(1.2초 쉼)에서 문장이 잘림(`38.5도`→`8.5°`, `코막 팀`), `복통`→`폭통` 반복,
       `타이레놀`→`타이륜`. 5dB 소음은 두 엔진 모두 크게 틀림(turbo는 13-noise5에서 같은 말 반복 환각).
       turbo가 놓친 것을 기기가 맞힌 경우도 있음(12-noise15 `머리가 부모`, 12-slow `코마킹`). → 두 결과 비교(검증) 유지가 맞다.
       엄격 비교(시작 시점까지 같아야 OK)라 숫자는 낮게 나옴: 같은 녹음 수 기기/turbo = 빠르게 0/3, 끊어 0/2, 소음15 0/2, 소음5 0/0, 느리게 0/1, 긴 0/0 (각 5개, 긴 2개)
     - 이 분석에서 찾은 규칙 버그 수정(커밋함): 마침표 없는 기기 인식 문장에서 다음 문장의 시점이 앞 증상에 붙음
       (`기침이 심해졌어요 오늘 아침에는` → 기침 시작 '오늘 아침'). `_onset_for_symptom` 경계에 `요 ` 추가, 테스트 추가, API 402개 통과
     - 같은 분석의 받아 적기 오인을 규칙에서 받아 줌(2026-10-06, 커밋 안 함): `폭통`→복통, `콧막힘`·`코 막힘`·`코마킹`·`코마킨`→코막힘
       (`코 막힘` 띄어쓰기는 원래 놓치던 버그), turbo의 `38.5度`→발열, `2일 전, 저녁부터`의 쉼표, 용량 없이 `타이륜을/타이레노 먹었어요`처럼
       `타이`로 시작하는 이름은 들린 그대로 약 후보(`타이밍` 등 제외). 같은 위치에 약 이름이 두 번 잡히면 용량이 사라지던 문제도 함께 막음.
       27개 녹음 분석에서 놓친 증상 41→32. API 테스트 412개 통과. 남은 것: `수정/추정 복용`(두 정), `크고 쑤시고`(쿡쿡), `어디 저녁`(어제)은 추측이 커서 안 함
     - 정리하기 기다림 없애기(2026-10-06, 사용자 선택, 커밋 안 함): 전에는 정리하기를 누르면 turbo가 끝날 때까지 기다렸다
       (43초 녹음은 최대 약 30초). 이제 기기 인식 글로 바로 확인 화면을 열고 turbo는 뒤에서 확인(`App.tsx` `checkAgainstTurbo`).
       다르면 확인 화면 위에 두 문장 고르기(`verificationBox`, 녹음 화면과 공용). 빠른 인식을 고르면 확인 화면에서 고친 내용 유지,
       정밀 인식을 고르면 다시 정리. turbo가 끝날 때까지 저장 버튼은 "정밀 인식 확인 중…"으로 막음.
       웹 빌드 성공, 웹 테스트 63개 통과, 폰에 설치함. **실제 마이크로 이 흐름은 아직 확인하지 않았다**.
       같이 고침: `pnpm test`가 안드로이드 빌드 캐시(`android/app/.cxx`) 안의 whisper.cpp 테스트까지 돌려 실패 → `vitest run --dir src`
     - 사용자 실제 마이크 시험(2026-10-07) 후 고침(커밋 안 함): turbo가 숫자를 한글로 적음(`삼십팔 점 오 도`, `오백 밀리그램`, `삼회`, `이일 전`)
       → 웹 `spokenNumbersToDigits`(`transcriptCheck.ts`, 테스트 9개)로 turbo 결과를 숫자로 바꿔 보여 줌(도는 30~45만, 회는 `했/정도` 등 앞만).
       약·알레르기 이름이 한 글자만 틀리면(4글자 이상, `페니슐린`·`헤니실린`·`타이레농`·`타이레노`) 알려진 이름으로 바꿈(`_known_spelling`, API 417개 통과).
       전의 "들린 그대로 남김" 원칙을 한 글자 차이에 한해 바꾼 것. 확인 화면에서 환자가 고칠 수 있다. 실제 시험에서 turbo 처리 11~32초(녹음 18~20초에 약 12초)
     - 실제 시험 2차(2026-10-07): 상자·숫자 변환 정상 확인(사용자). 사용자 의견 "증상을 누르는 중에 정밀 인식 결과가 위에 갑자기 뜨는 게 별로"
       → 결과 상자를 **저장 버튼 바로 위**로 옮기고(위 내용이 밀리지 않음), 문장 통째 고르기 대신 **항목별 반영/무시**
       (`factChanges`가 `factDifferences`를 대체: 증상 추가·빼기·있음/없음·시작 시점, 복용약·알레르기 추가·빼기).
       반영하면 확인 화면에 그 항목만 더하거나 바꿔서 환자가 고친 다른 내용(정도 등)은 남는다. 다 처리할 때까지 저장 버튼은
       "위 정밀 인식 결과를 먼저 확인해 주세요"이고, 누르면 상자로 이동. 녹음 화면에서는 상자를 뺐다.
       3차 시험(2026-10-07): 사용자 "UI가 더 지저분해짐"(항목마다 버튼 2개) → 사용자 선택으로 **한 줄 요약**으로 바꿈:
       "정밀 인식이 N가지를 다르게 들었어요 [모두 반영] [무시]" + 접힌 "무엇이 다른지 보기"(항목 목록·정밀 인식 문장).
       `resolveVerification`이 다른 항목만 더하고 바꿈(고친 내용 유지). 시작 시점만 다르면 `기침 시작: 저녁부터 → 어제 저녁부터`.
       3차 시험 로그: 두 인식 모두 `숨이 약간 작은/찬한`(차는)으로 들어 호흡곤란을 놓침(규칙으로 못 받음, 남은 오인).
       4차 시험: 한 줄 요약이 제일 낫다(사용자). 펼친 목록을 "두통 추가" 대신 **들린 말 비교**로 바꿈(사용자 요청, 표현은 사용자 선택):
       안내 한 줄 "처음 들은 말 → 다시 확인한 말" + `"머리가 크고" → "머리가 아프고" (두통 있음, 어제 저녁부터)`.
       `describeChanges`(`transcriptCheck.ts`)가 두 문장을 단어 단위로 맞춰(LCS) 증상 근거(source_text)·약 이름·시작 시점 주변의
       다른 단어를 찾는다. 못 찾으면 `(못 들음)`/`(없음)`. 띄어쓰기만 다른 단어(`한 번에`/`한번에`)는 넓히지 않음.
       5차 시험(자연스러운 34초 녹음 2개, turbo 31초): 사용자 요청으로 작은 "정밀 인식 문장 보기"(접힘) 추가. 다른 점이 없거나
       처리한 뒤에도 "정밀 인식으로 확인했어요 · 정밀 인식 문장 보기"로 남는다. 통증 점수 숫자(`십점 만점에 칠점`→`10점 만점에 7점`).
       웹 테스트 77개 통과, 폰 설치함(폰 화면 확인 전).
       **같은 시험에서 찾은 규칙 오류(두 엔진 공통, 아직 안 고침)**: `배가 아프거나 토한 적은 없고`→복통 있음, `열이나 기침은 없어요`→발열 있음,
       `체온을 재보니까 38.2도였어요`→발열 못 잡음, `머리도 아팠지만 지금은 괜찮아졌어요`→두통 있음, `설사 네 번`→1회,
       통증 점수 `7점`이 복통이 아닌 설사·두통에 붙음, `페니실린을 먹고 두드러기가 생긴 적`→발진(현재)·복용약으로 잡힘(알레르기여야 함),
       `사흘 전부터 목이 따끔거리고 기침이 나기 시작`→기침 시작 시점 없음
       → 규칙 수정(2026-10-07, 커밋 안 함, 새 테스트 `tests/test_intake_natural_speech.py`, API 424개 통과, 연구 분석 놓침 32 그대로):
       `거나/이나/과/와/랑/하고`로 묶인 증상 뒤 같은 절에 `없/않`이 오면 모두 없음(`_is_negated_in_list`, `고`로 이어지면 그대로 있음),
       `아팠지만 지금은 괜찮아졌어요`→없음, `체온을 (재보니까) 38.2도`→발열, `열과`도 열로 읽음,
       `X을 먹고 두드러기가 생긴 적이 있어요`→알레르기 X(현재 증상·복용약에서 뺌, `DRUG_REACTION_PATTERN`),
       정도·횟수는 같은 증상의 **뒤 언급**(`배가 아픈 정도는 7점`)에도 붙고 횟수는 횟수 세는 증상(구토·설사)끼리만 다툼,
       `심해졌`은 정도가 아니라 추세, 횟수·추세는 `요 ` 문장 경계를 넘지 않음(정도는 다음 문장 점수 때문에 넘음).
       이어서 처리(API 425개 통과, 연구 분석 놓침 32 그대로): `A...고 (주어) B`로 이어진 두 증상은 B에 시작 시점이 없으면 A의 것을 씀
       (`사흘 전부터 목이 따끔거리고 기침이`→둘 다 3일 전부터, `배가 아프고 속이 메스꺼웠어요`→메스꺼움도), `토한 적은 없고`→구토 없음,
       `숨 쉬는 것도 괜찮아요`→호흡곤란 없음(`괜찮아졌`은 좋아지는 중이라 있음 유지), `처음에는 7점이었는데 지금은 4점`→`7/10점 → 4/10점`
       6차 시험(비염 31초): 사용자 "빠른 인식도 이제 잘 잡는다". **turbo가 31초 녹음 중간(약 25~30초)을 통째로 빠뜨림**
       (`꽃가루 알레르기 … 혈압약 … 배가 아프거나`) → 30초 창 경계로 추정. 기기 인식은 그 부분을 맞게 들음.
       그래서 확인 상자에서 **turbo가 못 들은 것은 빼자고 하지 않음**(증상 빼기 없음, 약·알레르기 빼기는 다른 이름으로 바뀐 경우만).
       규칙: `채치기`·`코도 박혀서`, 추세도 뒤 언급에 붙음(`콧물은 줄었지만 코막힘은 아직`), `6점 아니 4점`→4점,
       `콧물이 나고 재채기가`처럼 띄어 쓴 `~고`도 시작 시점 공유(`요 ` 문장 끝은 넘지 않음). API 427개, 웹 79개 통과, 앱 재설치.
       긴 녹음 나눠 변환 시험(2026-10-07, 구현 안 함): 43~44초 녹음 2개를 15~25초 사이 가장 조용한 곳에서 잘라 폰 turbo로 비교
       (스크립트는 scratchpad, 저장소에 없음). 지금 설정(창 = 녹음+5초)으로 나누면 **정확도가 나빠짐**(같은 문장 반복 환각,
       `열이 308까지`, 알레르기 깨짐, 2회 모두). 임시로 창을 30초 전체로 하니 나눠도 통째와 같은 정확도였지만 조각마다 약 17초라
       녹음 중 미리 변환해도 **종료 후 대기가 거의 줄지 않음**(44초 녹음 추정 33초 vs 통째 35초). 실험 빌드는 되돌림(JNI 그대로, 6.9초 녹음 6.6초 확인).
       결론: 노트20에서는 나눠 변환으로 빨라지지 않는다. 30초 넘는 녹음의 빠뜨림은 "turbo가 못 들은 것은 빼지 않음"으로 피해를 막아 둔 상태
       기기 인식 단어 힌트 시험(2026-10-07, 구현 안 함): `EXTRA_BIASING_STRINGS`(Android 13+)에 의료 단어 36개(복통·재채기·타이레놀·페니실린·사흘 전부터 등)를
       넣고 녹음 27개를 다시 돌림 → 오류 47→48로 **효과 없음**(`폭통`, `4월 초부터`, `타이륜` 그대로, 차이는 실행마다의 흔들림).
       노트20의 구글 오프라인 한국어 인식은 이 힌트를 반영하지 않는 것으로 보임. 코드 되돌리고 원래 앱 재설치(APK에 힌트 없음 확인).
       측정 동안만 "USB 연결 시 화면 꺼지지 않음"을 켰다가 다시 끔(값 0)
     - 문구 녹음 평가(2026-10-07, 정답 있는 첫 평가): 문구 `local-cache/stt-samples/scripts_2026-10-07.txt`(21~27번)를 사용자가 노트20 녹음 앱으로
       읽어 `새로운 녹음 14~20.m4a`로 올림(14=21 … 20=27, 8~23초). PyAV로 16kHz WAV 변환(`stt-samples/wav/`) 후 폰에서 기기 인식(실시간 흘려 넣기)과
       turbo 실행, 결과 `stt-samples/results_2026-10-07.jsonl`(Git 제외). 측정은 `scripts/run_recordings.py`(폰, 화면 켜 둠),
       채점은 `scripts/eval_recordings.py`(문구 파일과 비교, 녹음↔문구 자동 짝짓기, turbo 숫자는 웹 `spokenNumbersToDigits`를 Node로 실행).
       이 방식은 엔진·설정·규칙은 앱과 같고, 녹음 경로(녹음 앱 압축)·화면 동작·동시 실행 부하만 다르다(사용자에게 설명함).
       문구 자체에서 찾은 규칙 버그 고침: 마침표 뒤 `아픈 정도는 7점`(turbo가 마침표를 넣음), `6점, 아니 7점`, `배는 처음엔 많이 아팠는데`,
       `설사한 적은 없어요`, `고혈압이랑 당뇨가`, `39도까지 … 지금은 37.8도`→`39℃ → 37.8℃`, **소수점에서 문장을 자르던 버그**(`37.8`의 `.`),
       `콧가루`→꽃가루, `채채기`, `따끈거리고`, 용량 `밀리그램`→`mg`, 웹 숫자 변환 `이백밀리그림`. API 435개, 웹 80개 통과, 앱 재설치.
       결과(정답 문구의 규칙 결과와 비교, 7개 녹음의 틀린 항목 수): 기기 인식 5, turbo 3. 남은 것은 모두 받아 적기 오인
       (기기: `가래는`→`가리는`, `꽃가루 알레르기`→`꽃가루를 내리기`, `설사한`→`설수한`, `나아졌어요`→`나와줬어요`; turbo: `메스꺼워요`→`맸을 거예요`,
       `잤어요`→`샀어요`). 확인 상자의 모두 반영까지 하면 남는 것은 약 2개(`가래 없음` 둘 다 놓침, 복통 정도·추세는 비교 대상이 아님).
       기기 인식은 녹음 종료 후 0.02~0.13초, turbo는 8~23초 녹음에 6~16초
     - turbo 단어 목록(prompt) 시험(2026-10-07, 구현 안 함): JNI에 `initial_prompt`(UTF-8)를 임시로 넣고 녹음 15개를 폰에서 목록 없이/있이 비교.
       목록: 증상 15개·약/알레르기 6개·`38.5도, 500mg`(쉼표로 구분). 좋아진 것: `가리는`→`가래는`(19번), `채채기`→`재채기`, `타이레농`→`타이레놀`.
       나빠진 것: 13번에서 `테렌을`이 깨진 글자(`테레�`)가 되고 `페니실린, 알레르기`처럼 쉼표가 끼어 **알레르기가 사라지고 페니실린이 복용약으로** 잡힘
       (목록의 쉼표 문체를 따라 함). 시간 약 +4%. 정답 녹음 7개 틀린 항목 3→2지만 새 오류 종류가 생겨 기준 미달 → JNI·Java 되돌리고 원래 앱 재설치.
       참고: PC whisper-cli(Windows)는 한글 인자를 UTF-8로 받지 못해, 예전 "예시 문장 prompt가 결과를 망침"도 글자 깨짐 탓이었을 수 있다.
       규칙 쪽 `페니실린, 알레르기` 쉼표 허용은 해 둠(알레르기로 읽고 복용약에서 뺌, API 436개 통과).
       쉼표 없는 목록(띄어쓰기만)으로 재시험(같은 날): 좋아진 것 `가리는`→`가래는`, `채채기`→`재채기`. 알레르기 문제는 사라짐.
       그러나 13번 `테렌을`은 여전히 깨진 글자(`테레�`)라 복용약이 빠지고, 목록 문체를 따라 마침표가 사라짐. 시간 +3%.
       정답 녹음 틀린 항목 3→2, 정답 없는 녹음에서 복용약 1개 손실 → 이득과 손실이 비슷해 **되돌림**(원래 앱 재설치).
       판단: 단어 목록은 효과가 있지만 깨진 글자 위험이 있다. 정답 녹음이 20~30개로 늘면 다시 비교할 만하다
     - 사용자 요청(2026-10-07): "USB 연결 시 화면 꺼지지 않음"은 **사용자가 끄라고 할 때까지 켜 둔다**(현재 값 2)
     - 녹음 추가 전 보완(2026-10-07, 커밋 안 함):
       ① 오늘 생긴 `7/10점 → 4/10점` 같은 정도 값을 응급 판정·변화 배지·최고 정도가 읽지 못하던 문제 → 화살표 뒤(지금 값)를 읽음
          (`symptomOptions.ts`, `timeline.ts`, `symptomEpisodes.ts`, 테스트 추가). `4/10점 → 8/10점` 복통이 응급 안내에서 빠질 뻔했다.
       ② turbo가 내는 깨진 글자(`테레�`, `타이래�을`)를 `WhisperPlugin.java`에서 지움 → 남은 글자(`테레 500mg`)가 복용약 후보로 읽힘.
       ③ 확인 상자가 정도·횟수·추세도 비교(`detailChanges`): turbo가 들은 값이 다르면 `복통 정도: 경미함 → 심함, 추세: 없음 → 좋아지는 중`.
          turbo가 못 들은 값은 우리 것을 유지. 모두 반영하면 이 값들도 바뀜.
       ④ 채점 `eval_recordings.py`에 **merged**(모두 반영 후 확인 화면) 점수 추가(앱 규칙을 Python으로 따라 함).
       오늘 녹음 7개 틀린 항목: 기기 5, turbo 3, **합친 결과 1**(`가래 없음`, 두 인식 모두 `가리는`). 웹 83개, API 436개 통과, 앱 재설치
     - 폰 개발자 옵션 "USB 연결 시 화면 꺼지지 않음": 2026-10-07 한때 껐다가, 같은 날 사용자 요청으로 다시 켜 둠(사용자가 끄라고 할 때까지, 끄기: `adb shell settings put global stay_on_while_plugged_in 0`)
   - 배포 때는 USB·adb가 필요 없다(사용자와 상의, 2026-10-05): Play 스토어(또는 APK 공유)로 설치, 모델은 첫 실행 때 내려받기
     (Play Asset Delivery나 다운로드 서버, 와이파이 안내·진행률·이어받기), 증상 정리 규칙을 TypeScript로 옮겨 PC 없이.
     Play 등록에는 개발자 계정(1회 25달러), 앱 서명, 개인정보 처리방침, 건강 앱 정책 신고, 새 개인 계정은 출시 전 비공개 테스트(테스터·약 2주)가 필요.
     순서: 먼저 폰 속도가 쓸 만한지 확인 → 느리면 모델·최적화부터 다시 정한다

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
      - **증상 종류 추가는 나중 과제(사용자 합의, 2026-10-07)**: 먼저 정답 녹음으로 기존 65종 정확도를 올린다. 늘리면 잘못 잡을 위험,
        TypeScript 이식 양, 팀원 DDXPlus 연결표 작업이 함께 커진다. 예외: 실제 녹음·시연 시나리오에서 자주 놓치는 증상은 그때 추가
        (후보는 docs/RECORDING_BACKLOG.md 3절). 순서: 정답 녹음 늘리기 → 기존 정확도 보완 → 자주 놓친 증상만 추가 →
        팀원 통합 방식에 맞춰 목록 정리 → 폰 단독용 TypeScript 이식
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
- **팀원 진행 확인 (2026-10-05, 읽기만 함)**: 팀원 커밋은 2026-10-02의 2개(`4f69e8b`, `87a0cf8`)뿐이고 그 뒤 새 커밋·브랜치·PR이 없다
  (이후 커밋은 모두 우리 동기화). 아래는 `MedMap/week02_2026-09-28_to_10-04/README.md` 기준(코드 미실행).
  - 방향: 실기기 시험 후 최종 제품을 **의사용 진단 안전망**으로 재정의, 환자 쪽은 intake/companion(정보 공급)
  - 9/30 의사 화면 기초(진단 먼저 입력 → 확률 없는 감별 후보 5~8 → 의사 질문 ≤3), 실시간 오프라인 STT(말하는 중 자막 p95 465ms)
  - 10/1 기기 간 인계 번호, 기기별 잠금, 통합 QA, 환자 화면 진단 후보 숨김, 오프라인 배포 검사
  - 10/2 의사 화면에 확인 소견 전체 표시, STEP13B 실패 분석, 다음 핵심 실험 EXP-SN-001(흉통 감별) 사전등록 rev2
  - 미완(팀원 보고): 진단 검증·미설명 소견·놓친 대안 찾기 미구현, EXP-SN-001 실행 0, 진료지침 출처의 AI 사용 허가 문제로
    질환 지식 작성 막힘, 실제 환자·의사·임상의 검수 없음
  - 우리와의 관계: 대체로 역할이 나뉨(팀원 의사 화면·연구 / 우리 진료 전 기록·폰 안 STT). 겹침: 실시간 STT(팀원 PC 스트리밍 vs
    우리 폰 whisper.cpp), 인계(팀원 번호·서버 메모리 15분 vs 우리 QR·PDF). 최신 진행은 사용자가 팀원에게 직접 확인하기로 함

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
- 진료 전 기능과의 연결점(2026-10-04 정리): 지금은 데이터를 주고받지 않는다. ① 한국어 증상 65개 → DDXPlus 소견 코드 연결표가
  있어야 진료 중 기능이 진료 전 기록을 쓸 수 있다. ② 연구의 "다음에 물어볼 정보(NBInfo)"가 되면 진료 전 추가 질문을 진단을 가르는
  질문으로 바꿀 수 있다. 진료 중 기능 방식을 정할 때 이 트랙 계획도 같이 정한다
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

- 2026-10-06 Codex 자문(사용자가 Claude에 전달할 제안 요청): 현재 코드는 단일 worker와 자막 완료 대기로 자막/최종 추론을 순차 실행한다.
  이전 동시 실행 설명은 철회한다. 5~6초는 현 구성의 관측 결과이며 모든 온디바이스 STT의 한계가 아니다.
  미시험 후보: sherpa-onnx 한국어 streaming Zipformer `sherpa-onnx-streaming-zipformer-korean-2024-06-16`(공식 int8 제공).
  메인 앱 교체 전에 같은 합성 테스트 음성으로 의료 핵심 정보 정확도와 노트20 실시간 입력 지연을 비교할 것.
  turbo와 증상 정리 결과의 일치율(기존 small 2/8 등)을 사람 정답 대비 STT 정확도와 혼동하지 않는다.
  새 엔진이 정확도 기준을 통과하면 최종 결과에도 사용, 자막만 통과하면 자막 대체만 검토(이 경우 turbo 최종 대기 시간은 남음).
  참고: https://k2-fsa.github.io/sherpa/onnx/pretrained_models/online-transducer/zipformer-transducer-models.html#sherpa-onnx-streaming-zipformer-korean-2024-06-16-korean
  이번 작업은 조사·제안과 이 기록 추가만 수행. 코드 변경·모델 설치·실기기 벤치마크·테스트·커밋·push 없음. 주 개발은 Claude에서 진행 중.

- **정답 녹음 문구 재료: docs/RECORDING_BACKLOG.md** (2026-10-07 사용자 요청). 사용자는 실제 환자가 아니라 증상·연구용 표현을 고르기 어렵다.
  시험에서 드러난 오인·규칙 약점·아직 녹음 안 한 증상을 여기에 쌓고, 사용자가 "녹음 문구 달라"고 하면 여기서 골라 문구를 만든다.
  시험이 끝날 때마다 이 문서(상태, 녹음 기록)를 갱신한다.

README.md · PROJECT_STRUCTURE.md · docs/STT_SPEC.md · docs/INTAKE_EXTRACTION.md · docs/PRIVACY.md · docs/RECORDING_BACKLOG.md ·
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
