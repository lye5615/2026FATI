# FATI 환경설정

# 환경설정 (MacBook)

## 순서

1. macOS 개발 환경 준비
2. Homebrew 설치
3. 필수 프로그램 설치
4. DeepRacer 로컬 학습환경 구축

---

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
DRfC 스크립트(`bin/activate.sh`)는 **bash 4 이상**에서만 동작하므로 최신 bash를 설치합니다.

```sh
brew install bash
```

설치 확인:

```sh
/opt/homebrew/bin/bash --version
```

`version 5.x` 이상이 출력되면 정상입니다.
(Intel Mac이라면 경로가 `/usr/local/bin/bash` 입니다.)

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
> MacBook에서는 Docker를 사용하기 전에 **Docker Desktop이 실행되어 있어야 합니다.**

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
4. 학습 데이터 업로드
5. 학습 시작
6. 학습 종료
7. 학습된 모델 확인
8. Viewer 확인

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
> 다른 위치에 복제했다면 그 경로로 바꿔서 읽으면 됩니다.
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

> **중요: `sudo`를 붙이지 마세요.**
>
> `init.sh`는 관리자 권한이 필요한 부분에서 스스로 `sudo`를 호출합니다.
> 전체를 `sudo bin/init.sh ...`로 실행하면
>
> - `data/`, `tmp/`, `custom_files/`, `system.env` 등이 root 소유가 되고
> - MinIO 접속 정보가 내 홈이 아닌 `/var/root/.aws`에 만들어져
>
> 이후 `source bin/activate.sh`에서 다음 오류가 납니다.
>
> ```text
> ERROR: AWS credentials not found in profile 'minio'.
> ```
>
> 이미 `sudo`로 실행했다면 아래 명령으로 되돌린 뒤 다시 진행합니다.
>
> ```sh
> cd ~/Documents/GitHub/deepracer-for-cloud
> sudo chown -R $(id -un):staff .
> ln -sfn ~/.aws docker/volumes/.aws
> bin/init.sh -a cpu -c local
> ```

초기 실행 시 필요한 Docker Image를 다운로드하기 때문에 시간이 오래 걸릴 수 있습니다.

## /tmp/sagemaker 권한 설정

`init.sh`는 학습용 임시 폴더 `/tmp/sagemaker`를 root 권한으로 만듭니다.
Docker Desktop은 공유 폴더에 대한 컨테이너 쓰기를 **호스트 사용자 권한**으로 처리하기 때문에,
이 폴더 소유자를 내 계정으로 바꿔주지 않으면 학습 시작 시 실패합니다.

```sh
sudo chown -R $(id -u):$(id -g) /tmp/sagemaker
```

이 과정을 건너뛰면 `dr-start-training`에서 `Sagemaker is not running.`이 뜨고,
컨테이너 로그에 다음 오류가 남습니다.

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

> **중요**
>
> 터미널을 새로 열면 기본 셸이 zsh입니다.
> `source bin/activate.sh`를 실행하기 전에 반드시 **bash로 먼저 진입**해야 합니다.
> zsh나 기본 bash(3.2)에서 실행하면 다음과 같은 오류가 발생합니다.
>
> ```text
> No configuration file.
> bad substitution
> ```

bash로 진입합니다.

```sh
bash
```

프롬프트가 `bash-5.3$` 형태로 바뀌면 정상입니다.

폴더로 이동합니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
```

DRfC 명령어를 활성화합니다.

```sh
source bin/activate.sh
```

> ## 대원칙
>
> `dr-start-training`, `dr-update-env`, `dr-summary` 등 `dr-`로 시작하는 명령어가
>
> ```text
> command not found
> ```
>
> 와 같이 실행되지 않는다면 가장 먼저 아래 명령어를 다시 실행합니다.
>
> ```sh
> bash
> cd ~/Documents/GitHub/deepracer-for-cloud
> source bin/activate.sh
> ```

필요한 설정을 업데이트합니다.

```sh
dr-update
dr-update-env
```

## 트랙 이름 변경

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

저장한 뒤 설정을 반영합니다.

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

> `dr-update-env`, `dr-summary` 등의 명령어가 `command not found`로 실행되지 않는다면
> 먼저 아래 명령어로 DRfC 환경을 다시 활성화합니다.
>
> ```sh
> bash
> cd ~/Documents/GitHub/deepracer-for-cloud
> source bin/activate.sh
> ```

---

# 4. 학습 데이터 업로드

학습에 필요한 설정 파일과 Reward Function 등의 파일을 로컬 MinIO 저장소에 업로드합니다.

```sh
dr-upload-custom-files
```

Reward Function을 수정했다면 학습을 시작하기 전에 다시 실행해야 합니다.

```sh
dr-upload-custom-files
```

---

# 5. 학습 시작

DeepRacer 모델 학습을 시작합니다.

```sh
dr-start-training
```

학습이 정상적으로 시작되면 터미널에 로그가 계속 출력됩니다.

예를 들어 다음과 같은 내용이 나타납니다.

```text
Episode...
Reward...
Training...
```

처음 실행하는 경우 Docker Image와 학습 환경을 준비하는 과정 때문에 시간이 오래 걸릴 수 있습니다.

## 이미 존재하는 모델 경로 (`-w` 옵션)

이전 학습이 실패했거나 같은 이름으로 다시 학습하면 다음 메시지가 나옵니다.

```text
Selected path s3://bucket/rl-deepracer-sagemaker exists. Delete it, or use -w option. Exiting.
```

기존 학습 결과를 덮어쓰지 않으려는 안전장치입니다.
버릴 결과라면 `-w`(wipe)를 붙여 해당 경로를 지우고 새로 시작합니다.

```sh
dr-start-training -w
```

> **주의**
>
> `-w`는 해당 모델 경로를 **통째로 삭제**합니다.
> 남겨야 할 학습 결과가 있다면 대신 `run.env`의 `DR_LOCAL_S3_MODEL_PREFIX`를
> 새 이름으로 바꾼 뒤 `dr-update-env`를 실행하세요.

---

# 6. 학습 종료

학습을 종료하려면 먼저 학습 로그가 출력되고 있는 터미널에서

```text
Ctrl + C
```

를 누릅니다.

그다음 반드시 아래 명령어를 실행합니다.

```sh
dr-stop-training
```

> `Ctrl + C`만 누르고 끝내지 않는 것을 권장합니다.
>
> `dr-stop-training`을 실행하지 않으면 학습에 사용된 Docker 컨테이너가 계속 남아 있을 수 있습니다.

---

# 7. 학습된 모델 확인

학습 결과는 로컬 MinIO 저장소에 저장됩니다.

기본 경로에서 확인합니다.

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

학습 설정이나 모델 이름에 따라 실제 저장 경로가 달라질 수 있으므로 출력된 경로를 사용하면 됩니다.

---

# 8. 학습 진행 상황 Viewer 확인

학습이 진행되는 동안 Viewer를 통해 시뮬레이션 화면을 확인할 수 있습니다.

학습 로그가 출력되고 있는 터미널은 그대로 두고 **새 Terminal 탭 또는 창**을 엽니다.

새 터미널에서 bash로 진입한 뒤 프로젝트 폴더로 이동합니다.

```sh
bash
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

Viewer를 종료하려면 다음 명령어를 사용합니다.

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

그다음 다시 학습을 시작합니다.

```sh
dr-start-training
```

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

## 3. bash 진입 후 DeepRacer 폴더로 이동

```sh
bash
cd ~/Documents/GitHub/deepracer-for-cloud
```

## 4. DRfC 명령어 활성화

```sh
source bin/activate.sh
```

## 5. 필요한 파일 업로드

Reward Function이나 설정 파일을 수정했다면 다시 업로드합니다.

```sh
dr-upload-custom-files
```

수정한 것이 없다면 상황에 따라 생략할 수 있습니다.

## 6. 학습 시작

```sh
dr-start-training
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

## `No configuration file.` / `bad substitution`

```text
grep: /Users/.../system.env: No such file or directory
No configuration file.
```
```text
bin/activate.sh: line 116: ${DR_DOCKER_STYLE,,}: bad substitution
```

zsh 또는 macOS 기본 bash(3.2)에서 `source bin/activate.sh`를 실행한 경우입니다.
`brew install bash` 후 `bash`로 진입한 뒤 다시 실행하세요.

---

## `ERROR: AWS credentials not found in profile 'minio'.`

`bin/init.sh`를 `sudo`로 실행해서 MinIO 접속 정보가 `/var/root/.aws`에 만들어진 경우입니다.

```sh
cd ~/Documents/GitHub/deepracer-for-cloud
sudo chown -R $(id -un):staff .
ln -sfn ~/.aws docker/volumes/.aws
bin/init.sh -a cpu -c local
```

---

## `Connection was closed ... http://localhost:9000/...`

```text
fatal error: Connection was closed before we received a valid response
from endpoint URL: "http://localhost:9000/bucket?list-type=2&..."
```

MinIO 컨테이너가 떠 있지 않은 것입니다. 상태를 확인합니다.

```sh
docker service ls          # s3_minio 가 0/1 이면 실패한 상태
docker logs $(docker ps -a --filter name=s3_minio --format '{{.Names}}' | head -1)
```

로그에 `Unable to initialize backend: file access denied`가 보이면
`data/` 폴더가 root 소유라 MinIO가 쓰지 못하는 것입니다. 위의 `chown`을 실행한 뒤:

```sh
docker stack rm s3
source bin/activate.sh
```

---

## `Sagemaker is not running.`

`/tmp/sagemaker`가 root 소유일 때 발생합니다.

```sh
sudo chown -R $(id -u):$(id -g) /tmp/sagemaker
dr-stop-training
source bin/activate.sh
dr-start-training
```

---

## `Selected path ... exists. Delete it, or use -w option.`

이전 학습 결과가 남아 있는 것입니다. 버려도 되면 `dr-start-training -w`.
자세한 내용은 위 [5. 학습 시작] 항목을 참고하세요.

---

# 이 저장소에 적용한 소스 수정

DRfC 원본(upstream)의 macOS 지원은 **Colima** 환경을 전제로 작성되어 있어,
Docker Desktop 환경에서는 아래 수정이 필요합니다.
GitHub에서 새로 clone 받은 경우 같은 수정을 다시 적용해야 합니다.

| 파일 | 수정 내용 | 해결한 증상 |
|---|---|---|
| `bin/activate.sh` | 맨 위에 bash 4+ 검사 추가 | zsh/bash 3.2에서 엉뚱한 오류 대신 안내 문구 출력 |
| `bin/activate.sh` | `DR_DOCKER_MAJOR_VERSION`을 사용하는 위치보다 **앞에서** 정의하도록 순서 교정 | `[: : 정수 값이 필요함` |
| `bin/activate.sh`, `bin/module/summary.sh` | `grep -oP` → `grep -oE` (BSD grep은 `-P` 미지원) | `grep: invalid option -- P` |
| `bin/scripts_wrapper.sh` | `_dr_use_colima` 판별 함수 추가. docker context가 colima일 때만 `colima ssh` 경로를 쓰고, Docker Desktop이면 호스트 경로를 그대로 사용 | `ERROR: Colima is required on macOS.` |
| `bin/scripts_wrapper.sh` | Docker Desktop일 때 `/tmp/sagemaker`를 현재 사용자 소유로 생성 | `Sagemaker is not running.` |

---

# 핵심 명령어 정리

## DRfC 명령어가 안 될 때

```sh
bash
cd ~/Documents/GitHub/deepracer-for-cloud
source bin/activate.sh
```

## 학습 파일 업로드

```sh
dr-upload-custom-files
```

## 학습 시작

```sh
dr-start-training
```

## 학습 종료

```sh
dr-stop-training
```

## Viewer 시작

```sh
dr-start-viewer
```

## Viewer 종료

```sh
dr-stop-viewer
```

## 모델 위치 찾기

```sh
find ~/Documents/GitHub/deepracer-for-cloud/data/minio -name "model" -type d
```
