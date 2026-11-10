# AI-codyssey
🚀 [기술 문서] 개발 워크스테이션 구축 및 Docker/Git 검증 결과 보고서
1. 프로젝트 개요
목표: 로컬 개발 환경의 재현성을 확보하기 위해 터미널(CLI), Docker(컨테이너), Git/GitHub을 활용한 개발 워크스테이션 환경을 직접 구축하고 검증한다.
주요 학습 내용:
절대/상대 경로 및 파일 권한 제어(rwx, 600, 700)
Docker 엔진 구동 확인 및 컨테이너 생명주기 관리
Dockerfile을 이용한 커스텀 이미지 제작 및 포트 매핑
볼륨 및 바인드 마운트를 통한 데이터 영속성 검증
Git 버전 관리 및 GitHub 연동
2. 실행 환경
OS: Windows 11 (PowerShell / WSL2 Kernel)
Shell: UCRT64 / PowerShell
Docker Version: 29.9.2 (Desktop-linux, API 29.9)
Git Version: git version 2.56.0.windows.4
3. 수행 항목 체크리스트
[x] 터미널 기본 조작 및 폴더 구성 (mkdir, touch, ls)
[x] 권한 변경 실습 (chmod 600, chmod 700)
[x] Docker 설치 및 기본 점검 (docker info, docker --version)
[x] hello-world 컨테이너 실행 성공
[x] Ubuntu 컨테이너 진입 및 제어 (docker run -it --name mybox ubuntu)
[x] Dockerfile 기반 커스텀 이미지 제작 및 빌드
[x] 포트 매핑 및 브라우저/curl 접속 증거
[x] Docker 바인드 마운트 및 실시간 반영 검증
[x] Docker 볼륨 생성 및 데이터 영속성 검증
[x] Git 설정 및 GitHub 저장소 연동
4. 핵심 수행 로그 및 검증 결과
1) Docker 설치 및 기본 점검
PowerShell
PS C:\WINDOWS\system32> docker info
# 클라이언트 버전: 29.9.2, 서버 버전: 29.9.2, 운영체제: Docker Desktop (WSL2 기반) 정상 동작 확인 완료
2) 컨테이너 실행 및 파일/권한 제어 실습
PowerShell
# Ubuntu 컨테이너 생성 및 접속
PS C:\WINDOWS\system32> docker run -it --name mybox ubuntu

# 디렉토리 생성 및 파일 작성 실습
root@46f968e80102:/# mkdir temp
root@46f968e80102:/# cd temp
root@46f968e80102:/temp# touch file.txt
root@46f968e80102:/temp# mkdir mydir

# 초기 권한 확인 (기본값)
root@46f968e80102:/temp# ls -l
total 4
-rw-r--r-- 1 root root     0 Nov  9 09:59 file.txt
drwxr-xr-x 2 root root  4096 Nov  9 10:00 mydir

# 권한 변경 실습 (파일 600, 디렉토리 700)
root@46f968e80102:/temp# chmod 600 file.txt
root@46f968e80102:/temp# chmod 700 mydir

# 변경 후 권한 비교 확인 증거
root@46f968e80102:/temp# ls -l
total 4
-rw------- 1 root root     0 Nov  9 09:59 file.txt
drwx------ 2 root root  4096 Nov  9 10:00 mydir
3) 컨테이너 재시작 및 관리 로그
PowerShell
# 컨테이너 종료 후 상태 확인
PS C:\WINDOWS\system32> docker ps -a
CONTAINER ID   IMAGE         COMMAND         CREATED         STATUS                   PORTS     NAMES
46f968e80102   ubuntu        "/bin/bash"     2 minutes ago   Exited (0) 22 seconds ago          mybox

# 컨테이너 구동 및 exec를 통한 재접속
PS C:\WINDOWS\system32> docker start mybox
mybox
PS C:\WINDOWS\system32> docker exec -it mybox bash
root@46f968e80102:/# 
4) Dockerfile 커스텀 이미지 제작 및 포트 매핑 실습
PowerShell
# 1. Dockerfile 빌드 (초기 빈 파일 에러 해결 후 정상 빌드 완료)
PS C:\dockerfile with codyssey> docker build -t my-web .
[+] Building 14.2s (7/7) FINISHED                 docker:desktop-linux
 => [internal] load build definition from Dockerfile                0.0s
 => => transferring dockerfile: 111B                                0.0s
 => [internal] load metadata for docker.io/library/nginx:latest     2.3s
 => [1/2] FROM docker.io/library/nginx:latest@sha256:165ddde8751e   8.7s
 => [2/2] COPY index.html /usr/share/nginx/html/index.html          0.5s
 => exporting to image                                              2.5s
 => => naming to docker.io/library/my-web:latest                    0.0s

# 2. 포트 매핑을 통한 컨테이너 실행 및 확인
PS C:\dockerfile with codyssey> docker run -d -p 7566:80 --name my-web my-web
eb75cf470aebcb2c105c5d85f16fe59b6f5d9361c6ddadd8827bee366e4c0a2a

PS C:\dockerfile with codyssey> docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                      NAMES
eb75cf470aeb   my-web    "/docker-entrypoint.…"   12 seconds ago   Up 12 seconds   0.0.0.0:7566->80/tcp, [::]:7566->80/tcp    my-web

웹 서버 접속 결과: (사진 첨부)
5) Docker 바인드 마운트 및 실시간 반영 검증
PowerShell
# 기존 컨테이너 정리 후 바인드 마운트 적용 (-v 옵션 사용)
PS C:\dockerfile with codyssey> docker rm -f my-web
my-web

PS C:\dockerfile with codyssey> cd "C:\dockerfile with codyssey"
PS C:\dockerfile with codyssey> docker run -d -p 7566:80 --name my-web-live -v ${PWD}:/usr/share/nginx/html nginx
768b9d528a150bb0eb5f0f99147d9ffe17873d3ed241d868be4965165e341de8

실시간 수정 반영 결과: (사진 첨부)
6) Docker 볼륨 영속성 검증
PowerShell
# 1. Docker 볼륨 생성
PS C:\WINDOWS\system32> docker volume create mydata
mydata

# 2. 볼륨을 연결한 Ubuntu 컨테이너 실행 및 테스트 파일 작성 후 종료
PS C:\WINDOWS\system32> docker run -it --name vol-test -v mydata:/data ubuntu
root@a55f5517eb05:/# echo "hello volume" > /data/memo.txt
root@a55f5517eb05:/# exit
exit

# 3. 기존 컨테이너 삭제
PS C:\WINDOWS\system32> docker rm vol-test
vol-test

# 4. 새로운 임시 컨테이너를 실행하여 삭제된 컨테이너의 볼륨 데이터 생존 확인
PS C:\WINDOWS\system32> docker run -it --rm -v mydata:/data ubuntu cat /data/memo.txt
hello volume

# 5. 도커 볼륨 목록 확인
PS C:\WINDOWS\system32> docker volume ls
DRIVER    VOLUME NAME
local     mydata
7) Git 설정 및 GitHub 연동 로그
Bash
김려원@DESKTOP-6CGCS3L UCRT64 ~
$ git config --global user.name "CroaRio"

김려원@DESKTOP-6CGCS3L UCRT64 ~
$ git config --global user.email "krw7566@gmail.com"

김려원@DESKTOP-6CGCS3L UCRT64 ~
$ git config --global init.defaultBranch main

김려원@DESKTOP-6CGCS3L UCRT64 ~
$ git config --list
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=schannel
core.autocrlf=true
core.fscache=true
core.symlinks=false
pull.rebase=false
credential.helper=manager
credential.https://dev.azure.com.usehttppath=true
init.defaultbranch=master
user.name=CroaRio
user.email=krw7566@gmail.com
init.defaultbranch=main

GitHub 계정 연동: (사진 첨부)
