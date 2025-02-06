# HEVC 비디오 프레임 간 참조 관계 분석 도구 사용법

## 테스트 목적
- ExoPlayer를 이용 중인 RTSP 스트리밍 플레이어 개발 도중, 

## 테스트 환경
- M1 chip MacOS Sequoia 15.2 Mac Studio에서 진행했습니다.

## 사전 준비 사항
- h265 코덱으로 인코딩된 비디오 파일을 준비합니다.
- Xcode의 최신 버전이 설치돼 있어야 합니다.
- homebrew가 설치돼 있어야 합니다.
- homebrew를 이용해서 ffmpeg를 설치해야 합니다.

## Quick Start