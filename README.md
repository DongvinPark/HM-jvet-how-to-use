# HEVC 비디오 프레임 간 참조 관계 분석 도구 사용법

## 테스트 개요
- HM:jvet 은 hevc 코덱으로 인코딩된 비디오 파일을 분석할 때 사용할 수 있는 유용한 command line tool 입니다.
- hevc 비디오 내의 P 프레임이 코덱에 의해서 디코드 될 때 다른 P 프레임들을 참조하고 있는지 확인할 때 사용할 수 있습니다.

## 테스트 환경
- M1 chip MacOS Sequoia 15.2 Mac Studio에서 진행했습니다.

## 사전 준비 사항
- h265 코덱으로 인코딩된 비디오 파일을 준비합니다.
- Xcode의 최신 버전이 설치돼 있어야 합니다.
- homebrew가 설치돼 있어야 합니다.
- homebrew를 이용해서 git, gcc, g++, cmake, ffmpeg를 설치해야 합니다.

## how to start
1. HM:jvet 깃랩 프로젝트를 깃 클론합니다.
```shell
git clone https://vcgit.hhi.fraunhofer.de/jvet/HM
```
<br><br/>
2. 클론한 HM 프로젝트의 루트 디렉토리로 이동한 후 biuid 디렉토리를 만들고, 그 안으로 이동합니다.
```shell
cd HM
mkdir build
cd build
```
<br><br/>
3. 다음의 명령을 순차적으로 실행하여 tool들의 executable binary 파일들을 만들어냅니다.
```shell
cmake .. -G "Xcode" -DCMAKE_CXX_FLAGS="-Wno-unused-command-line-argument"

cmake --build . --config Release
```
<br><br/>
4. HM 프로젝트의 루트 디록토리로 이동한 후, 아래의 명령어를 실행하여 필요한 바이너리 파일의 실제 디렉토리 경로를 찾아냅니다.
```shell
find . -name "TAppDecoderAnalyser" -type f -exec ls -l {} \;

그 결과, 다음과 같이 위치가 출력될 것입니다.
-rwxr-xr-x  1 dongvin99  staff  994064  2  6 15:37 ./bin/xcode/clang-16.0/x86_64/release/TAppDecoderAnalyser

맥OS의 파인더를 이용해서 프로젝트 루트 디렉토리 >> bin >> xcode >> clang-16.0 >> x86_64 >> release 의 순서로 클릭해 들어가면 실행파일들을 찾아낼 수 있습니다.
```
<br><br/>
5. TAppDecoderAnalyser를 사용합니다. 해당 tool은 raw H265 bitstream을 입력으로 받기 때문에, 분석 대상 비디오 파일이 존재하는 디렉토리로 이동해서 아래의 명령으로 bitstream을 만들어줍니다.
```shell
만들어진 bitstream의 이름은 input.265이고, 다음 단계에서 입력 파일로서 사용됩니다.
ffmpeg -i target-video.mp4 -c:v copy -bsf:v hevc_mp4toannexb input.265
```
<br><br/>
6. TAppDecoderAnalyser 실행파일이 위치하는 디렉토리로 돌아와서 아래의 명령어를 실행하여 비디오 프레임 간의 참조 관계를 출력하게 만듭니다.
```shell
./TAppDecoderAnalyser -b /.../path_to_input_bitstream_file/.../input.265 -o output.yuv --OutputDecodedSEIMessagesFilename ref_info.txt
```
<br><br/>
7. 터미널에서 결과를 확인합니다.
```text
실제 실행 결과의 일부를 가져왔습니다.
POC 0는 현재 gop 안에서 첫 번째 프레임이므로 Key 프레임입니다.
POC 5 번은 gop 인에서 다섯 번째 프레임이므로 P 프레임이고, 디코드 되는 과정에서 Key 프레임과 P 프레임 4개(POC 1~4)를 참조했음을 알 수 있습니다.

... 초략

POC    0 TId: 0 ( I-SLICE, QP 32 ) [DT  0.154] [L0 ] [L1 ] [:,(unk)] 
POC    1 TId: 0 ( P-SLICE, QP 40 ) [DT  0.034] [L0 0 ] [L1 ] [:,(unk)] 
POC    2 TId: 0 ( P-SLICE, QP 40 ) [DT  0.034] [L0 1 0 ] [L1 ] [:,(unk)] 
POC    3 TId: 0 ( P-SLICE, QP 39 ) [DT  0.034] [L0 2 1 0 ] [L1 ] [:,(unk)] 
POC    4 TId: 0 ( P-SLICE, QP 39 ) [DT  0.036] [L0 3 2 1 0 ] [L1 ] [:,(unk)] 
POC    5 TId: 0 ( P-SLICE, QP 39 ) [DT  0.036] [L0 4 3 2 1 ] [L1 ] [:,(unk)]
POC    6 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 5 4 3 2 ] [L1 ] [:,(unk)] 
POC    7 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 6 5 4 3 ] [L1 ] [:,(unk)] 
POC    8 TId: 0 ( P-SLICE, QP 38 ) [DT  0.042] [L0 7 6 5 4 ] [L1 ] [:,(unk)] 
POC    9 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 8 7 6 5 ] [L1 ] [:,(unk)] 
POC   10 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 9 8 7 6 ] [L1 ] [:,(unk)] 
POC   11 TId: 0 ( P-SLICE, QP 38 ) [DT  0.041] [L0 10 9 8 7 ] [L1 ] [:,(unk)] 
POC   12 TId: 0 ( P-SLICE, QP 39 ) [DT  0.037] [L0 11 10 9 8 ] [L1 ] [:,(unk)] 
POC   13 TId: 0 ( P-SLICE, QP 39 ) [DT  0.035] [L0 12 11 10 9 ] [L1 ] [:,(unk)] 
POC   14 TId: 0 ( P-SLICE, QP 39 ) [DT  0.036] [L0 13 12 11 10 ] [L1 ] [:,(unk)] 
POC   15 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 14 13 12 11 ] [L1 ] [:,(unk)] 
POC   16 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 15 14 13 12 ] [L1 ] [:,(unk)] 
POC   17 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 16 15 14 13 ] [L1 ] [:,(unk)] 
POC   18 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 17 16 15 14 ] [L1 ] [:,(unk)] 
POC   19 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 18 17 16 15 ] [L1 ] [:,(unk)] 
POC   20 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 19 18 17 16 ] [L1 ] [:,(unk)] 
POC   21 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 20 19 18 17 ] [L1 ] [:,(unk)] 
POC   22 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 21 20 19 18 ] [L1 ] [:,(unk)] 
POC   23 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 22 21 20 19 ] [L1 ] [:,(unk)] 
POC   24 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 23 22 21 20 ] [L1 ] [:,(unk)] 
POC   25 TId: 0 ( P-SLICE, QP 38 ) [DT  0.040] [L0 24 23 22 21 ] [L1 ] [:,(unk)] 
POC   26 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 25 24 23 22 ] [L1 ] [:,(unk)] 
POC   27 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 26 25 24 23 ] [L1 ] [:,(unk)] 
POC   28 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 27 26 25 24 ] [L1 ] [:,(unk)] 
POC   29 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 28 27 26 25 ] [L1 ] [:,(unk)] 
POC   30 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 29 28 27 26 ] [L1 ] [:,(unk)] 
POC   31 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 30 29 28 27 ] [L1 ] [:,(unk)] 
POC   32 TId: 0 ( P-SLICE, QP 38 ) [DT  0.038] [L0 31 30 29 28 ] [L1 ] [:,(unk)] 
POC   33 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 32 31 30 29 ] [L1 ] [:,(unk)] 
POC   34 TId: 0 ( P-SLICE, QP 37 ) [DT  0.041] [L0 33 32 31 30 ] [L1 ] [:,(unk)] 
POC   35 TId: 0 ( P-SLICE, QP 37 ) [DT  0.039] [L0 34 33 32 31 ] [L1 ] [:,(unk)] 
POC   36 TId: 0 ( P-SLICE, QP 38 ) [DT  0.034] [L0 35 34 33 32 ] [L1 ] [:,(unk)] 
POC   37 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 36 35 34 33 ] [L1 ] [:,(unk)] 
POC   38 TId: 0 ( P-SLICE, QP 38 ) [DT  0.036] [L0 37 36 35 34 ] [L1 ] [:,(unk)] 
POC   39 TId: 0 ( P-SLICE, QP 37 ) [DT  0.040] [L0 38 37 36 35 ] [L1 ] [:,(unk)] 
POC   40 TId: 0 ( P-SLICE, QP 37 ) [DT  0.041] [L0 39 38 37 36 ] [L1 ] [:,(unk)] 
POC   41 TId: 0 ( P-SLICE, QP 37 ) [DT  0.040] [L0 40 39 38 37 ] [L1 ] [:,(unk)] 
POC   42 TId: 0 ( P-SLICE, QP 37 ) [DT  0.039] [L0 41 40 39 38 ] [L1 ] [:,(unk)] 
POC   43 TId: 0 ( P-SLICE, QP 38 ) [DT  0.040] [L0 42 41 40 39 ] [L1 ] [:,(unk)] 
POC   44 TId: 0 ( P-SLICE, QP 38 ) [DT  0.040] [L0 43 42 41 40 ] [L1 ] [:,(unk)] 
POC   45 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 44 43 42 41 ] [L1 ] [:,(unk)] 
POC   46 TId: 0 ( P-SLICE, QP 37 ) [DT  0.043] [L0 45 44 43 42 ] [L1 ] [:,(unk)] 
POC   47 TId: 0 ( P-SLICE, QP 37 ) [DT  0.044] [L0 46 45 44 43 ] [L1 ] [:,(unk)] 
POC   48 TId: 0 ( P-SLICE, QP 37 ) [DT  0.043] [L0 47 46 45 44 ] [L1 ] [:,(unk)] 
POC   49 TId: 0 ( P-SLICE, QP 38 ) [DT  0.042] [L0 48 47 46 45 ] [L1 ] [:,(unk)] 
POC   50 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 49 48 47 46 ] [L1 ] [:,(unk)] 
POC   51 TId: 0 ( P-SLICE, QP 38 ) [DT  0.039] [L0 50 49 48 47 ] [L1 ] [:,(unk)] 
POC   52 TId: 0 ( P-SLICE, QP 37 ) [DT  0.042] [L0 51 50 49 48 ] [L1 ] [:,(unk)] 
POC   53 TId: 0 ( P-SLICE, QP 37 ) [DT  0.041] [L0 52 51 50 49 ] [L1 ] [:,(unk)] 
POC   54 TId: 0 ( P-SLICE, QP 37 ) [DT  0.041] [L0 53 52 51 50 ] [L1 ] [:,(unk)] 
POC   55 TId: 0 ( P-SLICE, QP 38 ) [DT  0.040] [L0 54 53 52 51 ] [L1 ] [:,(unk)] 
POC   56 TId: 0 ( P-SLICE, QP 38 ) [DT  0.042] [L0 55 54 53 52 ] [L1 ] [:,(unk)] 
POC   57 TId: 0 ( P-SLICE, QP 38 ) [DT  0.037] [L0 56 55 54 53 ] [L1 ] [:,(unk)] 
POC   58 TId: 0 ( P-SLICE, QP 38 ) [DT  0.040] [L0 57 56 55 54 ] [L1 ] [:,(unk)] 
POC   59 TId: 0 ( P-SLICE, QP 38 ) [DT  0.040] [L0 58 57 56 55 ] [L1 ] [:,(unk)]

... 후략
```
<br><br/>

## quick start
- 본 프로젝트를 깃 클론한 다음, 본 프로젝트의 루트 디렉토리 내에 분석 대상 비디오 파일을 복사해 넣습니다.
- 아래의 명령어 두 개를 차례대로 실행합니다.
```text
ffmpeg -i target-video.mp4 -c:v copy -bsf:v hevc_mp4toannexb input.265

./TAppDecoderAnalyser -b input.265 -o output.yuv --OutputDecodedSEIMessagesFilename ref_info.txt
```