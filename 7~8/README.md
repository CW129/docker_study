## 07 유틸리티 컨테이너로 작업하기.

### utility container란

- 특정환경만을 포함하는 컨테이너(공식용어는 아님.)
- 애플리케이션 빌드를 도와주기 위해 도구로써 사용되는 애플리케이션, 서버, 데이터베이스, 작업환경 구성도구들을 일컫는 느낌.

### docker exec와 -it 플래그

- -it의 활용 : docker run -it image_name을 통해 컨테이너를 생성과 동시에 인터랙티브 모드로 사용 가능.

    docker run -it image_name image_cli로 image_name 뒤 원래 있는 default 명령어를 오버라이딩 할 수 있다. 예) docker run -it node npm init

- exec 명령어 : docker exec <이미 실행중인 컨테이너 name or id>를 통해 실행 중인 컨테이너 내부에 명령을 전달 할 수 있다.

**node:version_num-alpine** : alpine이란 리눅스의 경량화 이미지로, 일반 이미지에 비해 용량이 약 8~10배 정도 차이남.

### ENTRYPOINT

- 컨테이너 기동 시에 명령어 첫 줄에 항상 입력됨.
- 강의에서는 npm을 ENTRYPOINT로 사용하였는데 node 이미지는 npm 명령어를 사용하므로 ENTRYPOINT의 적합한 사용예시가 됨.
- docker cli를 사용할 경우 docker ~ ENTRYPOINT <이미지 내부 실행할 cli> 형태로 사용 가능.
- docker-compose cli를 통해 yaml파일에 정의된 명세대로 컨테이너를 기동할 경우, docker-compose run <service_name> <inner_container_cli> 와 같은 형태로 명령을 전달할 수 있다.

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
**docker-compose run --rm** : docker-compose up을 사용할 경우 docker-compose down을 통해 컨테이너 종료 및 삭제를 한번에 동작시킬 수 있다. docker-compose run을 사용했을 경우에는 --rm옵션으로 종료 시 자동삭제 옵션을 추가해주어야 한다. 이 옵션이 없을 경우 수동으로 일일이 삭제해주어야 함.

(사용하지 않는 컨테이너는 docker container prune으로 모두 제거 가능)


## 08 Laravel, php 도커화 프로젝트
![image](https://user-images.githubusercontent.com/59682268/222943176-b15d62f1-d988-43d4-8cb0-a48ac9471c0c.png)

### docker-compose.yaml

```yaml
version: "3.8"

services: 
  server:
    # image: 'nginx:stable-alpine'
    build:
      # context: ./dockerfiles
      # dockerfile: nginx.dockerfile
      context: .
      dockerfile: dockerfiles/nginx.dockerfile
    ports: 
      - '8000:80'
    volumes: 
      - ./src:/var/www/html
      # nginx.conf가 아닌 conf.d/default.conf를 사용하는 이유 : 더 큰 nginx 구성으로 병합하기 위함.
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - php
      - mysql
  php:
    build:
      context: .
      dockerfile: dockerfiles/php.dockerfile
    # volumes:
    #   - ./src:/var/www/html:delegated # 컨테이너가 일부 데이터를 기록해야 하는 경우 그에 대한 결과를 호스트 머신에 즉시 반영하지 않고 배치 처리함으로써 성능이 좀 더 나아짐.
    # ports:
      # - '3000:9000'
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  composer:
    build:
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  artisan:
    build:
      context: .
      dockerfile: dockerfiles/php.dockerfile
    volumes:
      - ./src:/var/www/html
    entrypoint: ["php", "/var/www/html/artisan"]
  npm:
    image: node:14
    working_dir: /var/www/html
    entrypoint: ["npm"]
    volumes:
      - ./src:/var/www/html
```

### nginx.conf

```conf
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
        # php로 dns이름을 설정. 이는 docker-compose의 server.php에서 가져온 이름.
        # fastcgi_pass php:3000;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

### composer.dockerfile

```Dockerfile
FROM composer:latest

WORKDIR /var/www/html

# composer 명령을 경고나 종속성 없이 실행하기 위한 --ignore 옵션임
ENTRYPOINT [ "composer", "--ignore-platform-reqs" ]
```

### nginx.dockerfile

```Dockerfile
FROM nginx:stable-alpine

WORKDIR /etc/nginx/conf.d

COPY nginx/nginx.conf .

RUN mv nginx.conf default.conf

WORKDIR /var/www/html

COPY src .

# CMD를 실행하지 않으면 default command로 nginx server를 시작하는 명령어가 실행된다.

# src, nginx 디렉토리가 현재 dockerfile 외부에 존재하므로 이대로 빌드하면 실패하게 됨.

# docker-compose.yaml에서 context를 . dockerfile을 dockerfiles/nginx.dockerfile로 설정해야 함.
```

### php.dockerfile
```Dockerfile
FROM php:8.0-fpm-alpine

WORKDIR /var/www/html

COPY src .

RUN docker-php-ext-install pdo pdo_mysql

RUN chown -R www-data:www-data /var/www/html

# dockerfile에 CMD 또는 ENTRYPOINT가 없다면 base image의 CMD, ENTRYPOINT가 실행됨.
# 여기서는 PHP 인터프리터를 호출함.
```
