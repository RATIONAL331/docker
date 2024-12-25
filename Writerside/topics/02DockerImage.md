# Docker Image

## Image & Container
* 컨테이너는 이미지의 구체적 실행 인스턴스
* 이미지는 컨테이너의 템플릿 또는 청사진
  * 실제로 코드를 실행하는데 필요한 도구를 포함
* 이미지 하나로 여러 컨테이너에서 실행할 수 있음

## Key Command
* `docker run`
  * 이미지를 기반으로 실행 인스턴스 생성
  * 즉 이미지를 기반으로 컨테이너 생성
  * Attached 모드로 수행(foreground)
    * -d 옵션을 활용하여 Detached(background)로 수행가능
  * --rm을 추가하면 해당 컨테이너가 중지될 때 자동으로 컨테이너가 삭제됨
  * --name을 추가하면 컨테이너의 이름을 지정할 수 있음
* `docker run node`
  * docker hub에서 node 이미지를 찾아 실행
* `docker run -it node` (interactive)
  * 컨테이너 내부와 interactive 가능
  * -i는 STDIN을 지속적으로 여는 옵션
  * -t는 pseudo TTY를 할당(터미널 생성)
* `docker ps -a`
  * -a 옵션이 없다면 현재 실행중인 프로세스만 표시
* `docker start`
  * Detached 모드로 수행
  * -a 옵션을 활용하면 Attached(foreground)로 수행
    * -i 옵션을 추가하면 interactive 가능
* `docker attach`
  * Detached 된 컨테이너에 터미널 연결
* `docker stop`
  * 실행중인 컨테이너 중지
* `docker logs`
  * 해당 컨테이너에 출력된 로그 확인
  * -f 옵션을 활용하면 지속 수신 모드로 진입
* `docker rm`
  * 컨테이너 삭제
  * 컨테이너를 삭제하기 위해서는 컨테이너가 중지된 상태여야 함
* `docker rmi`
  * 이미지 삭제
  * 이미지를 삭제하기 위해서는 해당 이미지가 컨테이너에서 사용중이면 안됨(정지되어 있더라도)
* `docker cp {from} {to}`
  * 컨테이너에 로컬 파일 및 폴더를 복사하거나 로컬에 컨테이너가 가진 파일 및 폴더를 복사
  * `docker cp {localDir}/. {container_name}:{toDir}`
    * 로컬의 localDir 안의 전체 파일 내용을 지정한 컨테이너의 디렉토리에 복사
  * `docker cp {container_name}:{fromDir/fromFile} {localDir}`
    * 컨테이너 디렉토리 또는 파일의 내용을 로컬의 localDir 로 복사

## Dockerfile

```Docker
# docker hub에 있는 node 이미지 사용
FROM node 

# 컨테이너의 working dir 설정
WORKDIR /app

# COPY (이미지 외부 경로) (이미지 내부 경로)
# COPY (복사되어야 할 호스트 파일 경로) (컨테이너 파일 시스템 경로)
COPY . /app 

# RUN 명령어는 이미지가 생성될 때 명령어를 수행하게끔 함
# 기본적으로 모든 명령은 컨테이너의 working dir에서 수행하며 기본값은 루트
# RUN command parameter
RUN npm install 

# 빌드 과정에서 서버 실행과 같은 지속적인 동작은 사용하면 안됨
# 이미지를 빌드하는 동안 서버가 실행됨
# RUN node server.js # 이미지를 만들기에 적합하지 않은 명령어

# 컨테이너 내부에서 수신 대기중인 포트가 있을 때 호스트에게 노출할 것임을 '문서화'
EXPOSE 80

# CMD 명령어는 RUN 명령어와 다르게 이미지를 기반으로 컨테이너가 시작될 때 수행 됨
# CMD는 Dockerfile에서 한 번만 사용 가능하며 여러번 작성할 시 마지막에 작성한 것만 유효
# CMD ["exec", "parameter", ...]
CMD ["node", "server.js"]
```
## Docker Build & Run
```Console
# 명시된 dir에서 Dockerfile을 찾아 빌드
docker build . 

# 빌드 시 태그 명을 지정할 수 있음
# docker build -t groupname:latest .

# 수행 (아직 정상 수행 X)
docker run IMAGE_ID
```

### publish 기능을 사용하지 않을 때
```Console
CONTAINER ID   IMAGE          COMMAND                   CREATED          STATUS          PORTS     NAMES
8c8fde363427   b8f63c4bb912   "docker-entrypoint.s…"   10 seconds ago   Up 10 seconds   80/tcp    blissful_jepsen
```

```Console
# docker stop CONTAINER (아이디 또는 이름)
# 실행 중인 컨테이너 종료
docker stop 8c8fde363427
```

## Docker Run with Publish
```console
# -p (publish) 호스트 포트:컨테이너 포트
docker run -p 3000:80 IMAGE_ID
```
```console
CONTAINER ID   IMAGE          COMMAND                   CREATED         STATUS         PORTS                  NAMES
91f5fc10ac0d   b8f63c4bb912   "docker-entrypoint.s…"   6 seconds ago   Up 5 seconds   0.0.0.0:3000->80/tcp   pensive_mayer
```
* 해당 명령 수행 후 localhost:3000 접속 가능

## Image
![Container Image Layer](imageLayer.png)
* Dockerfile에 작성한 각각의 문장은 레이어 기반으로 동작
* 만약 실행한 문장을 캐싱할 수 있다면 도커는 이를 감지할 수 있음

```docker
FROM node 

WORKDIR /app

# 해당 문장을 먼저 실행하여
# 패키지가 변경된 경우에만 npm install을 수행하게끔 변경
COPY package.json /app

# COPY . /app => 해당 문장을 npm install 뒤로 미뤄서 무거운 작업을 막기
# 뒤로 미루지 않는다면 server.js가 변경되면 아래 문장은 항상 수행 됨
# 다시 말해 RUN npm install과 같은 무거운 작업이 항상 실행됨
RUN npm install 

# 이제 npm install은 이제 package.json이 변경된 경우메만 실행됨
# server.js가 변경되면 아래 문장 수행
COPY . /app

EXPOSE 80

CMD ["node", "server.js"]
```

### Image Tag
* groupName:version 로 이루어짐
* `docker build -t groupname:latest .` 로 지정 가능
* `docker tag {oldName} {newName}`로 기존 태그를 가지는 이미지로 새로운 태그명으로 복사본을 생성 가능

## Docker Hub Image
* Hub에 올리려는 이미지의 태그명을 같은 Hub Repo와 같은 이름으로 지정
* Hub에서 받으려면 Hub Repo명으로 Pull

```Console
# docker tag는 기존에 oldName 가지는 이미지를 newName으로 새롭게 이미지 생성(복사)
# docker tag {oldName} {newName}
docker tag rational331:latest rational331/helloserver-docker:latest

# push 도중에 에러 발생시 로그인 확인
docker login

# docker push {repoName(:version)}
docker push rational331/helloserver-docker

# docker pull {repoName(:version)}
docker pull rational331/helloserver-docker
```

### Update & Run
* 이미지를 `docker run`으로 수행할 때 이미지가 로컬에 존재하면 해당 로컬 이미지를 먼저 사용
  * 없다면 Hub에서 이미지를 찾아 `pull`
* 만약 업데이트 되었는지 확인하려면 `docker pull`로 확인해야함