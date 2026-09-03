🛠️ macOS + Docker Desktop 환경에서 Sagemaker is not running 해결

🚨 에러 상황

dr-start-training -w 실행 시 다음과 같이 서비스는 생성되지만 마지막에 SageMaker가 실행되지 않았다는 메시지가 출력될 수 있다.

dr-start-training -w

예시:

Creating service deepracer-0_rl_coach
Creating service deepracer-0_robomaker
Waiting up to 15 seconds for Sagemaker to start up...
Sagemaker is not running.

서비스 상태를 확인한다.

docker service ls

문제가 발생한 경우 다음처럼 rl_coach만 0/1 상태로 종료될 수 있다.

NAME                    REPLICAS
deepracer-0_rl_coach    0/1
deepracer-0_robomaker   1/1
s3_minio                1/1

⸻

🔍 원인 확인

먼저 rl_coach 로그를 확인한다.

docker service logs deepracer-0_rl_coach --tail 100

다음과 같은 에러가 발생했다면:

PermissionError: [Errno 13] Permission denied: '/tmp/sagemaker/tmp...'

SageMaker Local이 임시 작업 디렉터리로 사용하는 /tmp/sagemaker에 쓰기 권한이 없어 rl_coach 컨테이너가 종료된 것이다.

즉 문제의 핵심은 Docker Desktop이나 ARM64 이미지 자체가 아니라 /tmp/sagemaker 디렉터리의 권한 문제이다.

⸻

💡 해결 방법

먼저 실행 중인 학습 서비스를 중지한다.



/tmp/sagemaker 디렉터리를 생성한다.

sudo mkdir -p /tmp/sagemaker

쓰기 권한을 부여한다.

sudo chmod 777 /tmp/sagemaker

권한을 확인한다.

ls -ld /tmp/sagemaker

정상 예시:

drwxrwxrwx  2 root  wheel  ... /tmp/sagemaker

기존 임시 파일이 남아 있다면 삭제한다.

sudo rm -rf /tmp/sagemaker/*

이 명령은 /tmp/sagemaker 내부의 임시 파일만 삭제하며 MinIO에 저장된 모델 파일을 삭제하는 명령은 아니다.

그다음 다시 학습을 시작한다.

dr-start-training -w

⸻

✅ 정상 동작 확인

서비스 상태를 확인한다.

docker service ls

정상이라면 다음처럼 모든 서비스가 1/1 상태가 된다.

NAME                    REPLICAS
deepracer-0_rl_coach    1/1
deepracer-0_robomaker   1/1
s3_minio                1/1

이 상태라면 SageMaker, RoboMaker, MinIO 서비스가 모두 정상 실행 중인 것이다.

⸻

📋 실제 학습 진행 여부 확인

rl_coach 로그를 확인한다.

docker service logs deepracer-0_rl_coach --tail 100

다음과 같이 Episode가 증가하면 실제 학습이 정상적으로 진행 중인 것이다.

Training> Name=main_level/agent, Worker=0, Episode=1, Total reward=48.65, Steps=102, Training iteration=0
Training> Name=main_level/agent, Worker=0, Episode=2, Total reward=58.19, Steps=194, Training iteration=0
Training> Name=main_level/agent, Worker=0, Episode=3, Total reward=23.37, Steps=240, Training iteration=0
...
Training> Name=main_level/agent, Worker=0, Episode=20, Total reward=28.07, Steps=880, Training iteration=0

Episode가 일정 개수 누적되면 정책 학습도 수행된다.

Policy training> Surrogate loss=...
Policy training> KL divergence=...
Policy training> Entropy=...

체크포인트가 저장되는 로그도 확인할 수 있다.

Checkpoint> Saving in path=['./checkpoint_sagemaker/agent/1_Step-880.ckpt']
Uploaded 3 files for checkpoint 1

모델 파일도 MinIO에 저장된다.

saved intermediate frozen graph: rl-deepracer-sagemaker/model/model_1.pb

즉 다음 세 가지가 보이면 학습이 정상적으로 진행 중인 것이다.

Training> Episode=...
Policy training> ...
Checkpoint> Saving...

⸻

⚠️ 처음 학습할 때 Unable to find deepracer checkpoint json이 출력되는 경우

새 모델을 처음 생성하는 과정에서 다음과 같은 메시지가 출력될 수 있다.

Unable to find deepracer checkpoint json
Unable to find the best deepracer checkpoint number.
Unable to find the last deepracer checkpoint number.

처음에는 기존 체크포인트가 존재하지 않기 때문에 발생할 수 있는 메시지이다.

이후 다음과 같이 체크포인트가 정상 생성되면 문제가 아니다.

Checkpoint> Saving in path=['./checkpoint_sagemaker/agent/1_Step-880.ckpt']
Uploaded 3 files for checkpoint 1

그리고 다음과 같이 기존 체크포인트 정보를 다시 읽어오는 로그가 나타나면 정상이다.

Successfully downloaded deepracer checkpoint json
Best checkpoint number: 0, Last checkpoint number: 0

⸻

⚠️ ERROR: Colima is not running 메시지가 계속 출력되는 경우

Docker Desktop을 사용하고 있는데도 다음 메시지가 출력될 수 있다.

ERROR: Colima is not running. Start it with: colima start

예를 들어 다음과 같이 반복적으로 출력될 수 있다.

ERROR: Colima is not running. Start it with: colima start
Waiting up to 15 seconds for Sagemaker to start up...
ERROR: Colima is not running. Start it with: colima start
ERROR: Colima is not running. Start it with: colima start
...
Sagemaker is not running.

이는 macOS용 DRfC 스크립트 내부에 Colima 상태를 확인하는 로직이 포함되어 있기 때문이다.

Docker Desktop을 사용하고 있는 경우에는 이 메시지만으로 실제 학습 실패 여부를 판단하면 안 된다.

반드시 다음 명령어로 실제 Docker 서비스 상태를 확인한다.

docker service ls

다음처럼 모두 1/1이라면:

deepracer-0_rl_coach      1/1
deepracer-0_robomaker     1/1
s3_minio                  1/1

실제 서비스는 정상 실행 중인 것이다.

그리고 다음 명령어로 학습 로그를 확인한다.

docker service logs deepracer-0_rl_coach --tail 100

다음처럼 Episode가 증가하고 있다면:

Training> Name=main_level/agent, Worker=0, Episode=1, ...
Training> Name=main_level/agent, Worker=0, Episode=2, ...
Training> Name=main_level/agent, Worker=0, Episode=3, ...

Sagemaker is not running이라는 문구가 출력되었더라도 실제 학습은 정상적으로 진행되고 있는 상태이다.

즉 Docker Desktop 환경에서는 다음 메시지보다:

Sagemaker is not running.

실제 Docker 상태와 학습 로그를 기준으로 판단해야 한다.

⸻

🧱 Apple Silicon 아키텍처 확인

Apple Silicon Mac에서는 DeepRacer 이미지의 아키텍처를 확인할 수 있다.

docker image inspect awsdeepracercommunity/deepracer-simapp:6.0.4-cpu --format '{{.Architecture}}'

정상 예시:

arm64

Docker Desktop 환경의 아키텍처도 확인한다.

docker info --format '{{.Architecture}}'

예시:

aarch64

두 환경 모두 ARM64 계열로 정상 동작할 수 있다.

따라서 rl_coach가 0/1이면서 로그에 다음과 같은 에러가 있다면:

PermissionError: [Errno 13] Permission denied: '/tmp/sagemaker/...'

아키텍처 문제가 아니라 /tmp/sagemaker 권한 문제를 먼저 해결해야 한다.

⸻

📌 학습 진행 상황 확인 명령어

전체 로그 확인:

docker service logs deepracer-0_rl_coach

최근 로그만 확인:

docker service logs deepracer-0_rl_coach --tail 200

Episode 로그만 확인:

docker service logs deepracer-0_rl_coach 2>&1 | grep "Training>"

실시간으로 Episode만 확인:

docker service logs -f deepracer-0_rl_coach 2>&1 | grep --line-buffered "Training>"

실시간 로그를 보고 있을 때:

Ctrl + C

를 눌러도 로그 확인만 종료된다.

학습 자체는 Docker 서비스에서 계속 실행된다.

⸻

🛑 실제 학습 중지

실제 학습을 종료하려면 다음 명령어를 사용한다.

dr-stop-training

이 명령을 실행해야 rl_coach와 robomaker 서비스가 종료된다.

⸻

✅ 최종 해결 방법 요약

macOS Apple Silicon + Docker Desktop 환경에서 다음 오류가 발생한다면:

Sagemaker is not running.

먼저 서비스 상태를 확인한다.

docker service ls

rl_coach가 0/1이라면 로그를 확인한다.

docker service logs deepracer-0_rl_coach --tail 100

다음 에러가 보이는 경우:

PermissionError: [Errno 13] Permission denied: '/tmp/sagemaker/...'

아래 순서로 해결한다.

dr-stop-training
sudo mkdir -p /tmp/sagemaker
sudo chmod 777 /tmp/sagemaker
sudo rm -rf /tmp/sagemaker/*
dr-start-training -w

이후 다시 확인한다.

docker service ls

정상 상태:

deepracer-0_rl_coach      1/1
deepracer-0_robomaker     1/1
s3_minio                  1/1

마지막으로 학습 로그를 확인한다.

docker service logs deepracer-0_rl_coach --tail 100

다음처럼 Episode가 계속 증가한다면 정상적으로 학습 중이다.

Training> Name=main_level/agent, Worker=0, Episode=1, ...
Training> Name=main_level/agent, Worker=0, Episode=2, ...
Training> Name=main_level/agent, Worker=0, Episode=3, ...

Docker Desktop 환경에서는 ERROR: Colima is not running 또는 Sagemaker is not running 메시지가 출력되더라도 실제 Docker 서비스가 모두 1/1이고 Episode 로그가 증가한다면 학습은 정상적으로 진행되고 있는 것이다.