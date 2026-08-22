# 1주차 과제 — ROS2 / GIT / Docker

## 1. Linux Ubuntu 24.04 설치 및 세팅

LG그램(16ZD95P) 노트북에 Windows와 듀얼부팅으로 Ubuntu 24.04 LTS를 설치했다.

- Windows 디스크 관리에서 볼륨 축소로 40GB 파티션 확보
- BIOS(F2)에서 Secure Boot 비활성화
- Rufus로 부팅 USB 생성 후 "Install Ubuntu alongside Windows Boot Manager"로 설치 (자동 파티셔닝)
- 설치 후 Chrome, Git, VSCode, 한글 입력기(ibus-hangul) 세팅 완료

## 2. ROS2

### 2-1. ROS2가 뭔지, 왜 사용하는지

로봇 하나에는 카메라, 라이다, 모터, 센서 등 수많은 부품이 있고, 이걸 제어하는 프로그램들도 각각 따로 동작한다. 이 프로그램들끼리 실시간으로 데이터를 주고받아야 로봇이 하나처럼 움직이는데, ROS2는 이 통신 구조를 표준화해서 제공하는 프레임워크다.

통신을 주고받는 독립된 프로그램 단위를 **노드(Node)**라고 하며, 노드들은 아래 세 가지 방식으로 통신한다.

- **토픽(Topic)**: 지속적, 일방향 데이터 스트림. 응답 없이 계속 발행됨 (예: 센서 데이터).
- **서비스(Service)**: 요청 한 번에 응답 한 번 (함수 호출과 유사).
- **액션(Action)**: 시간이 걸리는 작업에 사용. 중간 진행상황 피드백 + 최종 결과.

### 2-2. ROS2 Jazzy 설치

Ubuntu 24.04에는 ROS2 Jazzy Jalisco가 대응된다. 공식 apt 저장소를 등록한 뒤 설치했다.

```bash
sudo apt install ros-jazzy-desktop -y
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
```

### 2-3. 워크스페이스와 패키지 구조

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
```

- `src/`: 패키지 코드 위치
- `build/`, `install/`, `log/`: 빌드 시 자동 생성
- 패키지는 `package.xml`(의존성) + `setup.py`/`CMakeLists.txt`(빌드 설정) + 노드 코드로 구성

### 2-4. 토픽·서비스·액션 실습 (turtlesim)

```bash
sudo apt install ros-jazzy-turtlesim -y
ros2 run turtlesim turtlesim_node
ros2 run turtlesim turtle_teleop_key
```

- **토픽**: teleop 노드가 `/turtle1/cmd_vel`에 속도 명령을 지속 발행 → turtlesim이 구독하여 거북이 이동
- **서비스**: `/spawn` 호출로 새 거북이 생성, 요청-응답 1회로 종료
```bash
  ros2 service call /spawn turtlesim/srv/Spawn "{x: 3, y: 3, theta: 0, name: 'turtle2'}"
```
- **remap**: 동일한 teleop 프로그램으로 두 번째 거북이를 조종하기 위해 토픽 이름 재매핑
```bash
  ros2 run turtlesim turtle_teleop_key --ros-args --remap turtle1/cmd_vel:=turtle2/cmd_vel
```
- **액션**: `/rotate_absolute`로 회전 명령, 중간 진행상황 피드백 후 최종 결과 수신
```bash
  ros2 action send_goal /turtle2/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 1.57}"
```

### 2-5. 가제보 실습

Ubuntu 24.04 + Jazzy 조합에는 Gazebo Harmonic이 대응된다.

```bash
sudo apt install ros-jazzy-ros-gz -y
gz sim
```

Empty 월드에서 오브젝트를 추가하고 물리 시뮬레이션(중력, 낙하)이 동작하는 것을 확인했다.

## 3. GIT

### 3-1. GIT이 무엇이고 왜 사용하는지

Git은 파일의 변경 이력을 관리하는 버전 관리 도구다. 모든 변경을 스냅샷(commit)으로 기록해 과거 시점으로 되돌아갈 수 있고, 브랜치로 작업을 분리하거나 여러 사람과 협업할 수 있게 해준다.

### 3-2. SSH 키 생성 및 add·commit·pull·push 실습

```bash
ssh-keygen -t ed25519 -C "이메일"
cat ~/.ssh/id_ed25519.pub   # GitHub Settings → SSH and GPG keys에 등록
ssh -T git@github.com        # 연결 테스트
```

로컬 저장소 연결은 SSH 방식으로 설정 (기존 HTTPS clone 시 비밀번호 요구 문제 해결):
```bash
git remote set-url origin git@github.com:아이디/저장소.git
```

커밋 작성자 정보 설정 (없으면 commit이 조용히 실패함):
```bash
git config --global user.name "이름"
git config --global user.email "이메일"
```

기본 작업 흐름:
```bash
git add week1.md
git commit -m "커밋 메시지"
git push
```

### 3-3. Branch, Stash, Worktree 실습

**Branch**: main에서 갈라져 나온 독립적인 작업 공간. 한쪽 브랜치의 커밋은 다른 브랜치에 영향을 주지 않는다.
```bash
git checkout -b test    # 새 브랜치 생성 및 이동
git diff main test       # 두 브랜치 간 차이 비교
```

**Stash**: 커밋하기 애매한 미완성 변경사항을 임시로 보관. add 여부와 무관하게 전체 변경사항을 보관함에 넣는다.
```bash
git stash        # 임시 보관 (작업 폴더가 깨끗한 상태로 복원됨)
git stash pop     # 다시 꺼내 적용
```

**Worktree**: 하나의 저장소(커밋 이력)를 공유하면서, 브랜치별로 별도의 작업 폴더를 만들어 동시에 열어둘 수 있는 기능. 같은 브랜치는 동시에 하나의 worktree에서만 체크아웃 가능하다.
```bash
git worktree add ../week1-main-copy main   # main 브랜치용 별도 폴더 생성
git worktree list                           # 목록 확인
git worktree remove ../week1-main-copy      # 삭제
```

디버깅에 유용한 명령어: `git log --oneline --all --graph`로 전체 브랜치 흐름을 한눈에 확인 가능.

## 4. Docker

### 4-1. 도커 사용 이유와 시스템 동작

Docker는 프로그램과 그 실행 환경(라이브러리, 설정 등)을 하나의 격리된 상자(컨테이너)에 담아 어디서든 동일하게 실행되게 해주는 도구다. 가상머신과 달리 호스트 커널을 공유해 가볍고 빠르다.

- **이미지(Image)**: 실행 환경의 설계도, 읽기 전용
- **컨테이너(Container)**: 이미지를 실제로 실행한 상태

```bash
sudo apt install docker.io -y
sudo usermod -aG docker $USER   # sudo 없이 실행하기 위한 그룹 등록 (재로그인 필요)
docker run hello-world           # 설치 확인
```

### 4-2. 이미지 및 컨테이너 생성 실습

```bash
docker run -it ubuntu bash       # 이미지로 컨테이너 생성 및 진입
```

컨테이너 안은 호스트와 완전히 격리된 별도의 리눅스 환경임을 확인 (`cat /etc/os-release` 등).

### 4-3. 주요 명령어 실습

**컨테이너/이미지 목록 및 삭제**
```bash
docker ps -a        # 전체 컨테이너 목록 (멈춘 것 포함)
docker images        # 이미지 목록
docker rm [컨테이너ID]   # 컨테이너 삭제
docker rmi [이미지명]    # 이미지 삭제
```

**exec — 실행 중인 컨테이너에 접속**
```bash
docker run -d --name test-container ubuntu sleep 300   # 백그라운드로 컨테이너 유지
docker exec -it test-container bash                      # 접속
```

**commit — 컨테이너 상태를 새 이미지로 저장**
```bash
docker commit test-container my-custom-image
```
컨테이너 안에서 만든 변경사항이 새 이미지에 반영되어, 이후 이 이미지로 생성한 컨테이너에는 해당 변경사항이 그대로 포함됨을 확인했다.

**compose — 여러 컨테이너를 설정 파일로 동시 관리**

`docker-compose.yml`:
```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  redis:
    image: redis
```
```bash
docker-compose up -d
```
nginx, redis 두 컨테이너가 동시에 실행되고, 포트 매핑(8080→80)을 통해 브라우저에서 nginx 기본 페이지 접속을 확인했다.

**Docker Hub**: 이미지를 온라인에서 주고받는 저장소 (GitHub과 유사한 역할). 오늘은 `hello-world`, `ubuntu`, `nginx`, `redis` 등 공개 이미지를 pull하는 용도로만 사용했다.

