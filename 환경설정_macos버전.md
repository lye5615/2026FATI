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
> DeepRacer 환경설정 및 학습을 진행하기 전에
> **Docker Desktop이 실행되어 있어야 합니다.**

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
2. DRfC 초기 설정 (`init.sh`)
3. DRfC 명령어 활성화
4. 학습 데이터 업로드
5. 학습 시작
6. 학습 종료
7. 학습된 모델 확인
8. Viewer 확인

> **중요**
>
> Repository를 처음 clone한 경우에는 반드시 **`init.sh`를 먼저 실행**해야 합니다.
>
> `init.sh`를 실행하기 전에 `source bin/activate.sh` 또는 `dr-*` 명령어부터 실행하지 않습니다.

---

# 1. DRfC Repository 복제

DeepRacer for Cloud Repository를 복제합니다.

```sh
git clone https://github.com/aws-deepracer-community/deepracer-for-cloud
```

복제한 폴더로 이동합니다.

```sh
cd deepracer-for-cloud
```

파일이 정상적으로 받아졌는지 확인합니다.

```sh
ls
```

---

# 2. DRfC 초기 설정

## Docker 실행 확인

먼저 Docker Desktop이 실행되어 있는지 확인합니다.

```sh
docker ps
```

오류가 발생하지 않으면 정상입니다.

---

## Docker Buildx 확인

```sh
docker buildx version
```

정상적으로 버전이 출력되면 다음 단계로 넘어갑니다.

---

## init.sh 실행

> **처음 환경을 구축할 때 반드시 이 과정을 먼저 진행합니다.**

Apple Silicon(M1/M2/M3/M4) MacBook에서는 GPU가 아닌 **CPU 환경**으로 설정합니다.

`deepracer-for-cloud` 디렉터리에서 실행합니다.

```sh
cd ~/deepracer-for-cloud
```

초기 설정을 실행합니다.

```sh
sudo bin/init.sh -a cpu -c local
```

여기서:

- `-a cpu` : CPU를 이용하여 학습
- `-c local` : 로컬 저장소(MinIO)를 사용

초기 실행 시 필요한 Docker Image 및 학습 환경을 준비하기 때문에 시간이 오래 걸릴 수 있습니다.

**반드시 `init.sh`가 완료된 후 다음 단계로 넘어갑니다.**

> **MacBook에서는 다음과 같은 Ubuntu용 명령어를 실행할 필요가 없습니다.**
>
> ```text
> sudo service docker start
> sudo reboot -f
> ```
>
> Docker Desktop이 실행되어 있으면 됩니다.

---

# 3. DRfC 명령어 활성화

> `init.sh`를 정상적으로 완료한 후 진행합니다.

DeepRacer 관련 명령어는 기본적으로 `deepracer-for-cloud` 디렉터리에서 실행합니다.

먼저 폴더로 이동합니다.

```sh
cd ~/deepracer-for-cloud
```

DRfC 명령어를 활성화합니다.

```sh
source bin/activate.sh
```

이제 `dr-`로 시작하는 DeepRacer 관련 명령어를 사용할 수 있습니다.

예:

```sh
dr-summary
```

---

## `dr-*` 명령어가 실행되지 않는 경우

`dr-start-training`, `dr-update-env`, `dr-summary` 등의 명령어 실행 시

```text
command not found
```

가 발생한다면 다음 명령어를 다시 실행합니다.

```sh
cd ~/deepracer-for-cloud
source bin/activate.sh
```

> **단, Repository를 처음 clone한 직후라면**
> `source bin/activate.sh`보다 먼저 `init.sh`가 완료되어 있어야 합니다.

필요한 설정을 업데이트합니다.

```sh
dr-update
dr-update-env
```

---

# 4. 학습 데이터 업로드

학습에 필요한 설정 파일과 Reward Function 등의 파일을 로컬 MinIO 저장소에 업로드합니다.

```sh
dr-upload-custom-files
```

Reward Function 또는 학습 설정 파일을 수정했다면 학습 시작 전에 다시 실행합니다.

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
cd ~/deepracer-for-cloud/data/minio/bucket/rl-deepracer-sagemaker
```

모델 폴더를 확인합니다.

```sh
ls model
```

모델 파일이 정상적으로 생성되어 있다면 학습 결과가 저장된 것입니다.

## 경로가 없는 경우

위 경로가 존재하지 않는다면 다음 명령어로 `model` 폴더를 찾습니다.

```sh
find ~/deepracer-for-cloud/data/minio -name "model" -type d
```

학습 설정이나 모델 이름에 따라 실제 저장 경로가 달라질 수 있으므로 출력된 경로를 사용하면 됩니다.

---

# 8. 학습 진행 상황 Viewer 확인

학습이 진행되는 동안 Viewer를 통해 시뮬레이션 화면을 확인할 수 있습니다.

학습 로그가 출력되고 있는 터미널은 그대로 두고 **새 Terminal 탭 또는 창**을 엽니다.

새 터미널에서 프로젝트 폴더로 이동합니다.

```sh
cd ~/deepracer-for-cloud
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
cd ~/deepracer-for-cloud
source bin/activate.sh
dr-upload-custom-files
```

그다음 다시 학습을 시작합니다.

```sh
dr-start-training
```

> 이미 최초 환경설정에서 `init.sh`를 완료했다면 Reward Function을 수정할 때마다 `init.sh`를 다시 실행할 필요는 없습니다.

---

# 오랜만에 MacBook을 켠 뒤 다시 학습하는 방법

이미 최초 환경설정에서 `init.sh`까지 정상적으로 완료했다면 컴퓨터를 다시 켤 때마다 `init.sh`를 실행할 필요는 없습니다.

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

## 3. DeepRacer 폴더로 이동

```sh
cd ~/deepracer-for-cloud
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

즉, **최초 환경설정을 이미 완료한 MacBook**이라면 일반적으로 다음 순서로 다시 시작하면 됩니다.

```sh
open -a Docker

cd ~/deepracer-for-cloud
source bin/activate.sh

dr-upload-custom-files
dr-start-training
```

> `init.sh`는 **최초 환경설정 시 먼저 실행하는 명령어**이며,
> 정상적으로 환경설정이 끝난 이후 매번 학습할 때마다 실행하는 명령어는 아닙니다.

---

# 에러 슈팅

## `source bin/activate.sh` 실행 시 `no configuration file provided: not found`

### 에러 내용

`source bin/activate.sh` 실행 과정에서 다음과 같은 오류가 발생할 수 있습니다.

```text
no configuration file provided: not found
```

### 원인

Repository를 clone한 후 **DRfC 초기 설정(`init.sh`)을 완료하지 않은 상태에서 `source bin/activate.sh`를 실행한 경우** 발생할 수 있습니다.

### 해결

먼저 `deepracer-for-cloud` 디렉터리로 이동합니다.

```sh
cd ~/deepracer-for-cloud
```

Docker Desktop이 실행되어 있는지 확인합니다.

```sh
docker ps
```

그다음 DRfC 초기 설정을 먼저 실행합니다.

```sh
sudo bin/init.sh -a cpu -c local
```

초기 설정이 정상적으로 완료된 후 다시 실행합니다.

```sh
source bin/activate.sh
```

> **최초 환경설정 순서**
>
> ```text
> Repository clone
>       ↓
> init.sh
>       ↓
> source bin/activate.sh
>       ↓
> dr-* 명령어 사용
> ```

---

# MacBook에서 Ubuntu 문서와 다른 점

| Ubuntu 문서 | MacBook |
|---|---|
| `su`로 root 전환 | 필요 없음 |
| `sudo passwd root` | 필요 없음 |
| `sudo service docker start` | 필요 없음 |
| Ubuntu 터미널 사용 | macOS Terminal 사용 |
| Docker daemon 직접 실행 | Docker Desktop 실행 |
| `/data/minio/...` | `~/deepracer-for-cloud/data/minio/...` |
| Linux 패키지 관리자 `apt` | macOS 패키지 관리자 `brew` |

---

# 핵심 명령어 정리

## 최초 환경설정

```sh
cd ~/deepracer-for-cloud
sudo bin/init.sh -a cpu -c local
source bin/activate.sh
```

> **최초에는 `init.sh` → `source bin/activate.sh` 순서를 지킵니다.**

## DRfC 명령어가 안 될 때

이미 `init.sh`를 완료한 환경이라면:

```sh
cd ~/deepracer-for-cloud
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
find ~/deepracer-for-cloud/data/minio -name "model" -type d
```