# MedMap 로컬 저장공간 관리

프로젝트 종료 후 한 번에 정리할 수 있도록 프로젝트 전용 파일을 가능한 한 이 저장소 아래에 둔다.
이 문서는 삭제 명령이 아니라 위치 기록이다. 실제 삭제 전에는 GitHub 업로드와 미커밋 파일을 다시 확인한다.

## 프로젝트 전용 저장 위치

|항목|경로|GitHub 보관|복원 방법|
|---|---|---|---|
|프로젝트 코드·문서|MedMap 저장소 루트|예|GitHub clone|
|DDXPlus 원본|`data/`|아니오|`01_download.py`|
|Python 환경|`apps/api/.venv/`|아니오|`scripts/setup_api.ps1`|
|Frontend 패키지|`apps/web/node_modules/`|아니오|추후 lockfile 기준 설치|
|Whisper 모델|`local-cache/huggingface/`|아니오|첫 모델 실행 시 재다운로드|
|모바일 시험용 인증서|`local-cache/mobile-https/`|아니오|`scripts/setup_mobile_https.ps1`|
|임시 외부 시험 도구|`local-cache/tools/cloudflared.exe`|아니오|필요할 때 공식 배포본 재다운로드|
|pip 캐시|`local-cache/pip/`|아니오|패키지 설치 시 재생성|
|pnpm 캐시|`local-cache/pnpm-store/`|아니오|lockfile 기준 재생성|
|실험 결과|`runs/`|선별|재현 코드 또는 최종 결과 보존|
|모델 체크포인트|`checkpoints/`|아니오|필요 시 별도 원격 보관|
|whisper.cpp 소스·빌드·폰 모델|`local-cache/whisper-cpp/`|아니오|docs/ANDROID_APP.md 절차로 다시 받기·빌드|
|STT 비교용 모델(sherpa)|`local-cache/sherpa/`|아니오|비교 끝남, 다시 필요 없음|
|Vulkan·SPIRV 헤더(폰 GPU 시험)|`local-cache/vulkan-headers/`, `spirv-headers*/`|아니오|공식 저장소에서 다시 받기|
|EXAONE 모델(학습 문장 생성)|`local-cache/ollama/`|아니오|`ollama pull exaone3.5:7.8b`|
|학습용 Python 환경(PyTorch 2.11 CUDA, transformers)|`local-cache/train-venv/`(4.5GB)|아니오|venv 만들고 `pip install torch --index-url https://download.pytorch.org/whl/cu128`, `numpy transformers`|
|학습한 증상 모델|`local-cache/intake-model/`|아니오|`apps/web/train/train.py`로 다시 학습|
|생성한 학습 문장|`local-cache/intake-train/`|아니오|`apps/web/train/generate.ts`로 다시 생성(같은 문장은 안 나옴)|
|녹음 시험 파일·결과|`local-cache/stt-samples/`, `stt-research/`|아니오|실제 목소리라 보존 여부를 사용자에게 확인|
|팀 저장소 작업 복사본|`local-cache/team-repo/`|예(팀 저장소)|`scripts/sync_team_repo.ps1`이 다시 받음|
|안드로이드 빌드 결과|`apps/web/android/app/build/`, `.gradle/`|아니오|`scripts/install_android_app.ps1`|

## 프로젝트 밖에서 사용하는 항목

- Git for Windows와 NVIDIA 드라이버는 공용 프로그램이므로 MedMap 종료 시 삭제하지 않는다.
- **MedMap 때문에 프로젝트 밖에 설치한 것**(종료 시 지울 후보, 2026-10-21 확인):
  - Ollama 프로그램(약 2.8GB)과 `~/.ollama`, `%LOCALAPPDATA%\Ollama`, 시작프로그램 `Ollama.lnk` → `scripts/remove_ollama.ps1 -Execute`
    (프로젝트 안 `local-cache/ollama`, `local-cache/intake-train`도 함께 지움)
  - Android SDK의 NDK `30.0.16248370`(2.1GB)과 CMake `4.1.2`(56MB): 2026-10-05 폰 앱 빌드용으로 추가.
    Android Studio → SDK Manager → SDK Tools에서 체크 해제로 지운다
  - `~/.gradle`(약 1.1GB): 2026-10-06부터 생긴 Gradle 캐시로 보인다. 다른 안드로이드 작업이 없으면 지워도 다음 빌드 때 다시 받는다
  - 노트20의 MedMap 앱과 앱 안 모델: 폰에서 앱 삭제(기록도 함께 지워지므로 필요하면 먼저 백업 내보내기)
- **MedMap 전부터 있던 것**(지우지 않음): Android Studio, Android SDK 나머지(2025-11 설치, 약 10GB), Visual Studio 2022 C++ 도구, Java.
- Codex에 포함된 Python은 다른 작업도 사용하는 공용 런타임이므로 삭제하지 않는다.
- 사용자에게 받은 원본 문서는 OneDrive/카카오톡 받은 파일에 있으며 프로젝트 파일과 별도로 관리된다.
- GitHub 인증정보는 Windows 자격 증명 관리자에 있다. 프로젝트 폴더 삭제와 별개다.

## 용량 확인

일반 PowerShell에서 다음을 실행한다.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\storage_report.ps1
```

프로젝트가 끝나면 다음 순서로 정리한다.

1. `git status`로 미커밋 파일 확인
2. 필요한 코드·문서·최종 결과를 GitHub에 push
3. 원격 커밋과 복원 방법 기록
4. `storage_report.ps1`로 실제 삭제 대상과 용량 확인
5. 사용자가 삭제 범위를 승인한 뒤 프로젝트 전용 경로만 삭제
   - 프로젝트 안: `scripts/cleanup-project.ps1`(그냥 실행하면 목록만, `-Execute`면 삭제. `local-cache` 전체, 가상환경, `data` 등)
   - 프로젝트 밖: 위 "MedMap 때문에 프로젝트 밖에 설치한 것" 목록

## 2026-10-21 현재 용량

|항목|대략적인 용량|
|---|---:|
|`local-cache/whisper-cpp`|4.6 GB|
|`local-cache/ollama`(EXAONE)|4.5 GB|
|Python 환경 `apps/api/.venv`|2.3 GB|
|`local-cache/huggingface`(Whisper)|1.6 GB|
|`local-cache/pip`|1.4 GB|
|`local-cache/sherpa`|0.55 GB|
|DDXPlus `data/`|0.17 GB|
|그 밖의 `local-cache`, 안드로이드 빌드 결과|약 0.35 GB|
|**프로젝트 안 합계**|**약 15.5 GB**|
|프로젝트 밖(Ollama 프로그램, NDK·CMake, `~/.gradle`)|약 6 GB|

현재는 어떤 파일도 삭제하지 않는다.

## 2026-09-29 현재 용량

|항목|대략적인 용량|
|---|---:|
|저장소 전체|5.45 GB|
|DDXPlus 원본|0.17 GB|
|Python 환경|2.25 GB|
|웹 패키지|0.07 GB|
|Whisper·패키지 캐시|2.95 GB|

용량은 실행할 때마다 달라질 수 있으므로 삭제 직전에 `storage_report.ps1`로 다시 확인한다.
현재 패키지 캐시에는 GPU 라이브러리 설치 파일이 포함되어 있다. 설치 완료 후에는
`local-cache/pip/`를 삭제해도 실행에 영향이 없으며, 필요하면 다시 받을 수 있다.
