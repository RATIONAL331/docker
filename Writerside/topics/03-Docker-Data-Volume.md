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

![Volume & Bind Mounts](bindMounts.png)

## Key Command
* `docker volume ls`

### Anonymous Volume
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
### Named Volume
* `docker run --rm -v feedback:/app/feedback {imageName:version}`
  * -v 옵션은 명명 볼륨을 지정할 수 있고 {볼륨이름}:{컨테이너 내부 매핑} 로 작성
  * 컨테이너가 제거되더라도 유지
  * 재실행할 때 -v 옵션으로 기존 명명된 볼륨을 지정하면 해당 내용을 컨테이너 내부로 매핑 가능


## Bind Mounts
* 볼륨과 비슷하지만 볼륨은 도커과 관리
* 호스트 머신 상에 매핑될 컨테이너 경로를 설정
* `docker run --rm -v "/Users/rational331/Desktop/data-volumes-02-added-dockerfile:/app" {imageName:version}`
  * -v 옵션은 볼륨 뿐만 아니라 바인드 마운트를 지정할 수 있고 {호스트 폴더 및 파일}:{컨테이너 내부 매핑} 로 작성
  * 호스트 폴더 및 파일 지정시 전체 경로 또는 단축어를 사용함
    * `-v $(pwd):/app (Mac)`
