# Docker Deployment

## Development & Production
<tabs>
    <tab title="Development">
        <ul>
            <li><p>컨테이너는 런타임 환경을 캡슐화 해야하지만 코드까지 캡슐화할 필요는 없음</p></li>
            <li><p>애플레케이션을 실행하는데 필요한 도구만 있으면 됨</p></li>
            <li><p>애플레케이션에 대한 코드를 외부에서 주어지면 컨테이너를 다시 시작하지 않고도 최신 코드 반영</p></li>
            <li><p>바인드 마운트를 통하여 호스트의 파일을 실행중인 컨테이너에서 사용가능</p></li>
        </ul>
    </tab>
    <tab title="Production">
        <ul>
            <li><p>반드시 독립적으로 수행될 수 있어야 함</p></li>
            <li><p>즉 이미지는 반드시 어떠한 추가 코드나 설정 정보 없이 수행될 수 있어야 함</p></li>
            <li><p>바인드 마운트를 사용하지 않고 반드시 코드 스냅샷 복사본을 사용해야 함</p></li>
            <li><p>즉 이미지 빌드 시 소스 코드를 이미지에 복사해야 함</p></li>
        </ul>
    </tab>
</tabs>

## AWS EC2
* EC2 인스턴스를 설정한 후 다음 명령어 입력하여 EC2 인스턴스에 도커 설치
```Console
sudo yum update -y
sudo yum -y install docker

sudo service docker start
 
sudo usermod -a -G docker ec2-user

# 아래 명령은 로그아웃 후 다시 접속하여 수행
sudo systemctl enable docker

# 정상적으로 설치되었는지 확인
docker version
```
* 배포 방식은 로컬 호스트 머신에서 빌드를 모두 한 후 EC2에서 이미지를 받아서 배포

### Local Machine
#### Docker Ignore
```text
node_modules
Dockerfile
*.pem
```
* `docker build --platform linux/amd64 -t node-dep-example .`
* 도커 허브의 리포지토리 명과 같게 태그 추가
  * `docker tag node-dep-example {hubRepository}`
* `docker push {hubRepository}`

### EC2
* `sudo docker pull rational331/helloserver-docker`
* `sudo docker run -d --rm -p 80:80 rational331/helloserver-docker`
* 그 후 인스턴스에 할당된 IP로 접속

## AWS ECS (Elastic Container Service)
* EC2는 자원 관리 및 장비 관리를 직접해야하지만 ECS를 사용하면 관리, 업데이트 및 스케일링을 단순화 할 수 있음
* 해당 서비스를 사용하면 더 이상 컨테이너 배포 및 실행을 직접 도커 명령으로 하지 않음
* 배포할 이미지를 Docker Hub 또는 다른 컨테이너 레지스트리 저장소에 있는 것으로 컨테이너 설정
* FARGATE 디폴트 옵션은 서버리스와 관련되며 필요할 때만 EC2 인스턴스 서버를 구동
* 이미지를 새롭게 빌드하여 배포하려면 ECS의 Task(Update Service)를 새롭게 추가해야 함
  * 업데이트 된 컨테이너를 배포할 때 마다 새로운 IP를 할당하기 때문에 ECS 자체에 로드밸런서를 붙여야 함

## Deploy Multi Container
* 각각의 컨테이너가 항상 동일한 머신에서 실행되지는 않게될 수 있음
  * 즉, 더 이상 Network IP Resolving 기능을 사용하지 못함
* AWS ECS에서는 동일 태스크에 컨테이너를 추가하면 항상 동일한 머신에서 수행됨
* 같은 태스크에 컨테이너를 새롭게 추가하여 여러개의 컨테이너 설정 가능

### Load Balancer
* 업데이트 된 컨테이너를 배포할 때 마다 Public IP가 바뀌는 문제를 해결하기 위한 한 가지 수단
* 타겟 그룹에 대해서 Health Check 옵션과 로드 밸런서의 보안 설정을 해준 후 로드 밸런서 자체의 DNS 로 접근 가능

### EFS (Elastic File System)
* EFS를 활용하면 볼륨으로 사용 가능
* 네트워크 관련 설정에서 보안 관련 설정을 바꿔줘야 함
  * 인바운드 규칙으로 NFS 타입을 설정해야 함
  * 해당 인바운드 규칙을 설정하지 않으면 ECS와 EFS 간의 통신이 이루어지지 않

## Dockerfile.prod (Multistage Build)
* React, Vue, Angular 와 같은 프로젝트와 같이 배포에 다른 방식을 사용할 수 있음
```Docker
# FROM {image} as {stageName}
FROM node as build

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

RUN npm run build

# FROM 을 다시 사용하면 새로운 베이스 이미지로 전환 (멀티 스테이지)
FROM nginx:stable-alpine

# COPY --from={stageName} {A} {B} 기존에 있던 스테이지의 내용을 가져옴
# 로컬 호스트 머신에 있는 프로젝트 폴더를 참조하지 않고 기존 스테이지 파일 시스템의 경로를 참조
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

# nginx 시작
CMD ["nginx", "-g", "daemon off;"]
```
* `docker build -f frontend/Dockerfile.prod ./frontend`
  * -f 옵션으로 도커파일을 직접 지정 가능
  * 멀티 스테이지 도커 파일에서 필요하다면 --target 옵션을 사용하여 해당 부분만 빌드가 가능함
    * `docker build --target build -f frontend/Dockerfile.prod ./frontend`
      * `FROM nginx:stable-alpine` 부터 수행하지 않음