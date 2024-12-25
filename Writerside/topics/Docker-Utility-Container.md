# Docker Utility Container

## Utility Container
* 환경만 포함하는 컨테이너를 의미
    * 즉 애플리케이션을 실행하지 않는 컨테이너
* 특정 작업을 실행하기 위해 지정한 명령과 함께 실행
* 컨테이너를 시작할 시에 이미지 이름 뒤에 부가 명령을 추가하는 형식으로 수행
* `docker exec {containerName}` 명령을 사용하면 실행중인 컨테이너에 명령을 수행할 수 있음
    * -it 플래그를 사용하면 터미널 모드로 진입가능
* 동적으로 생성된 파일의 소유권 관련된 아티클은 아래 확인
  * https://vsupalov.com/docker-shared-permissions/

```docker
FROM node:14-alpine

WORKDIR /app
```
* 단순한 환경만 설정한 후 `docker build -t node-util .` 수행
* `docker run -it node-util npm init` 
  * 뒤의 `npm init`을 같이 수행하도록 오버라이딩
* 이 때 바인드 마운트를 사용한다면 컨테이너의 내용을 호스트로 미러링 가능
  * `docker run -it -v {hostDir}:{containerWorkDir} node-util npm init`
    * `node`이미지를 수행하고 `npm init`를 같이 수행
  * package.json 파일이 호스트 파일에 미러링
* 위 기능을 사용하면 호스트 머신에 node 없이도 필요한 기능을 받을 수 있음

## ENTRYPOINT
* `CMD` 와 유사하지만 큰 차이가 존재
  * `CMD` 경우에 `docker run`의 추가 명령을 오버라이딩
  * 즉 `CMD`로 설정된 문장이 무시됨
* `ENTRYPOINT`의 경우 `docker run`의 추가 명령을 `ENTRYPOINT` 명령 뒤에 추가 
  * 즉 `ENTRYPOINT` 명령이 먼저 수행되고 `docker run`의 추가 명령 수행

```docker
FROM node:14-alpine

WORKDIR /app

ENTRYPOINT [ "npm" ]
```
* `ENTRYPOINT` 까지 설정 후`docker build -t node-util .` 수행
  * `docker run -it -v {hostDir}:{containerWorkDir} node-util init`
  * `ENTRYPOINT`에 이미 `npm`이 설정되어서 `init` 를 수행하면 `npm init`가 수행됨

## With Compose
```yaml
version: "3.8"
services:
  npm:
    build: ./
    stdin_open: true
    tty: true
    volumes:
      - ./:/app
```
* `docker compose run {serviceName} {exec}`
  * `docker compose run npm init`
  * 서비스 이름으로 단일 서비스 대상으로 지정하여 실행할 수 있음
  * 해당 명령어는 `docker compse up`과 다르게 --rm 옵션을 추가하지 않음
  * `docker compose run --rm npm init`
    * --rm 을 사용하여 실행되고 자동으로 컨테이너 삭제가 가능