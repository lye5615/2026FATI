# FATI 환경설정_Mac_OS_ver.

# 목차

1. macOS 개발 환경 준비
   1.1 터미널 실행  
   1.2 Xcode Command Line Tools 설치  

2. Homebrew 설치

3. 필수 프로그램 설치
   3.1 Git 설치  
   3.2 Python 설치  
   3.3 jq 설치  
   3.4 Bash 설치  
   3.5 Docker Desktop 설치  

4. DeepRacer 로컬 학습환경 구축
   4.1 DRfC Repository 복제  
   4.2 Docker Image 및 초기 설정  
   4.3 Docker Buildx 확인  
   4.4 DRfC 초기 설정  
   4.5 `/tmp/sagemaker` 권한 설정  
   4.6 DRfC 명령어 활성화  
   4.7 트랙 이름 변경  
   4.8 학습 데이터 업로드  
   4.9 학습 시작  
   4.10 이미 존재하는 모델 경로 처리 (`-w` 옵션)  
   4.11 학습 상태 확인  
   4.12 Docker 서비스 상태 확인  
   4.13 실제 학습 로그 확인  
   4.14 학습 로그 실시간 확인  
   4.15 초기 체크포인트 관련 메시지 확인  
   4.16 학습 종료  
   4.17 학습된 모델 확인  
   4.18 모델 경로가 없는 경우 확인 방법  
   4.19 학습 진행 상황 Viewer 확인  
   4.20 Viewer 종료  

5. Reward Function 수정 후 다시 학습하기

6. 오랜만에 MacBook을 켠 뒤 다시 학습하는 방법
   6.1 Docker Desktop 실행  
   6.2 Docker 상태 확인  
   6.3 Bash 진입  
   6.4 DeepRacer 폴더로 이동  
   6.5 DRfC 명령어 활성화  
   6.6 필요한 파일 업로드  
   6.7 학습 시작  

7. MacBook에서 Ubuntu 문서와 다른 점

8. 문제 해결
   8.1 `No configuration file.` / `bad substitution`  
   8.2 `ERROR: AWS credentials not found in profile 'minio'.`  
   8.3 `Connection was closed ... http://localhost:9000/...`  
   8.4 `Sagemaker is not running.`  
   8.5 Docker 서비스 상태로 학습 성공 여부 확인  
   8.6 `/tmp/sagemaker` 권한 문제 해결  
   8.7 `ERROR: Colima is not running.`  
   8.8 `Selected path ... exists. Delete it, or use -w option.`  
   8.9 Apple Silicon 아키텍처 확인  

9. 이 저장소에 적용한 소스 수정
   9.1 `bin/activate.sh` 수정  
   9.2 `bin/module/summary.sh` 수정  
   9.3 `bin/scripts_wrapper.sh` 수정  
   9.4 Docker Desktop 환경 대응 수정  
   9.5 Colima 의존 로직 수정  
   9.6 `/tmp/sagemaker` 자동 권한 처리 수정  

10. 핵심 명령어 정리
   10.1 DRfC 명령어가 실행되지 않을 때  
   10.2 현재 설정 확인  
   10.3 학습 파일 업로드  
   10.4 학습 시작  
   10.5 기존 모델 삭제 후 학습 시작  
   10.6 Docker 서비스 상태 확인  
   10.7 최근 학습 로그 확인  
   10.8 Episode 로그만 확인  
   10.9 Episode 실시간 확인  
   10.10 실제 학습 종료  
   10.11 Viewer 시작  
   10.12 Viewer 종료  
   10.13 모델 위치 찾기  

11. 학습 오류 발생 시 빠른 확인 순서
   11.1 `docker service ls`로 서비스 상태 확인  
   11.2 모든 서비스가 `1/1`인 경우  
   11.3 `rl_coach`가 `0/1`인 경우  
   11.4 `PermissionError` 발생 시 해결  
   11.5 `/tmp/sagemaker` 권한 재설정  
   11.6 학습 재시작  
   11.7 Episode 증가 여부 확인  
   11.8 최종 학습 정상 여부 판단 기준

---

## 순서

1. macOS 개발 환경 준비
2. Homebrew 설치
3. 필수 프로그램 설치
4. DeepRacer 로컬 학습환경 구축

# 1. macOS 개발 환경 준비

## 터미널 실행

`⌘ + Space`를 누른 뒤 `Terminal`을 검색하여 실행합니다.

## Xcode Command Line Tools 설치

```sh
xcode-select --install
```

설치가 완료되었는지 확인합니다.

```sh
xcode-select -p
```

정상적으로 설치되었다면 다음과 비슷하게 출력됩니다.

```text
/Library/Developer/CommandLineTools
```

---

# 2. Homebrew 설치

Homebrew는 macOS에서 프로그램과 패키지를 설치하기 위한 패키지 관리자입니다.

Homebrew가 설치되어 있지 않다면 설치합니다.

설치 후 다음 명령어로 정상적으로 설치되었는지 확인합니다.

```sh
brew --version
```

버전 정보가 출력되면 정상입니다.

---

# 3. 필수 프로그램 설치

## Git 설치

```sh
brew install git
```

설치 확인:

```sh
git --version
```

이미 Git이 설치되어 있다면 설치 과정은 생략해도 됩니다.

---

## Python 설치

```sh
brew install python
```

설치 확인:

```sh
python3 --version
pip3 --version
```

---

## jq 설치

```sh
brew install jq
```

설치 확인:

```sh
jq --version
```

---

## Bash 설치

macOS의 기본 셸은 zsh이고, 기본 탑재된 bash도 3.2 버전입니다.

DRfC 스크립트(`bin/activate.sh`)는 **bash 4 이상**에서 동작하므로 최신 bash를 설치합니다.

```sh
brew install bash
```

설치 확인:

```sh
/opt/homebrew/bin/bash --version
```

`version 5.x` 이상이 출력되면 정상입니다.

> Intel Mac이라면 경로가 `/usr/local/bin/bash`일 수 있습니다.

---

## Docker Desktop 설치

```sh
brew install --cask docker
```

설치가 완료되면 Docker Desktop을 실행합니다.

```sh
open -a Docker
```

또는 `⌘ + Space`를 누른 뒤 `Docker`를 검색하여 직접 실행해도 됩니다.

> **중요**
>
> MacBook에서는 Docker 명령어를 사용하기 전에 **Docker Desktop이 실행되어 있어야 합니다.**

Docker가 완전히 실행된 후 다음 명령어를 입력합니다.

```sh
docker --version
docker compose version
docker ps
```

`docker ps` 실행 시 오류가 발생하지 않으면 정상입니다.

---

# DeepRacer 로컬 학습환경 구축 (MacBook)

Docker Desktop이 실행된 상태에서 진행합니다.

모든 명령어는 macOS의 Terminal에서 입력합니다.

## 순서

1. DRfC Repository 복제
2. Docker Image 및 초기 설정
3. DRfC 명령어 활성화
4. 트랙 설정
5. 학습 데이터 업로드
6. 학습 시작
7. 학습 상태 확인
8. 학습 종료
9. 학습된 모델 확인
10. Viewer 확인

전부 완료하면 로컬 환경에서 DeepRacer 모델을 학습할 수 있습니다.

---

# 1. DRfC Repository 복제

Repository를 받아둘 폴더를 만들고 그 안으로 이동합니다.

```sh
mkdir -p ~/Documents/GitHub
cd ~/Documents/GitHub
```

DeepRacer for Cloud Repository를 복제합니다.

```sh
git clone https://github.com/aws-deepracer-community/deepracer-for-cloud
```

복제한 폴더로 이동합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
```

> 이 문서는 이후 모든 경로를 `~/Documents/GitHub/deepracer-for-cloud` 기준으로 설명합니다.
>
> 다른 위치에 복제했다면 해당 경로로 바꿔서 읽으면 됩니다.
> 현재 위치는 `pwd`로 확인할 수 있습니다.

파일이 정상적으로 받아졌는지 확인합니다.

```sh
ls
```

---

# 2. Docker Image 및 초기 설정

먼저 Docker Desktop이 실행되어 있는지 확인합니다.

```sh
docker ps
```

오류가 발생하지 않으면 정상입니다.

## Docker Buildx 확인

```sh
docker buildx version
```

정상적으로 버전이 출력되면 다음 단계로 넘어갑니다.

## DRfC 초기 설정

Apple Silicon(M1/M2/M3/M4) MacBook에서는 GPU가 아닌 **CPU 환경**으로 설정합니다.

```sh
bin/init.sh -a cpu -c local
```

여기서:

- `-a cpu` : CPU를 이용하여 학습
- `-c local` : 로컬 저장소(MinIO)를 사용

초기 실행 시 필요한 Docker Image를 다운로드하기 때문에 시간이 오래 걸릴 수 있습니다.

> ### 중요: `sudo`를 붙이지 마세요.
>
> 다음과 같이 실행하면 안 됩니다.
>
> ```sh
> sudo bin/init.sh -a cpu -c local
> ```
>
> `init.sh`는 관리자 권한이 필요한 부분에서 스스로 `sudo`를 호출합니다.
> 전체 명령어를 `sudo`로 실행하면 `data/`, `tmp/`, `custom_files/`, `system.env` 등이 root 소유가 되거나 MinIO 접속 정보가 `/var/root/.aws`에 생성될 수 있습니다.
>
> 그 결과 이후 다음과 같은 오류가 발생할 수 있습니다.
>
> ```text
> ERROR: AWS credentials not found in profile 'minio'.
> ```

이미 `sudo`로 실행했다면 다음과 같이 권한을 복구한 뒤 초기 설정을 다시 진행합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
sudo chown -R $(id -un):staff .
ln -sfn ~/.aws docker/volumes/.aws
bin/init.sh -a cpu -c local
```

---

## `/tmp/sagemaker` 권한 설정

macOS + Docker Desktop 환경에서는 SageMaker Local이 사용하는 `/tmp/sagemaker` 디렉터리의 쓰기 권한 때문에 학습이 실패할 수 있습니다.

초기 설정 후 다음 명령어를 실행하는 것을 권장합니다.

```sh
sudo mkdir -p /tmp/sagemaker
sudo chown -R $(id -u):$(id -g) /tmp/sagemaker
```

권한을 확인합니다.

```sh
ls -ld /tmp/sagemaker
```

이 설정이 되어 있지 않으면 학습 시작 시 다음 오류가 발생할 수 있습니다.

```text
Sagemaker is not running.
```

컨테이너 로그에는 다음과 같은 실제 원인이 나타날 수 있습니다.

```text
PermissionError: [Errno 13] Permission denied: '/tmp/sagemaker/...'
```

> **MacBook에서는 다음과 같은 Ubuntu용 명령어를 실행할 필요가 없습니다.**
>
> ```text
> sudo service docker start
> sudo reboot -f
> ```
>
> Docker Desktop만 실행되어 있으면 됩니다.

---

# 3. DRfC 명령어 활성화

이후 DeepRacer 관련 명령어는 기본적으로 `deepracer-for-cloud` 폴더에서 실행합니다.

> ### 중요
>
> 터미널을 새로 열면 기본 셸이 zsh입니다.
>
> `source bin/activate.sh`를 실행하기 전에 반드시 **Homebrew로 설치한 최신 bash로 먼저 진입**해야 합니다.
>
> zsh나 macOS 기본 bash 3.2에서 실행하면 다음과 같은 오류가 발생할 수 있습니다.
>
> ```text
> No configuration file.
> bad substitution
> ```

bash로 진입합니다.

```sh
bash
```

프롬프트가 다음과 비슷하게 바뀌면 정상입니다.

```text
bash-5.3$
```

프로젝트 폴더로 이동합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
```

DRfC 명령어를 활성화합니다.

```sh
source bin/activate.sh
```

필요한 설정을 업데이트합니다.

```sh
dr-update
dr-update-env
```

> ## 대원칙
>
> `dr-start-training`, `dr-update-env`, `dr-summary` 등 `dr-`로 시작하는 명령어가
>
> ```text
> command not found
> ```
>
> 와 같이 실행되지 않는다면 가장 먼저 다음 명령어를 다시 실행합니다.
>
> ```sh
> bash
> cd ~/Documents/GitHub/deepracer-for-cloud
> source bin/activate.sh
> ```

---

# 4. 트랙 이름 변경

학습에 사용할 트랙을 `reInvent2019_track`으로 변경합니다.

프로젝트 폴더에서 `run.env` 파일을 엽니다.

```sh
nano run.env
```

파일에서 다음 항목을 찾습니다.

```text
DR_WORLD_NAME=reinvent_base
```

아래와 같이 변경합니다.

```text
DR_WORLD_NAME=reInvent2019_track
```

저장:

```text
Ctrl + O
Enter
Ctrl + X
```

설정을 반영합니다.

```sh
dr-update-env
```

현재 설정을 확인합니다.

```sh
dr-summary
```

다음과 같이 표시되면 정상적으로 적용된 것입니다.

```text
World / track    reInvent2019_track
```

---

# 5. 학습 데이터 업로드

학습에 필요한 설정 파일과 Reward Function 등을 로컬 MinIO 저장소에 업로드합니다.

```sh
dr-upload-custom-files
```

특히 다음과 같은 파일을 수정했다면 학습을 시작하기 전에 다시 업로드해야 합니다.

```text
custom_files/reward_function.py
custom_files/model_metadata.json
custom_files/hyperparameters.json
```

수정 후:

```sh
dr-upload-custom-files
```

---

# 6. 학습 시작

DeepRacer 모델 학습을 시작합니다.

```sh
dr-start-training
```

처음 실행하는 경우 Docker Image와 학습 환경을 준비하는 과정 때문에 시간이 오래 걸릴 수 있습니다.

- 만약 'ERROR: Colima is not running. Start it with: colima start', 'Sagemaker is not running.' 이런 문구가 떴다면? -> 아래 7. 학습 상태 확인을 봅시다

## 이미 존재하는 모델 경로 (`-w` 옵션)

이전 학습 결과가 같은 모델 경로에 남아 있으면 다음 메시지가 나타날 수 있습니다.

```text
Selected path s3://bucket/rl-deepracer-sagemaker exists. Delete it, or use -w option. Exiting.
```

기존 학습 결과를 버리고 처음부터 다시 학습하려면 `-w` 옵션을 사용합니다.

```sh
dr-start-training -w
```

> **주의**
>
> `-w`는 해당 모델 경로의 기존 학습 결과를 삭제합니다.
>
> 기존 결과를 남겨야 한다면 `run.env`의 `DR_LOCAL_S3_MODEL_PREFIX`를 새로운 이름으로 변경한 뒤 다음 명령어를 실행합니다.
>
> ```sh
> dr-update-env
> ```

---

# 7. 학습 상태 확인

`dr-start-training`의 마지막에 표시되는 문구만으로 학습 성공 여부를 판단하지 않는 것이 좋습니다.

특히 Docker Desktop 환경에서는 DRfC의 macOS용 Colima 관련 로직 때문에 다음과 같은 메시지가 출력될 수 있습니다.

```text
ERROR: Colima is not running. Start it with: colima start
```

또는:

```text
Sagemaker is not running.
```

**Docker Desktop을 사용하고 있다면 위 문구보다 실제 Docker 서비스 상태와 학습 로그를 기준으로 판단합니다.**

## 1단계: Docker 서비스 확인

```sh
docker service ls
```

정상적인 예:

```text
NAME                    REPLICAS
deepracer-0_rl_coach    1/1
deepracer-0_robomaker   1/1
s3_minio                1/1
```

세 서비스가 모두 `1/1`이면 실행 중입니다.

---

## 2단계: 실제 학습 로그 확인

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

다음과 같이 Episode가 증가하면 실제 학습이 진행 중입니다.

```text
Training> Name=main_level/agent, Worker=0, Episode=1, Total reward=48.65, Steps=102, Training iteration=0
Training> Name=main_level/agent, Worker=0, Episode=2, Total reward=58.19, Steps=194, Training iteration=0
Training> Name=main_level/agent, Worker=0, Episode=3, Total reward=23.37, Steps=240, Training iteration=0
```

일정량의 Episode가 누적되면 정책 학습 로그도 나타납니다.

```text
Policy training> Surrogate loss=...
Policy training> KL divergence=...
Policy training> Entropy=...
```

체크포인트가 저장될 때는 다음과 같은 로그가 나타납니다.

```text
Checkpoint> Saving in path=['./checkpoint_sagemaker/agent/1_Step-880.ckpt']
Uploaded 3 files for checkpoint 1
```

모델 파일이 생성되는 로그도 확인할 수 있습니다.

```text
saved intermediate frozen graph: rl-deepracer-sagemaker/model/model_1.pb
```

따라서 다음 로그가 나타난다면 학습이 정상적으로 진행되고 있다고 볼 수 있습니다.

```text
Training> ...
Policy training> ...
Checkpoint> Saving...
```

---
## 참고

### 학습 로그를 실시간으로 확인하는 방법

전체 실시간 로그:

```sh
docker service logs -f deepracer-0_rl_coach
```

Episode 관련 로그만 확인:

```sh
docker service logs deepracer-0_rl_coach 2>&1 | grep "Training>"
```

Episode를 실시간으로 확인:

```sh
docker service logs -f deepracer-0_rl_coach 2>&1 | grep --line-buffered "Training>"
```

실시간 로그 확인을 종료하려면:

```text
Ctrl + C
```

를 누릅니다.

> 여기서 `Ctrl + C`는 **로그 출력만 종료**합니다.
>
> Docker에서 실행 중인 실제 학습은 계속 진행됩니다.

---

### 처음 학습할 때 체크포인트 관련 메시지가 나오는 경우

새 모델을 처음 학습할 때 다음 메시지가 출력될 수 있습니다.

```text
Unable to find deepracer checkpoint json
Unable to find the best deepracer checkpoint number.
Unable to find the last deepracer checkpoint number.
```

처음에는 기존 체크포인트가 없기 때문에 나타날 수 있습니다.

이후 다음과 같이 새로운 체크포인트가 정상적으로 생성된다면 문제가 아닙니다.

```text
Checkpoint> Saving in path=['./checkpoint_sagemaker/agent/1_Step-880.ckpt']
Uploaded 3 files for checkpoint 1
```

또한 이후 다음과 같이 체크포인트 정보를 정상적으로 읽는 로그가 나타날 수 있습니다.

```text
Successfully downloaded deepracer checkpoint json
Best checkpoint number: 0, Last checkpoint number: 0
```

---

# 8. 학습 종료

실제 학습을 종료하려면 다음 명령어를 사용합니다.

```sh
dr-stop-training
```

학습 로그를 실시간으로 보고 있었다면 먼저:

```text
Ctrl + C
```

를 눌러 로그 출력을 종료한 뒤:

```sh
dr-stop-training
```

을 실행합니다.

> `Ctrl + C`만으로는 실제 Docker 학습 서비스가 종료되지 않을 수 있습니다.
>
> 실제 학습 종료에는 `dr-stop-training`을 사용합니다.

종료 후 상태를 확인하고 싶다면:

```sh
docker service ls
```

를 실행합니다.

---

# 9. 학습된 모델 확인

학습 결과는 로컬 MinIO 저장소에 저장됩니다.

기본 모델 prefix를 사용했다면 다음 위치를 확인합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud/data/minio/bucket/rl-deepracer-sagemaker
```

모델 폴더를 확인합니다.

```sh
ls model
```

모델 파일이 정상적으로 생성되어 있다면 학습 결과가 저장된 것입니다.

## 경로가 없는 경우

위 경로가 존재하지 않는다면 다음 명령어로 `model` 폴더를 찾습니다.

```sh
find ~/Documents/GitHub/deepracer-for-cloud/data/minio -name "model" -type d
```

학습 설정이나 모델 prefix에 따라 실제 저장 경로가 달라질 수 있으므로 출력된 경로를 사용하면 됩니다.

---

# 10. 학습 진행 상황 Viewer 확인

학습이 진행되는 동안 Viewer를 통해 시뮬레이션 화면을 확인할 수 있습니다.

학습 로그가 출력되고 있는 터미널은 그대로 두고 **새 Terminal 탭 또는 창**을 엽니다.

새 터미널에서 bash로 진입합니다.

```sh
bash
```

프로젝트 폴더로 이동합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
```

DRfC 명령어를 다시 활성화합니다.

```sh
source bin/activate.sh
```

Viewer를 실행합니다.

```sh
dr-start-viewer
```

터미널에 다음과 같은 형태의 주소가 출력됩니다.

```text
http://...
```

출력된 주소를 브라우저에 입력하면 학습 화면을 확인할 수 있습니다.

## Viewer 종료

```sh
dr-stop-viewer
```

---

# Reward Function 수정 후 다시 학습하기

기본 Reward Function 대신 직접 Reward Function을 수정하여 모델의 성능을 개선할 수 있습니다.

Reward Function 파일:

```text
custom_files/reward_function.py
```

파일을 수정한 뒤 학습 데이터를 다시 업로드합니다.

```sh
bash
cd ~/Documents/GitHub/deepracer-for-cloud
source bin/activate.sh
dr-upload-custom-files
```

그다음 학습을 시작합니다.

기존 모델 경로를 그대로 사용하면서 이전 결과를 삭제해도 된다면:

```sh
dr-start-training -w
```

기존 모델을 보존하려면 `run.env`에서 `DR_LOCAL_S3_MODEL_PREFIX`를 새로운 이름으로 변경한 뒤:

```sh
dr-update-env
dr-start-training
```

을 실행합니다.

---

# 오랜만에 MacBook을 켠 뒤 다시 학습하는 방법

컴퓨터를 종료했다가 다시 켠 경우 DeepRacer 환경 전체를 다시 설치할 필요는 없습니다.

## 1. Docker Desktop 실행

```sh
open -a Docker
```

Docker가 완전히 실행될 때까지 기다립니다.

## 2. Docker 상태 확인

```sh
docker ps
```

오류가 발생하지 않으면 정상입니다.

## 3. bash 진입

```sh
bash
```

## 4. DeepRacer 폴더로 이동

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
```

## 5. DRfC 명령어 활성화

```sh
source bin/activate.sh
```

## 6. 필요한 파일 업로드

Reward Function이나 설정 파일을 수정했다면 다시 업로드합니다.

```sh
dr-upload-custom-files
```

수정한 것이 없다면 상황에 따라 생략할 수 있습니다.

## 7. 학습 시작

```sh
dr-start-training
```

기존 모델 경로를 초기화하고 다시 학습해야 한다면:

```sh
dr-start-training -w
```

즉, 이미 환경설정을 완료한 MacBook이라면 일반적으로 다음 순서로 다시 시작하면 됩니다.

```sh
open -a Docker

bash
cd ~/Documents/GitHub/deepracer-for-cloud
source bin/activate.sh

dr-upload-custom-files
dr-start-training
```

---

# MacBook에서 Ubuntu 문서와 다른 점

| Ubuntu 문서 | MacBook |
|---|---|
| `su`로 root 전환 | 필요 없음 |
| `sudo passwd root` | 필요 없음 |
| `sudo service docker start` | 필요 없음 |
| Ubuntu 터미널 사용 | macOS Terminal 사용 |
| Docker daemon 직접 실행 | Docker Desktop 실행 |
| `/data/minio/...` | `~/Documents/GitHub/deepracer-for-cloud/data/minio/...` |
| Linux 패키지 관리자 `apt` | macOS 패키지 관리자 `brew` |

---

# 문제 해결 (증상별)

# 1. `No configuration file.` / `bad substitution`

다음과 같은 오류가 발생하는 경우:

```text
grep: /Users/.../system.env: No such file or directory
No configuration file.
```

또는:

```text
bin/activate.sh: line 116: ${DR_DOCKER_STYLE,,}: bad substitution
```

zsh 또는 macOS 기본 bash 3.2에서 `source bin/activate.sh`를 실행했을 가능성이 있습니다.

Homebrew bash가 설치되어 있는지 확인합니다.

```sh
/opt/homebrew/bin/bash --version
```

그다음:

```sh
bash
cd ~/Documents/GitHub/deepracer-for-cloud
source bin/activate.sh
```

을 실행합니다.

---

# 2. `ERROR: AWS credentials not found in profile 'minio'.`

다음 오류가 발생하는 경우:

```text
ERROR: AWS credentials not found in profile 'minio'.
```

`bin/init.sh`를 `sudo`로 실행하여 MinIO 접속 정보가 `/var/root/.aws`에 만들어졌을 가능성이 있습니다.

다음과 같이 복구합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
sudo chown -R $(id -un):staff .
ln -sfn ~/.aws docker/volumes/.aws
bin/init.sh -a cpu -c local
```

---

# 3. `Connection was closed ... http://localhost:9000/...`

다음과 같은 오류가 발생하는 경우:

```text
fatal error: Connection was closed before we received a valid response
from endpoint URL: "http://localhost:9000/bucket?list-type=2&..."
```

MinIO 서비스가 정상적으로 실행되지 않았을 가능성이 있습니다.

상태를 확인합니다.

```sh
docker service ls
```

`s3_minio`가 다음과 같이 `0/1`이라면 정상 실행되지 않은 상태입니다.

```text
s3_minio    0/1
```

MinIO 로그를 확인합니다.

```sh
docker logs $(docker ps -a --filter name=s3_minio --format '{{.Names}}' | head -1)
```

로그에 다음과 같은 메시지가 있다면:

```text
Unable to initialize backend: file access denied
```

`data/` 폴더가 root 소유라 MinIO가 쓰지 못하는 문제일 수 있습니다.

프로젝트 권한을 복구합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
sudo chown -R $(id -un):staff .
```

기존 MinIO Stack을 제거합니다.

```sh
docker stack rm s3
```

그다음 DRfC 환경을 다시 활성화합니다.

```sh
source bin/activate.sh
```

다시 상태를 확인합니다.

```sh
docker service ls
```

정상이라면:

```text
s3_minio    1/1
```

로 표시됩니다.

---

# 4. `Sagemaker is not running.`

macOS + Docker Desktop 환경에서 가장 주의해야 하는 오류입니다.

`dr-start-training` 또는 `dr-start-training -w` 실행 시 다음과 같이 표시될 수 있습니다.

```text
Creating service deepracer-0_rl_coach
Creating service deepracer-0_robomaker
Waiting up to 15 seconds for Sagemaker to start up...
Sagemaker is not running.
```

이 메시지가 나타났다고 해서 **무조건 학습에 실패한 것은 아닙니다.**

Docker Desktop 환경에서는 반드시 실제 서비스 상태와 로그를 확인해야 합니다.

## Step 1. 서비스 상태 확인

```sh
docker service ls
```

### 모두 `1/1`인 경우

```text
NAME                    REPLICAS
deepracer-0_rl_coach    1/1
deepracer-0_robomaker   1/1
s3_minio                1/1
```

서비스 자체는 실행되고 있습니다.

다음 명령어로 Episode가 증가하는지 확인합니다.

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

Episode가 계속 증가한다면 학습은 정상입니다.

```text
Training> Name=main_level/agent, Worker=0, Episode=74, ...
Training> Name=main_level/agent, Worker=0, Episode=75, ...
Training> Name=main_level/agent, Worker=0, Episode=76, ...
```

이 경우 `Sagemaker is not running.` 문구는 무시하고 실제 학습 상태를 기준으로 판단합니다.

### `rl_coach`만 `0/1`인 경우

예:

```text
NAME                    REPLICAS
deepracer-0_rl_coach    0/1
deepracer-0_robomaker   1/1
s3_minio                1/1
```

실제 학습 컨테이너가 실패한 것이므로 로그를 확인합니다.

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

다음과 같은 오류가 있다면:

```text
PermissionError: [Errno 13] Permission denied: '/tmp/sagemaker/...'
```

`/tmp/sagemaker` 권한 문제입니다.

---

## `/tmp/sagemaker` 권한 문제 해결

먼저 학습 서비스를 중지합니다.

```sh
dr-stop-training
```

디렉터리를 생성합니다.

```sh
sudo mkdir -p /tmp/sagemaker
```

기존 임시 파일을 제거합니다.

```sh
sudo rm -rf /tmp/sagemaker/*
```

현재 사용자에게 소유권을 부여합니다.

```sh
sudo chown -R $(id -u):$(id -g) /tmp/sagemaker
```

권한을 확인합니다.

```sh
ls -ld /tmp/sagemaker
```

그다음 다시 학습합니다.

```sh
dr-start-training -w
```

다시 서비스 상태를 확인합니다.

```sh
docker service ls
```

정상:

```text
deepracer-0_rl_coach      1/1
deepracer-0_robomaker     1/1
s3_minio                  1/1
```

마지막으로 실제 학습 로그를 확인합니다.

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

Episode가 증가하면 해결된 것입니다.

### 소유권 변경으로 해결되지 않는 경우

Docker Desktop의 파일 공유 권한 문제로 계속 실패한다면 테스트 목적으로 다음과 같이 `/tmp/sagemaker`에 전체 쓰기 권한을 부여할 수 있습니다.

```sh
sudo chmod 777 /tmp/sagemaker
```

이후 다시:

```sh
dr-start-training -w
```

을 실행합니다.

---

# 5. `ERROR: Colima is not running.`

Docker Desktop을 사용하는데 다음 메시지가 나타날 수 있습니다.

```text
ERROR: Colima is not running. Start it with: colima start
```

DRfC 원본의 macOS 지원 코드 일부가 Colima 환경을 전제로 작성되어 있기 때문입니다.

**Docker Desktop을 사용한다면 Colima를 별도로 실행할 필요는 없습니다.**

먼저:

```sh
docker service ls
```

를 확인합니다.

다음과 같이 모두 `1/1`이고:

```text
deepracer-0_rl_coach      1/1
deepracer-0_robomaker     1/1
s3_minio                  1/1
```

다음 명령어에서 Episode가 증가한다면:

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

```text
Training> Name=main_level/agent, Worker=0, Episode=1, ...
Training> Name=main_level/agent, Worker=0, Episode=2, ...
Training> Name=main_level/agent, Worker=0, Episode=3, ...
```

실제 학습은 정상적으로 진행되고 있는 것입니다.

> ### 판단 기준
>
> Docker Desktop 환경에서는 다음 메시지만 보고 실패로 판단하지 않습니다.
>
> ```text
> ERROR: Colima is not running.
> Sagemaker is not running.
> ```
>
> 대신 다음 두 가지를 확인합니다.
>
> **① Docker 서비스**
>
> ```sh
> docker service ls
> ```
>
> **② Episode 로그**
>
> ```sh
> docker service logs deepracer-0_rl_coach --tail 100
> ```

---

# 6. `Selected path ... exists. Delete it, or use -w option.`

다음 메시지가 나타나는 경우:

```text
Selected path s3://bucket/rl-deepracer-sagemaker exists. Delete it, or use -w option. Exiting.
```

이전 학습 결과가 같은 모델 prefix에 남아 있는 것입니다.

기존 결과를 삭제해도 된다면:

```sh
dr-start-training -w
```

기존 모델을 보존하려면 `run.env`의 `DR_LOCAL_S3_MODEL_PREFIX`를 새로운 이름으로 변경합니다.

그다음:

```sh
dr-update-env
dr-start-training
```

을 실행합니다.

---

# 7. Apple Silicon 아키텍처 확인

Apple Silicon Mac에서 문제가 발생했을 때 이미지 아키텍처를 확인할 수 있습니다.

현재 사용하는 DeepRacer 이미지 이름을 확인한 뒤:

```sh
docker image inspect <이미지이름> --format '{{.Architecture}}'
```

예를 들어:

```sh
docker image inspect awsdeepracercommunity/deepracer-simapp:6.0.6-cpu --format '{{.Architecture}}'
```

Docker Desktop의 아키텍처도 확인할 수 있습니다.

```sh
docker info --format '{{.Architecture}}'
```

Apple Silicon 환경에서는 `arm64`, `aarch64` 등 ARM64 계열로 표시될 수 있습니다.

만약 `rl_coach`가 `0/1`이면서 로그에:

```text
PermissionError: [Errno 13] Permission denied: '/tmp/sagemaker/...'
```

가 명확하게 나타난다면 아키텍처보다 `/tmp/sagemaker` 권한 문제를 먼저 해결합니다.

---

# 이 저장소에 적용한 소스 수정

DRfC 원본(upstream)의 macOS 지원은 Colima 환경을 전제로 작성된 부분이 있어 Docker Desktop 환경에서는 추가 수정이 필요할 수 있습니다.

GitHub에서 원본 저장소를 새로 clone한 경우 이러한 수정 사항이 유지되지 않을 수 있으므로 확인해야 합니다.

| 파일 | 수정 내용 | 해결한 증상 |
|---|---|---|
| `bin/activate.sh` | 맨 위에 bash 4+ 검사 추가 | zsh/bash 3.2에서 엉뚱한 오류 대신 안내 문구 출력 |
| `bin/activate.sh` | `DR_DOCKER_MAJOR_VERSION`을 사용하는 위치보다 앞에서 정의하도록 순서 교정 | `[: : 정수 값이 필요함` |
| `bin/activate.sh`, `bin/module/summary.sh` | `grep -oP` → `grep -oE` | macOS BSD grep의 `-P` 미지원 문제 |
| `bin/scripts_wrapper.sh` | `_dr_use_colima` 판별 함수 추가 | Docker Desktop에서도 Colima를 강제 요구하는 문제 |
| `bin/scripts_wrapper.sh` | Docker context가 Colima일 때만 `colima ssh` 사용 | `ERROR: Colima is required on macOS.` |
| `bin/scripts_wrapper.sh` | Docker Desktop에서는 호스트 경로를 그대로 사용 | Colima 전용 경로 처리 문제 |
| `bin/scripts_wrapper.sh` | Docker Desktop일 때 `/tmp/sagemaker`를 현재 사용자 소유로 생성 | `Sagemaker is not running.` 및 PermissionError |

---

# 핵심 명령어 정리

## DRfC 명령어가 안 될 때

```sh
bash
cd ~/Documents/GitHub/deepracer-for-cloud
source bin/activate.sh
```

---

## 현재 설정 확인

```sh
dr-summary
```

---

## 학습 파일 업로드

```sh
dr-upload-custom-files
```

---

## 학습 시작

```sh
dr-start-training
```

기존 모델을 지우고 다시 시작:

```sh
dr-start-training -w
```

---

## 서비스 상태 확인

```sh
docker service ls
```

---

## 최근 학습 로그 확인

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

---

## Episode만 확인

```sh
docker service logs deepracer-0_rl_coach 2>&1 | grep "Training>"
```

---

## Episode 실시간 확인

```sh
docker service logs -f deepracer-0_rl_coach 2>&1 | grep --line-buffered "Training>"
```

`Ctrl + C`는 로그 확인만 종료하며 학습은 계속됩니다.

---

## 실제 학습 종료

```sh
dr-stop-training
```

---

## Viewer 시작

```sh
dr-start-viewer
```

---

## Viewer 종료

```sh
dr-stop-viewer
```

---

## 모델 위치 찾기

```sh
find ~/Documents/GitHub/deepracer-for-cloud/data/minio -name "model" -type d
```

---

# 학습 오류 발생 시 빠른 확인 순서

`dr-start-training` 실행 후 문제가 발생하면 다음 순서대로 확인합니다.

```sh
docker service ls
```

### `rl_coach`, `robomaker`, `s3_minio`가 모두 `1/1`

학습 로그를 확인합니다.

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

Episode가 증가한다면 **정상 학습 중**입니다.

---

### `rl_coach`가 `0/1`

로그를 확인합니다.

```sh
docker service logs deepracer-0_rl_coach --tail 100
```

다음 오류가 있다면:

```text
PermissionError: [Errno 13] Permission denied: '/tmp/sagemaker/...'
```

다음 순서로 해결합니다.

```sh
dr-stop-training

sudo mkdir -p /tmp/sagemaker
sudo rm -rf /tmp/sagemaker/*
sudo chown -R $(id -u):$(id -g) /tmp/sagemaker

dr-start-training -w
```

그래도 같은 권한 오류가 발생한다면:

```sh
sudo chmod 777 /tmp/sagemaker
dr-start-training -w
```

다시 확인합니다.

```sh
docker service ls
docker service logs deepracer-0_rl_coach --tail 100
```

최종적으로:

```text
deepracer-0_rl_coach      1/1
deepracer-0_robomaker     1/1
s3_minio                  1/1
```

이고:

```text
Training> ... Episode=...
```

가 계속 증가한다면 정상적으로 학습 중입니다.

> ## 최종 판단 기준
>
> macOS + Docker Desktop 환경에서는
>
> ```text
> ERROR: Colima is not running.
> Sagemaker is not running.
> ```
>
> 등의 메시지가 출력될 수 있습니다.
>
> **이 문구 자체보다 `docker service ls`의 서비스 상태와 `rl_coach`의 Episode 로그를 기준으로 학습 성공 여부를 판단합니다.**