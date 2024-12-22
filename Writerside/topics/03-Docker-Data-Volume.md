# 03. Docker Data &amp; Volume

![Docker Data Type](dataAndVolume.png)

* 기본적으로 컨테이너가 삭제되면 컨테이너 내부에서 생성된 파일들은 모두 사라짐

## Volume
* 호스트 머신의 폴더; 호스트 컴퓨터에 장착된 저장장치를 컨테이너의 폴더로 맵핑
* 호스트 머신에 파일을 추가하면 컨테이너 내부에서 엑세스 가능
* 반대로도 컨테이너가 매핑된 경로에 파일을 추가하면 호스트에서도 사용 가능
* 컨테이너가 삭제되더라도 지속됨
* 익명 볼륨, 명명 볼륨이든 두 경우 모두 호스트 머신에 설정(도커가 관리)
  * 익명 볼륨은 컨테이너가 삭제될 때 볼륨도 같이 삭제
  * 명명 볼륨은 컨테이너가 삭제되더라도 볼륨은 삭제되지 않음
* 경로가 구체적일수록 우선순위가 높아짐
* 내부 경로 뒤에 `:ro` 입력시 읽기 전용 모드 볼륨으로 설정

![Volume & Bind Mounts](bindMounts.png)

## Key Command
* `docker volume ls`
* `docker volume create {newName}`
  * 자주 사용되지 않지만 직업 볼륨을 만들 때

## Anonymous Volume
* `docker run --rm -v /app/node_modules {imageName:version}`
  * -v 옵션으로 익명 볼륨을 지정할 수 있고 {컨테이너 내부 매핑} 으로만 작성
* 컨테이너에 이미 존재하는 특정 데이터를 잠글 때 유용
  * 바인드 마운트 적용시 적용 제외 대상을 지정할 때 사용 가능
  * 바인드 마운트에 의해서 덮어 쓰여짐 방지
  * 외부 경로보다 내부 경로의 우선 순위를 높일 때 사용
```Docker
FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

# VOLUME ["{ContainerDir}"] 
# 맵핑할 컨테이너 내부를 설정하여 익명 볼륨 설정
VOLUME ["/app/node_modules"]

CMD [ "npm", "start" ]
```
* Dockerfile 에서 VOLUME 명령어를 사용해도 익명 볼륨으로 사용됨
## Named Volume
* `docker run --rm -v feedback:/app/feedback {imageName:version}`
  * -v 옵션은 명명 볼륨을 지정할 수 있고 {볼륨이름}:{컨테이너 내부 매핑} 로 작성
  * 컨테이너가 제거되더라도 유지
  * 재실행할 때 -v 옵션으로 기존 명명된 볼륨을 지정하면 해당 내용을 컨테이너 내부로 매핑 가능


## Bind Mounts
* 볼륨과 비슷하지만 볼륨은 도커과 관리하지만 바인드 마운트는 호스트가 관리
* 호스트 머신 상에 매핑될 컨테이너 경로를 설정
* `docker run --rm -v "/Users/rational331/Desktop/data-volumes-02-added-dockerfile:/app" {imageName:version}`
  * -v 옵션은 볼륨 뿐만 아니라 바인드 마운트를 지정할 수 있고 {호스트 폴더 및 파일}:{컨테이너 내부 매핑} 로 작성
  * 호스트 폴더 및 파일 지정시 전체 경로 또는 단축어를 사용함
    * `-v $(pwd):/app (Mac)`

## Read-Only Volume
`docker run --rm -v "/Users/rational331/Desktop/data-volumes-02-added-dockerfile:/app:ro" {imageName:version}`

## ARGument & ENVironment
* Dockerfile 및 `docker run|build` 명령에서 사용 가능
* `ARG`는 빌드 타임 환경 변수 설정 관련 명령어
  * `docker build --build-arg`
* `ENV`는 런타임 환경 변수 설정 관련 명령어
  * `docker run --env`

### ENV
```Docker
FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

# ENV KEY defaultVALUE
ENV PORT 80

# 변수 명앞에 $를 붙여 환경 변수임을 알림
EXPOSE $PORT

# VOLUME ["{ContainerDir}"] 
# 맵핑할 컨테이너 내부를 설정하여 익명 볼륨 설정
VOLUME ["/app/node_modules"]

CMD [ "npm", "start" ]
```

* `docker run -d --rm -p 3000:8000 --env PORT=8000 {imageName}`
  * 위 명령어로 실행 시 포트 번호를 수행 시 설정할 수 있음
  * -e 로 줄여 사용 가능함
  * -e 역시 -v 처럼 여러개 사용 가능함
  * `--env-file {fileName}`를 통해 파일을 통해 환경 설정 변수를 불러올 수 있음
    * 아래와 갇이 파일안에 = 기준으로 키와 값을 지정
```Plain Text
PORT=8080
NAME=TEST
```

### ARG
```Docker
FROM node:14

# ARG key=value
ARG DEFAULT_PORT=80
 
WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

ENV PORT $DEFAULT_PORT

EXPOSE $PORT

VOLUME ["/app/node_modules"]

CMD [ "npm", "start" ]
```
* ARG 명령어로 정의한 변수는 CMD에서 사용할 수 없음
  * CMD 명령어는 컨테이너가 시작될 때 실행되는 런타임 명령어
* 빌드 시에 `--build-arg`로 변수를 재지정 할 수 있음
  * `docker build -t feedback-node:dev -- build-arg DEFAULT_PORT=8000 .`

### Docker Ignore
* 프로젝트 폴더에 `.dockerignore` 파일을 생성후 해당 파일 안에 COPY 명령을 통해 복사하면 안되는 폴더 및 파일을 지정 가
```plain text
Dockerfile
node_modules
.git 
```
* npm install 후 호스트에 있는 node_modules 폴더를 COPY 명령어를 통해 복사하지 않음



