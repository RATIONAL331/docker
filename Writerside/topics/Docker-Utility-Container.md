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
* `CMD` 또는 `ENTRYPOINT`가 없을 때는 베이스 이미지의 `CMD` 또는 `ENTRYPOINT`가 사용됨

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

## Example
### nginx.conf
```nginx
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    root /var/www/html/public;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

### php.dockerfile
```docker
FROM php:7.4-fpm-alpine

WORKDIR /var/www/html

RUN docker-php-ext-install pdo pdo_mysql
```

### composer.dockerfile
```Docker
FROM composer:latest

WORKDIR /var/www/html

ENTRYPOINT [ "composer", "--ignore-platform-reqs" ]
```

### docker-compose.yml
```yaml
version: "3.8"
services: 
  server:
    image: 'nginx:stable-alpine'
    ports: 
      - '8000:80'
    volumes: 
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./src:/var/www/html
    # 만약에 의존하고 있는 서비스가 있을 때 해당 서비스도 같이 실행
    depends_on:
      - php
      - mysql
  php:
    build: 
      context: ./dockerfiles
      dockerfile: php.dockerfile
    volumes:
      # delegated는 컨테이너의 내용이 호스트 머신에 즉시 반영하지 않고 배치 형태로 처리
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_DATABASE=homestead
      - MYSQL_USER=homestead
      - MYSQL_PASSWORD=secret
      - MYSQL_ROOT_PASSWORD=secret
  composer: # Utility Container
  # docker compose run --rm composer create-project --prefer-dist laravel/laravel ./
    build: 
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  artisan: # 라라벨 프레임 워크에서 DB 초기화 데이터를 쌓을 때 도움을 주는 유틸리티
    # docker compose run --rm artisan migrate
    build:
      context: . # 컨텍스트를 부모로 잡아야 하는 경우에는 컨텍스트 위치를 변경
      dockerfile: dockerfiles/php.dockerfile
    environment:
      - DB_CONNECTION=mysql
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_DATABASE=homestead
      - DB_USERNAME=homestead
      - DB_PASSWORD=secret
    volumes:
      - ./src:/var/www/html
    entrypoint: ["php", "/var/www/html/artisan"]
```
* Laravel 프로젝트를 위한 `docker compose run --rm composer create-project --prefer-dist laravel/laravel ./` 먼저 수행
* 그 후 `docker compose run --rm artisan migrate` 수행
* 최종적으로 `docker compose up server` 수행
  * `docker compose up {serviceName}` 으로 원하는 컨테이너만 실행할 수 있으며 `depends_on` 도 같이 수행
