# Project notes & credits

[← Project overview](../README.md)

## Project

| Item | Details |
| :--- | :--- |
| Name | Wakie-Talkie |
| Context | 중앙대학교 2024-1 캡스톤디자인 · Team 4 |
| Members | 강지인 · 이은화 · 최동우 |
| Platform | iOS |
| Deliverables | iOS 앱, Django API, 커스텀 TTS 서비스, 최종 보고서·매뉴얼, 공학학술제 시연 |

## Design evolution

**초기 언어 교환 아이디어를 AI와의 음성 대화에 집중하는 서비스로 구체화했습니다.** 제안서에는 사용자 간 VoIP 연결과 언어 교환이 포함되어 있었고, 최종 보고서에서는 AI 프로필 선택·커스텀 음성·알람·복습 기능으로 범위를 조정한 이유를 설명합니다.

| Phase | Outcome |
| :--- | :--- |
| Initial proposal | 알람과 언어 교환을 결합하는 아이디어, 사용자 간 통화 및 AI 활용 검토 |
| Final capstone project | 알람 기반 AI 대화, 통화 기록, 자동 단어장, 참조 음성을 활용한 커스텀 프로필 구현 |
| 2024 engineering fair | 실제 기기에서 알람·통화·다국어·기록·단어장·커스텀 AI 기능 시연 |

README의 기능 설명은 최종 구현과 시연 자료를 기준으로 합니다. 초기 제안서의 사용자 간 VoIP 통화는 현재 기능으로 포함하지 않습니다.

## Implementation scope

- 앱에서 녹음한 발화를 서버로 보내 STT → GPT → TTS로 처리하는 음성 대화
- SwiftData와 로컬 알림을 활용한 알람 일정 관리
- 대화 오디오·텍스트 기록과 AI 응답 기반 단어장 생성
- 참조 음성을 XTTS v2에 제공하는 커스텀 TTS 서비스
- iOS 클라이언트, 일반 API 서버, GPU 추론 서버의 연동

기상 효과, 학습 성과, 응답 지연의 개선율을 입증하는 정량 실험은 이 페이지의 주장에 포함하지 않았습니다. 화면·시연은 2024년 프로토타입의 기록이며, 현재 공개 운영 중인 서비스나 App Store 배포를 의미하지 않습니다.

## Source snapshot

문서 대조에 사용한 코드 버전입니다. 원본 저장소는 각각의 이력과 접근 권한을 유지합니다.

| Repository | Branch | Commit |
| :--- | :--- | :--- |
| [frontend](https://github.com/Wakie-Talkie/Wakie-Talkie-frontend) | `main` | [`9f378c4`](https://github.com/Wakie-Talkie/Wakie-Talkie-frontend/tree/9f378c46ed31952ff88808b678abcfe119de336b) |
| [Backend](https://github.com/Wakie-Talkie/Wakie-Talkie-Backend) | `main` | [`e9eb237`](https://github.com/Wakie-Talkie/Wakie-Talkie-Backend/tree/e9eb23757f80cea89c4c86bc61d9f0ad4a653b16) |
| [TTS](https://github.com/Wakie-Talkie/Wakie-Talkie-TTS) | `Wakie-Talkie-use-TTS` | [`9d5a6e7`](https://github.com/Wakie-Talkie/Wakie-Talkie-TTS/tree/9d5a6e7bb79a0d1ec9c9bb66d71723a19a763854) |
| [server](https://github.com/Wakie-Talkie/Wakie-Talkie-server) | `main` · Private | [`b30bc3d`](https://github.com/Wakie-Talkie/Wakie-Talkie-server/tree/b30bc3dd82d0d81cca636160d197598cf94abd4e) |

Document review date: 2026-09-28.

## Original project materials

| Material | Used for |
| :--- | :--- |
| `2024-1 Capstone final report team4.pdf` | 최종 기능, 음성·알람·기록·TTS 처리, 실제 앱 스크린샷 |
| `2024-1 Capstone project manual team4.pptx` | 개발 환경, 저장소 구성, 실행 방법, 원본 앱 아이콘 |
| `final_ppt (1).pdf` | 기능별 발표 화면 |
| `공학학술제 발표ppt 수동 ver.pptx` | 대표 이미지, 실제 기기 시연 영상 |
| `Proposal_team4.pdf` | 초기 기획과 최종 구현의 범위 구분 |
| `2024-1 캡스톤 중간발표.pdf`, `proposal_presentation_team4.pdf`, `final.pptx` | 중간 기획·설계 보조 자료 |
| 2024 CAU 공학학술제 프로젝트 포스터 | 프로젝트 소개·팀 정보·시연 맥락 |

README와 갤러리에 사용한 이미지는 원본 발표 자료와 보고서에서 추출하고 표시 크기만 조정했습니다. GIF는 발표 파일에 포함된 시연 영상 전체를 무음·8 fps·가로 280 px로 변환했습니다. 원본 보고서·발표 파일과 DB, 사용자 음성 파일은 이 저장소에 포함하지 않습니다.

## Open-source credits

커스텀 음성 합성은 [Coqui TTS](https://github.com/coqui-ai/TTS)와 XTTS v2를 기반으로 합니다. Wakie-Talkie의 TTS 저장소는 해당 프로젝트의 fork이며, 프로젝트 전용 API와 참조 음성 처리 코드를 추가했습니다. 오픈소스 코드와 모델에는 각각의 라이선스·사용 조건이 적용됩니다. 자세한 내용은 [원본 코드의 라이선스](https://github.com/coqui-ai/TTS/blob/dev/LICENSE.txt)와 모델 배포 문서를 확인하세요.

원본 앱 화면·아이콘·발표 자료의 크레딧은 Wakie-Talkie 팀에 있습니다. 이 소개 저장소에 별도의 일괄 오픈소스 라이선스를 추가하지 않았으며, 연결된 각 코드 저장소의 라이선스를 변경하지 않습니다.

[← Project overview](../README.md)
