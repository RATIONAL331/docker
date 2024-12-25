# Docker Compose
* `docker build`, `docker run` 명령을 대체할 수 있는 도구
* 여러개의 `docker build`, `docker run` 명령을 하나의 구성 파일로 만들 수 있음
* 위의 구성 파일과 Orchestration Commands(build, start, stop, ...) 와 함께 구성할 수 있음
* `Docker Compose`는 컨테이너를 여러개 구성할 때 많은 도움이 됨
  * `Dockerfile`을 대체하지는 않음
  * Image나 Container를 대체하지는 않음
  * 다중 호스트 머신에서 다중 컨테이너를 관리하는데 적합하지 않음 
    * 동일 호스트 머신에서 다중 컨테이너를 관리하는 것에 적합

## docker-compose.yml
```yaml
version: '3.8' # 도커 컴포즈의 버전
services: # 각 서비스(컨테이너) 대한 레이블 명명
          # 실제 실행시켰을 때 컨테이너 이름은 아니지만 도커에 의해서 관리되기 때문에 네트워크 이름으로 그대로 사용 가능
  mongodb: # --rm과 -d는 필요하지 않음
    # container_name: mongodb # 컨테이너 이름을 직접 명명해서 사용하려면 해당 옵션 사용
    image: 'mongo' # 사용할 이미지 이름
    volumes: # 볼륨은 - 로 여러개 지정 가능
      - mongo_data:/data/db
    environment: # 환경 설정 key=value 또는 key: value 로 지정
                 # key=value로 여러개 지정할 시 - 필요
      #- MONGO_INITDB_ROOT_USERNAME=max
      #- MONGO_INITDB_ROOT_PASSWORD=secret
      MONGO_INITDB_ROOT_USERNAME: max
      MONGO_INITDB_ROOT_PASSWORD: secret
    #env_file: # 환경 설정 파일을 이용하는 경우 env_file 사용 
      #- fileName
    #networks: 네트워크 설정이 추가적으로 필요하다면 작성; 
    # 대부분의 경우에 Compose 파일에 작성된 모든 서비스에 대해서 새 네트워크 환경을 자동으로 설정하여 네트워크 지정 불필요
      #- networkName # 자동 생성된 네트워크 뿐만 아니라 지정한 네트워크에도 추가 가능
  backend:
    build: ./backend # 빌드 대상이 될 경로 (Dockerfile 파일이 존재하는 경로)
    #build: # 좀 더 구체적인 형태로 작성 가능 (Dockerfile 파일 이름이 다른 이름으로 된 경우 등)
      #context: ./backend 
      #dockerfile: Dockerfile
      #args: # Dockerfile ARGument 지정 (key=value 또는 key: value)
      #      # key=value로 여러개 지정할 시 - 필요
      #  - name=value
    ports: # 목록형태로 지정
      - '80:80'
    volumes:
      - back_logs:/app/logs
      - ./backend:/app # 바인드 마운트
      - /app/node_modules # 익명 볼륨
    environment:
      - MONGODB_USERNAME=max
      - MONGODB_PASSWORD=secret
    depends_on: # 해당 컨테이너가 다른 컨테이너에 의존할 때 선언
      - mongodb # 의존할 레이블 지정
  frontend:
    build: ./frontend # 빌드는 항상 실행되지 않고 변경되었을 때만 수행
    ports:
      - '3000:3000'
    volumes:
      - ./frontend/src:/app/src
    stdin_open: true # -i 옵션
    tty: true # -t 옵션
    environment:
      - NODE_OPTIONS=--openssl-legacy-provider
    depends_on:
      - backend
    
volumes: # 도커 컴포즈가 인식해야 할 명명된 볼륨명(Named Volume) 작성 
         # 익명 볼륨 및 바인드 마운트는 지정하지 않음
  mongo_data: # 키만 작성하고 값은 작성하지 않아도 됨
  back_logs: # 키만 작성하고 값은 작성하지 않아도 됨
```

## Key Commands
* `docker compose up`
  * Compose 파일에서 찾을 수 있는 모든 서비스 시작
  * 컨테이너를 시작할 뿐 아니라, 모든 이미지를 가져와 빌드함
  * 모든 서비스에서 동일하게 사용할 디폴트 네트워크 생성
  * -d 옵션을 사용하면 detached 모드로 실행 가능
  * --build 옵션을 사용하면 이미지를 강제로 리빌드
* `docker compose down`
  * 모든 컨테이너가 삭제 및 생성된 디폴트 네트워크 삭제
  * 생성된 볼륨은 삭제되지 않음
    * 함께 삭제하려면 -v 옵션을 사용
* `docker compose build`
  * 컨테이너를 수행하지 않고 이미지를 빌드하기만 할 경우 사용