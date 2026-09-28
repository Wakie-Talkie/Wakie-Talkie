# Development guide

[← Project overview](../README.md) · [Architecture](ARCHITECTURE.md)

**실행 단위는 iOS 앱, Django 백엔드, FastAPI TTS 서비스입니다.** 이 저장소는 소개와 문서를 제공하며, 실행 코드는 아래 세 저장소에서 가져옵니다.

이 안내는 2024년 최종 매뉴얼과 저장소 코드를 대조해 정리했습니다. 현재 환경에서 전체 앱을 다시 빌드하거나 외부 AI API를 호출해 실행 검증한 결과는 아닙니다. 아래의 환경 설정과 재현 항목을 먼저 확인하세요.

## 1. Repositories and branches

```bash
git clone https://github.com/Wakie-Talkie/Wakie-Talkie-frontend.git
git clone https://github.com/Wakie-Talkie/Wakie-Talkie-Backend.git
git clone --branch Wakie-Talkie-use-TTS --single-branch \
  https://github.com/Wakie-Talkie/Wakie-Talkie-TTS.git
```

`Wakie-Talkie-server`는 최종 매뉴얼의 실행 구성에 포함되어 있지 않습니다. TTS는 기본 브랜치 대신 **`Wakie-Talkie-use-TTS`**를 사용해야 프로젝트의 `custom_TTS.py`와 의존성 파일을 확인할 수 있습니다.

## 2. Environment

| Component | Recorded environment / prerequisite |
| :--- | :--- |
| iOS | 최종 매뉴얼 기준 Xcode 15.2, iOS 17.2 이상 대상 |
| Backend | Django 4.2.13, DRF 3.15.1, OpenAI Python SDK 1.30.1 — `requirements.txt` 기준 |
| TTS | `setup.py`의 Python 범위: `>=3.9, <3.12`; 별도 가상환경 사용 |
| TTS dependencies | PyTorch 2.3.1, FastAPI 0.111.0, Uvicorn 0.30.1 — 프로젝트 의존성 파일 기준 |
| Audio tools | MP3/WAV 변환을 위한 FFmpeg |
| External services | 사용 가능한 OpenAI API 자격 증명, XTTS 모델 로드 환경 |
| Hardware | 마이크가 있는 iOS 기기; 프로젝트 당시 GPU 서버는 EC2 g4dn |

표의 버전은 프로젝트가 기록한 기준입니다. 최신 OS·SDK·모델 API에서의 호환성을 보장하는 버전 표는 아닙니다.

## 3. Django backend

별도 터미널에서 백엔드 디렉터리를 엽니다.

```bash
cd Wakie-Talkie-Backend
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

실행 전에 다음 설정을 확인합니다.

| Location | Configuration |
| :--- | :--- |
| `wakietalkie/ai_view.py`, `wakietalkie/call_view.py` | 원본의 `OPENAI_API_KEY = "key"`를 환경 변수에서 읽도록 로컬에서 변경 |
| 같은 두 파일의 TTS 요청 | 동일 머신이면 `http://127.0.0.1:5001`, 별도 서버면 해당 TTS 서버 주소 사용 |
| `djangoProject6/settings.py` | 로컬 개발용 설정, DB, media 경로, 접근할 host 확인 |
| 데이터 초기화 | 사용자, 언어, AI 프로필과 앱이 사용하는 ID의 대응 확인 |

환경 변수는 코드에서 읽도록 변경해야 적용됩니다. 예를 들어 기존 placeholder 대입문을 `OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]`로 바꾸고 실행 환경에서 값을 전달합니다. 키를 소스나 커밋에 저장하지 않습니다.

원본 저장소의 데이터 파일 대신 개발용 DB와 샘플 데이터를 준비한 뒤 실행합니다.

```bash
python manage.py migrate
python manage.py runserver 127.0.0.1:8000
```

실제 iPhone에서 접근하려면 로컬 호스트가 아닌, 기기에서 접속 가능한 개발 머신 주소와 바인딩 설정이 필요합니다. Django 개발 서버는 개발 환경에서 사용합니다.

## 4. Custom TTS service

새 터미널에서 **Python 3.9–3.11** 중 설치된 인터프리터로 가상환경을 만듭니다. 아래는 Python 3.10을 사용하는 예입니다.

```bash
cd Wakie-Talkie-TTS
python3.10 -m venv .venv
source .venv/bin/activate
python -m pip install -r wakie-talkie-requirements.txt
python -m uvicorn custom_TTS:app --host 127.0.0.1 --port 5001
```

`custom_TTS.py`의 직접 실행 구문은 다른 모듈명(`test_voice_trans:app`)을 참조합니다. 위 명령은 실제 파일의 `app`을 명시적으로 로드합니다. 실행 전에 CUDA와 PyTorch 호환성, 모델 다운로드·사용 조건, 의존성 설치 결과를 확인하세요.

서비스는 다음 작업을 수행합니다.

- 시작 시 `tts_models/multilingual/multi-dataset/xtts_v2` 로드
- CUDA 사용 가능 여부에 따른 장치 선택
- `POST /upload-ai-voice/`: 참조 음성 저장
- `POST /tts/`: `voice`, `language`, `text`를 받아 WAV 반환

백엔드와 GPU 서비스를 다른 머신에서 실행하면 TTS 바인딩 주소·네트워크 접근과 Django 측 URL을 함께 맞춰야 합니다.

## 5. iOS client

1. Xcode에서 `Wakie-Talkie.xcodeproj`를 엽니다.
2. 프로젝트의 signing team, deployment target과 실행할 기기를 선택합니다.
3. 마이크 사용 설명과 녹음 권한, 로컬 알림 권한을 확인합니다.
4. 최종 매뉴얼의 Background Modes 설정(`Audio, AirPlay, and Picture in Picture`)을 확인합니다.
5. Swift 파일에 분산된 API 주소를 실제 백엔드 주소로 맞춥니다. 특히 `AudioFileDataUploader.swift`의 통화 시작·업로드·종료 URL을 함께 변경합니다.
6. 로컬 HTTP 개발 환경을 사용하는 경우 필요한 host에 대한 ATS 예외를 개발 설정에서 적용합니다. 배포 환경에는 HTTPS를 사용합니다.
7. 언어·AI 프로필 데이터가 준비되어 있는지 확인한 뒤 실행합니다.

`localhost`는 실제 iPhone에서는 그 iPhone을 가리킵니다. 개발 머신의 서버를 사용할 때에는 같은 네트워크에서 접근 가능한 머신 주소를 지정해야 합니다.

## 6. Reproduction notes

아래는 문서 작성 중 소스 대조로 확인한 항목입니다. 원본 코드에 수정 사항을 적용했다는 의미는 아닙니다.

| Area | Check before running |
| :--- | :--- |
| Backend dependencies | `backports.zoneinfo==0.2.1`이 고정되어 있습니다. Python 3.9 이상에서는 표준 라이브러리와 환경에 맞게 해당 의존성 적용 조건을 조정해야 할 수 있습니다. |
| Language parameter | `stt_function`에 `Language` 모델 객체가 전달되는 경로가 있습니다. STT에는 API가 요구하는 언어 코드 문자열을 전달하도록 확인해야 합니다. |
| TTS entry point | `python custom_TTS.py`의 내부 모듈명이 실제 파일명과 다르므로 위의 Uvicorn 명령 사용 |
| Audio metadata | 실제 오디오는 WAV인데 일부 multipart MIME 또는 응답 filename은 m4a/mp3로 표기되어 있습니다. 파일 포맷과 메타데이터를 일치시킬 필요가 있습니다. |
| Call-end response | 백엔드 정상 생성 응답은 `201`, 클라이언트 일부 처리에서는 `200`만 성공으로 판정합니다. 기록 생성 후 오류로 표시되는지 확인해야 합니다. |
| Initial data | UI의 언어 선택 ID와 초기 AI 샘플 ID가 연결되어 있으므로 DB 데이터와 매핑 확인 |
| Concurrent calls | 통화 상태가 전역 변수에 저장되므로 우선 단일 통화로 검증; 다중 사용자 실행 전 세션 분리 필요 |
| Alarm state | foreground/background/terminated 상태를 나눠 알림과 화면 이동을 검증 |

## 7. Manual verification sequence

실행 환경을 구성한 뒤 아래 순서로 동작을 확인할 수 있습니다. 체크 결과를 이미 통과했다는 뜻은 아닙니다.

- [ ] 언어 목록과 AI 프로필 조회
- [ ] 기본 AI 프로필로 통화 시작 → 발화 업로드 → 응답 재생
- [ ] 응답 재생 종료 후 다음 발화 녹음
- [ ] 통화 종료 후 텍스트·오디오 기록과 단어장 조회
- [ ] 참조 음성 업로드 → 커스텀 프로필 생성 → 음성 합성
- [ ] 알람 추가·수정·삭제와 반복 요일 반영
- [ ] 앱 상태별 로컬 알림 수신과 통화 화면 진입

[← Project overview](../README.md)
